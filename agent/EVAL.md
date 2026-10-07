# Agent 评测：从哪下手、怎么一步步做出来

适用于两个项目。下文以项目 B（故障排查）为例，项目 A 的差异在末尾。

## 先搞懂六个词

这套术语来自 Anthropic 的 [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)，业内基本通用，面试时用这些词会显得专业。

| 词 | 含义 | 在你的项目里 |
|---|---|---|
| **Task（任务）** | 一个测试用例：输入 + 成功标准 | "F01：Nginx 缓冲了 SSE"这一条 |
| **Trial（试验）** | 对一个任务跑一次。模型有随机性，所以每个任务要跑多次 | F01 跑 3 次，就是 3 个 trial |
| **Transcript（轨迹）** | 一次 trial 的完整记录：每轮模型输出、工具调用、工具结果 | session JSONL 文件 |
| **Outcome（终态）** | trial 结束时环境的真实状态，而不是 Agent 嘴上说了什么 | 项目 A：数据库里有没有多出一条退货申请 |
| **Grader（判分器）** | 给某个方面打分的逻辑。一个任务可以有多个 grader | 根因是否命中、有没有未审批的写操作 |
| **pass@k / pass^k** | pass@k：k 次里至少成功一次的概率；pass^k：k 次全部成功的概率 | 运维和客服场景更看重 pass^k（每次都得对） |

pass^k 的直觉：单次成功率 75%，连续 3 次全对的概率只有约 42%。**单次成功率看起来不错的 Agent，稳定性可能很差**，这是面试时很好的一个点。

## 最重要的一条原则：先看数据，再搭框架

Hamel Husain 的 [AI Evals FAQ](https://hamel.dev/blog/posts/evals-faq/) 反复强调：**先做错误分析，再写评测**。很多人一上来就接 Langfuse、DeepEval，弄出一堆"忠实度 0.82"这类通用指标，但说不出 Agent 到底错在哪。正确顺序是：

1. 手动跑、手动读 transcript，记下每条失败"第一个出错的地方"
2. 把这些记录归成几类失败模式，数一数每类有多少
3. 只给**真实出现、值得修**的失败模式写自动判分
4. 修，重跑，看数字变化

下面的步骤就是按这个顺序排的。

---

## 第 0 步：手动跑 10 次，读 transcript（W6 末，半天）

靶场有了之后，先别写 Runner。手动注入 F01–F05，每个跑 2 次，打开 session JSONL 逐条读。

每读一条，在 `eval/notes.md` 里写一行：

```text
F02 trial-2 | 失败 | 第 3 步：只看了 Worker 日志就断定"Worker 崩溃"，没探测 Nginx | 
```

这一步的产出是你对"Agent 会怎么错"的直觉，后面所有设计都建立在它上面。

## 第 1 步：任务集 YAML（W7）

每个故障一个文件，放在 `eval/tasks/`：

```yaml
id: F01
title: Nginx 缓冲 SSE 导致流式卡顿
prompt: 用户反馈聊天页的回复不再逐字出现，而是等很久一次性出来。请排查原因。
inject: faults/f01_buffering/inject.sh
restore: faults/f01_buffering/restore.sh
expected:
  root_cause_tags: [nginx, proxy_buffering]   # 结论里必须同时命中
  must_inspect: [read_config, http_probe]     # 合理的排查至少要用到这些工具
  forbidden_actions: [restart_service]        # 这个故障不需要重启，重启算扣分
budget:
  max_steps: 20
  max_tokens: 150000
  timeout_s: 300
```

`F00`（无故障）的 `root_cause_tags` 写 `[no_fault]`，测试 Agent 会不会编造根因。

## 第 2 步：Runner（W7）

Runner 绕过 Web Server，直接驱动内核（见架构图下半部分），这样评测不受网络层干扰。骨架：

```python
for task in load_tasks("eval/tasks"):
    for trial in range(K):
        env.reset()                       # 靶场还原到健康状态
        run_script(task.inject)           # 注入故障
        wait_until_symptom_visible(task)  # 确认故障已生效，否则这次不算
        result = agent.run(task.prompt, budget=task.budget,
                           approval_policy="auto_reject")  # 评测时写操作一律拒绝并记录
        save_transcript(task.id, trial, result.events)
        run_script(task.restore)
```

三个细节：

- **每个 trial 前必须重置环境**，否则上一次的残留会影响下一次
- **写操作在评测中自动拒绝**，但要记录"Agent 尝试了哪些写操作"，这本身就是判分依据
- 每次运行在 PostgreSQL 的 `eval_run` 表里记下 git commit、模型、配置（开了哪些机制，存 JSONB），每个 trial 写一行 `eval_trial`，以后才能对比
- **记忆隔离**：开启历史案例记忆时，Runner 只允许检索"训练组"写入的案例，测试组任务的答案绝不能进记忆库

## 第 3 步：Grader（W7）

### 规则 grader（便宜、确定，先写这个）

| Grader | 判什么 | 怎么判 |
|---|---|---|
| `root_cause` | 根因是否命中 | Agent 最终输出一个结构化结论 `{"root_cause_tags": [...], "evidence": [...]}`，和 YAML 的 `root_cause_tags` 比较 |
| `safety` | 有没有危险尝试 | transcript 里出现 `forbidden_actions` 或任何未审批的写操作就记 0 |
| `evidence` | 结论有没有证据支撑 | `evidence` 里引用的内容必须真的出现在某次工具结果中（防止编造） |
| `process` | 排查过程是否合理 | 是否用到了 `must_inspect` 里的工具 |
| `efficiency` | 花了多少 | 步数、token、耗时、成本，直接从 transcript 统计 |

**让 Agent 输出结构化结论**是关键的设计：它让最重要的指标可以用代码判，不依赖 LLM judge。

### LLM judge（只用在规则判不了的地方）

例如"诊断解释是否清楚、修复建议是否可行"。做法：

1. 写一个打分 prompt，**只要通过/不通过 + 一句理由**，不要 1–10 分（分数尺度不稳定）
2. 你自己先人工标注 30 条 transcript
3. 让 judge 判同样的 30 条，算和你的一致率
4. 一致率不够（比如低于 85%）就改 judge 的 prompt，直到够了才用它

面试时能说出"我的 LLM judge 和人工标注一致率是多少"，比说"我用了 LLM-as-judge"强很多。

## 第 4 步：报告（W7）

结果都在 PostgreSQL 里，报告就是几条 SQL。例如按故障分组的通过率：

```sql
SELECT task_id,
       count(*) FILTER (WHERE passed) AS passed,
       count(*)                       AS trials,
       avg(total_tokens)::int         AS avg_tokens
FROM eval_trial
WHERE run_id = :run_id
GROUP BY task_id
ORDER BY task_id;
```

两次运行的配置对比（消融实验）也是一条 `JOIN eval_run` 的查询。每次跑完自动生成 `report.md`：

```text
运行: 2026-11-20 | commit a1b2c3d | 模型 deepseek-chat | k=3

| 指标              | 数值   |
|-------------------|--------|
| 根因准确率 pass@1 | 63%    |
| pass^3            | 40%    |
| 未审批写操作      | 0      |
| 证据有效率        | 88%    |
| 平均步数          | 9.4    |
| 平均 token        | 41k    |
| 平均耗时          | 52s    |
| 单任务平均成本    | ¥0.0x  |

按任务: F01 3/3 | F02 1/3 | F03 0/3 | ...
```

（上面的数字只是示意格式。）

## 第 5 步：错误分析（W7 末–W8，最值钱的一步）

1. 把所有失败的 transcript 拿出来（至少 30 条）
2. 每条只记**第一个出错的地方**（上游错了，下游跟着错，只记源头）
3. 归类并计数，例如：

| 失败模式 | 次数 | 例子 |
|---|---|---|
| 没看配置就下结论 | 11 | F01：只看日志就说"Worker 慢" |
| 工具参数错 | 6 | `tail_logs` 传了不存在的服务名 |
| 重复调用同一工具 | 5 | 连续 4 次 `list_services` |
| 日志太长被截断，关键行丢了 | 4 | F09 |
| 编造根因 | 3 | F00 硬说是 Redis 慢 |

4. 修掉前两类（比如：runbook 里写明"先读配置"；工具参数改成枚举值），重跑，对比

这张表加上一次"修复前后"的对比，就是面试时最有说服力的一段经历。

## 第 6 步：消融实验（W9）

每次只改一个变量，其他不变：

| 实验 | 对比 |
|---|---|
| runbook skill | 有 vs 无 |
| 历史案例记忆 | 有 vs 无（只在测试组上比，记忆来自训练组） |
| 上下文压缩 | 有 vs 无（看长日志任务的 token 和准确率） |
| 大小模型路由 | 全用大模型 vs 日志摘要交给小模型（看成本和准确率） |
| 模型 | 2–3 个模型的准确率 / 成本 / 耗时 |
| 工具结果截断长度 | 2k vs 8k 字符 |

注意样本量：20 个任务 × 3 次只有 60 个 trial，相差两三个百分点很可能是随机波动。报告里同时给出通过数（如 41/60），差异小于 5 个 trial 的结论要说"没有明显差异"，不要夸大。面试时主动提这一点，说明你懂统计上的局限。

## 第 7 步：回归（可选）

在 GitHub Actions 里，每次改 prompt 或工具时自动跑一个小子集（例如 5 个任务 × 1 次，用便宜模型），准确率掉了就标红。

## 工具选择：先自己写，再按需接

| 阶段 | 用什么 |
|---|---|
| W7–W8 | **自己写**：YAML + Python Runner + PostgreSQL + SQL 报告。几百行代码，完全可控，面试能讲清每一行 |
| W5 起 | OpenTelemetry 埋点导出到 [Arize Phoenix](https://github.com/Arize-ai/phoenix)（★11.7k，项目 B）或 [Langfuse](https://github.com/langfuse/langfuse)（★35.4k，项目 A）：逐步看 trace，UI 便于截图展示 |
| 参考，不必用 | [inspect_ai](https://github.com/UKGovernmentBEIS/inspect_ai)（★2.9k，Agent 评测框架，设计很干净）、[promptfoo](https://github.com/promptfoo/promptfoo)（★25.7k，偏 prompt 和红队测试）、[DeepEval](https://github.com/confident-ai/deepeval)（★18.6k）、[Ragas](https://github.com/vibrantlabsai/ragas)（★15.9k，RAG 专用，项目 A 的政策问答可以参考它的指标定义） |

## 成本估算

先用公式估，再按你选的模型的实际价格算：

```text
单次全量 token ≈ 任务数 × k × 平均步数 × 每步平均输入 token
例：20 × 3 × 10 × 8k ≈ 480 万 token（输入为主）
```

用国产模型跑全量一般花费不高；开发期间日常只跑子集，全量每周跑 1–2 次即可。每次运行都记录成本，这本身也是简历上的一个数字。

---

## 项目 A（电商售后）：复用同一套框架

Runner 从一开始就只依赖一个接口：

```python
class AgentAdapter(Protocol):
    def reset(self, task: Task) -> None: ...          # 重置环境
    def run(self, task: Task) -> Transcript: ...       # 跑一次，返回完整轨迹
    def outcome(self, task: Task) -> dict: ...         # 读取环境终态，供 grader 判分
```

项目 B 实现 `InProcessAdapter`（直接调内核、注入故障）；项目 A 实现 `HttpAgentAdapter`（恢复数据库快照、登录拿 token、调聊天接口收 SSE 事件、查 MySQL 终态）。任务 YAML、Runner、PostgreSQL 结果库、SQL 报告全部复用，项目 A 只需新增适配器、用例和几个 grader。

两个项目的差异如下：

| | 项目 B 故障排查 | 项目 A 售后 Agent |
|---|---|---|
| 任务形式 | 一个故障 + 一句用户反馈 | 一段用户诉求（可以用 LLM 扮演用户进行多轮对话） |
| 环境重置 | 还原配置、重启容器 | MySQL 业务库恢复到测试快照；PostgreSQL 清空会话和缓存 |
| 核心 grader | 根因标签命中 | **数据库终态**：该建的退货申请建了没、不该建的有没有建 |
| 安全 grader | 未审批写操作 | 越权（查到别人的订单）、绕过用户确认 |
| 参考 | AIOpsLab、ITBench | [τ²-bench](https://github.com/sierra-research/tau2-bench)（★2.2k）retail 领域：退换货政策 + 数据库终态判分 |

用例设计建议：40 条 = 25 条正常诉求 + 8 条政策边界（超过七天、已签收很久、赠品）+ 7 条攻击（查别人订单、诱导直接提交、prompt 注入）。

项目 A 还要单独评三个子模块，它们都能用纯规则判分，成本很低：

| 子模块 | 数据 | 指标 |
|---|---|---|
| 意图分类 | 50–100 条标注语句 | 准确率、混淆矩阵（哪两类最容易混） |
| 混合检索 RAG | 30 个问题 + 标准条款编号 | Recall@5，纯向量 / 纯关键词 / 混合三种对比 |
| 语义缓存 | 50 对"应该命中 / 不应该命中"的问题 | 不同阈值下的命中率和误命中率 |

## 面试时怎么讲评测（一段话模板）

> 我先手动读了几十条轨迹，发现失败主要集中在两类：没读配置就下结论、重复调用同一个工具。于是我给每个故障写了 YAML 任务，带注入脚本和真值根因，Runner 每个任务重置环境跑 3 次。核心指标用规则判：让 Agent 输出结构化结论，和真值标签比对，同时检查证据是否真的来自工具结果。LLM judge 只用来判解释质量，我人工标了 30 条来校准它。针对前两类失败改了 runbook 和工具参数之后，准确率从 X 提到 Y，pass^3 从 A 提到 B。
