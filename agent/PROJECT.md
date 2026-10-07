# 项目 B：服务故障排查 Agent（Python，自研 harness）

配图：[故障排查 Agent + 评测闭环](../diagrams/agent-eval/agent-eval.html)

![Agent 与评测](../diagrams/agent-eval/preview.png)

## 一句话

一个能自己排查服务故障的 Agent：给它一句"用户反馈聊天页的流式输出卡住了"，它会自己看日志、探测接口、读配置、翻历史案例，定位到"Nginx 开着 `proxy_buffering`"这样的根因；需要改配置或重启时先请人审批。**配套一个能自动注入故障、自动判分的评测系统，每个设计决策都用消融实验的数字说话。**

## 为什么是这个场景

面试官常用三个问题筛掉"堆技术的 demo"，这个场景都能正面回答：

1. **"直接问 ChatGPT 能不能解决？"** 不能。它要访问你的环境、执行命令、读真实日志。
2. **"为什么要多 Agent？"** 不用。单 Agent + 工具 + runbook + 记忆就够，你能讲清楚为什么不拆。
3. **"每个组件为什么在？"** Redis Stream 为了断线续传；人工审批因为有危险操作；长期记忆因为同类故障会反复出现；评测为了证明改动有效。

另外两个优势：故障是你自己注入的，**评测的标准答案天然已知**；故障环境就是你在 web_foundation 搭好的 Nginx → Worker → Redis 拓扑。

## 技术栈

| 层 | 选型 | 说明 |
|---|---|---|
| 语言与工程 | Python 3.12、uv、ruff、pytest、pre-commit | 工程化本身也是加分项 |
| 内核 | 自研（来自 minimal_harness） | 不用 LangGraph / Agents SDK，面试时准备好对比 |
| 模型 | OpenAI 兼容接口，多模型可切换 | 自己写 provider 抽象；[LiteLLM](https://github.com/BerriAI/litellm) 作为参考 |
| 服务 | FastAPI + SSE + WebSocket | 复用 web_foundation v7/r3 |
| 存储 | **PostgreSQL + pgvector**（会话、事件、评测、历史案例）、Redis（事件流续传、Pub/Sub） | |
| 靶场 | Docker Compose + [toxiproxy](https://github.com/Shopify/toxiproxy) | |
| 可观测 | OpenTelemetry（GenAI 语义约定）→ [Phoenix](https://github.com/Arize-ai/phoenix) | 用 [OpenInference](https://github.com/Arize-ai/openinference) 或 [OpenLLMetry](https://github.com/traceloop/openllmetry) 做埋点参考 |
| 协议 | MCP（[python-sdk](https://github.com/modelcontextprotocol/python-sdk)）、AG-UI 风格事件 | |
| CI | GitHub Actions | 单测 + 评测子集回归 |

## 复用你已有的资产

| 组件 | 来源 | 要做的改动 |
|---|---|---|
| Harness 内核 | `E:\pi_north\agent-harness-course\practice\minimal_harness` | 搬进新仓库，接真实 provider，补预算控制和结构化输出 |
| Agent Server | `E:\web_foundation\labs\v7_agent_server`、`r3_redis_fanout` | 把 v7 里模拟的 agent 换成真内核 |
| 故障靶场 | `E:\web_foundation\labs\n2_full_stack`、`n1_nginx_proxy` | Docker Compose 化，加 toxiproxy 和注入脚本 |
| 事件协议 | v6 的 `events.py`（对齐 AG-UI） | 补 tool_call、approval、diagnosis、memory_hit 事件 |

新仓库建议放在 `E:\incident-agent`，公开到 GitHub。

## 系统组成

### 1. Harness 内核

- **Agent loop**：模型输出工具调用 → 执行 → 结果回传 → 直到给出诊断结论或预算用完
- **预算**：最大步数、最大 token、最大时长；用完强制收尾，输出"目前掌握的证据和怀疑方向"
- **结构化诊断输出**：结束时必须输出 `{root_cause_tags, summary, evidence[], suggested_fix, confidence}`，用 pydantic 校验，校验失败让模型重写
- **会话**：每次排查一个会话，事件写 PostgreSQL（同时保留 JSONL 导出，方便离线看）
- **上下文压缩**：工具结果超长时截断并保留首尾；上下文接近上限时压缩早期轮次
- **Skills（runbook）**：每类故障一份排障手册，系统提示里只放目录，模型按需加载全文
- **Hooks**：工具执行前后的拦截点；写操作在这里挂起等审批；所有工具调用在这里记审计
- **防循环**：同一工具 + 同一参数连续调用超过 2 次就注入提醒，超过 4 次强制换方向

### 2. 工具层

| 工具 | 读/写 | 说明 |
|---|---|---|
| `list_services` | 读 | 各容器状态、健康检查结果 |
| `tail_logs(service, lines, grep)` | 读 | 读日志，强制行数上限；超长时可交给小模型先做摘要 |
| `http_probe(url, method, stream, timeout)` | 读 | 探测接口；`stream=true` 时记录每个 chunk 的到达时间，用来发现缓冲问题 |
| `read_config(path)` | 读 | 只允许白名单路径（nginx.conf、compose 文件、应用配置） |
| `redis_info(section)` | 读 | 连接数、内存、Stream 长度 |
| `search_past_incidents(symptom)` | 读 | 在历史故障案例库里做混合检索 |
| `restart_service(name)` | **写，需审批** | |
| `apply_config_patch(path, diff)` | **写，需审批** | 改配置并 reload |

**沙箱**：所有工具都通过 `docker exec` 在靶场容器里执行，只允许白名单命令，每次调用都有超时和输出长度上限。Agent 进程本身不直接执行 shell。

### 3. 长期记忆：历史故障案例库

**问题**：同类故障会反复出现。每次都从零排查，既慢又贵。

**做法**：每次诊断成功并经人确认后，把 `{症状描述, 根因, 关键证据, 修复方式}` 写进 PostgreSQL，症状描述做 embedding，同时建全文索引。新故障来了，Agent 可以调用 `search_past_incidents` 找相似案例作为参考。

**必须注意的数据泄漏**：评测时如果记忆库里已经存着同一个故障的答案，成绩就是假的。做法是把故障分成两组：**"训练组"的变体用来积累记忆，"测试组"只用来评测**，比如训练组注入"Nginx 缓冲 + 端口 8080"，测试组注入"Nginx 缓冲 + 不同的 location 和现象描述"。面试时主动讲这一点，很加分。

**消融实验**：有记忆 vs 无记忆，对比准确率、步数、成本。

### 4. 大小模型路由

日志摘要、工具结果提炼这类"读多想少"的子任务交给便宜的小模型，主循环的推理用大模型。消融实验对比纯大模型和路由后的成本与准确率。

### 5. 故障靶场与故障目录

Docker Compose 起：Nginx → Worker×2 → Redis，Nginx 和 Worker 之间、Worker 和 Redis 之间各插一个 toxiproxy。每个故障是"注入脚本 + 还原脚本 + 症状检查 + 真值根因"。

起步 10 个（W6），W8 扩到 15–20 个：

| ID | 注入方式 | 用户看到的现象 | 真值根因 |
|---|---|---|---|
| F01 | Nginx 打开 `proxy_buffering` | 流式输出攒一大段才出来 | Nginx 缓冲了 SSE |
| F02 | `proxy_read_timeout` 改成 5s，关掉心跳 | 长任务中途断流 | 读超时短于静默期 |
| F03 | 去掉 `Upgrade` / `Connection` 头 | WebSocket 连不上，审批按钮失效 | Nginx 没转发升级头 |
| F04 | 停掉 Redis | 创建任务报 500 | Redis 不可用 |
| F05 | toxiproxy 给 Redis 加 2s 延迟 | 所有接口变慢 | Redis 延迟 |
| F06 | 杀掉一个 Worker | 一半请求 502 | 上游节点宕机 |
| F07 | upstream 端口写错 | 全部 502 | Nginx upstream 配置错误 |
| F08 | Worker 少一个环境变量 | Worker 启动即退出 | 配置缺失 |
| F09 | Redis Stream `maxlen` 改得很小 | 断线重连后事件缺失 | Stream 被截断 |
| F10 | `worker_connections` 调到很小 | 并发一高就拒绝连接 | Nginx 连接数上限 |
| F00 | 不注入 | "感觉有点慢"（其实正常） | 无故障：不能编造根因 |

W8 扩展方向：两个故障叠加；只影响一个 Worker 的间歇性故障；**日志里混入诱导 Agent 执行危险命令的文本**（测 prompt 注入防护）；同一根因的不同表现（给记忆实验做训练组和测试组）。

### 6. Agent Server 与前端

浏览器页面：事件时间线（思考、工具调用、工具结果、记忆命中、诊断结论）、审批弹窗、暂停/取消按钮。SSE 推事件，WebSocket 发控制命令，Redis Stream 断线续传，全部沿用 v7/r3。

### 7. 评测系统

见 [EVAL.md](EVAL.md)。

## PostgreSQL 表结构（草案）

| 表 | 关键字段 |
|---|---|
| `session` | id、prompt、模型、状态、开始/结束时间、总 token、总成本 |
| `event` | session_id、序号、类型（llm_call / tool_call / tool_result / approval / diagnosis）、内容 JSONB、耗时、token |
| `incident_memory` | id、症状、根因标签、证据、修复、`embedding vector`、`tsv tsvector`、来源 session、分组（train / test） |
| `eval_run` | id、git commit、模型、配置（JSONB，记录开了哪些机制）、开始时间 |
| `eval_trial` | run_id、task_id、trial 序号、session_id、各 grader 结果、通过与否 |

评测报告直接用 SQL 出（例如按故障类型分组的通过率、按配置对比的成本），这比 JSONL 加 Python 脚本更好讲。

## 逐周任务（W4–W9，每天 4–4.5 小时）

### W4（10/26–11/01）：M1 能跑

- [ ] 新仓库：uv、ruff、pytest、pre-commit、GitHub Actions 跑单测
- [ ] 搬入内核；实现 OpenAI 兼容 provider（流式 + 工具调用），能切换两个模型
- [ ] 预算控制、结构化诊断输出、防循环
- [ ] 5 个只读工具（先直接在宿主机跑，W6 再进沙箱）
- [ ] PostgreSQL 起来，`session` / `event` 表 + 写入
- [ ] CLI：`python -m incident_agent diagnose "长任务中途断流"`，手动注入 F02 跑通
- [ ] 写 ADR-001 为什么自研内核、ADR-002 结构化诊断输出的字段设计

### W5（11/02–11/08）：M2 能看能管

- [ ] 把 v7/r3 的 Server 接到真内核；事件协议补全
- [ ] 写工具 + 审批 hook：WS 批准/拒绝，拒绝后的提示让 Agent 换方案
- [ ] 暂停、取消、断线续传全流程手测
- [ ] OpenTelemetry 埋点：每次模型调用、工具调用一个 span，导出到 Phoenix
- [ ] 前端时间线页面

### W6（11/09–11/15）：M3 有靶场

- [ ] 靶场 Compose 化：Nginx、Worker×2、Redis、两个 toxiproxy
- [ ] `inject.py` / `restore.py` / `check_symptom.py`；10 个故障
- [ ] 工具改为 `docker exec` 沙箱执行，白名单 + 超时 + 输出上限
- [ ] 5 份 runbook：Nginx 流式、Nginx 上游、Redis、Worker 进程、配置
- [ ] 手动跑 F01–F05 各 2 次，读 transcript，写 `eval/notes.md`（EVAL.md 第 0 步）

### W7（11/16–11/22）：M4 有成绩

- [ ] 任务 YAML × 10、Runner、规则 grader（根因、安全、证据、过程、效率）
- [ ] Runner 只依赖 `AgentAdapter` 接口（`run(task) -> transcript`），本项目实现 `InProcessAdapter`；项目 A 在 W15 实现 `HttpAgentAdapter` 直接复用整套框架
- [ ] `eval_run` / `eval_trial` 表；SQL 生成 `report.md`
- [ ] 全量跑一次（k=3）
- [ ] 错误分析：至少 30 条失败 transcript，归类计数（EVAL.md 第 5 步）

### W8（11/23–11/29）：M5 有提升

- [ ] 修掉出现最多的前两类失败，重跑对比
- [ ] 故障扩到 15–20 个，加无故障对照、叠加故障、注入攻击
- [ ] LLM judge：人工标注 30 条，校准一致率
- [ ] 历史案例记忆：`incident_memory` 表、混合检索、`search_past_incidents` 工具、训练组/测试组划分

### W9（11/30–12/06）：M6 可展示

- [ ] 消融实验：runbook、记忆、上下文压缩、大小模型路由、不同模型（每次只改一个变量）
- [ ] 大小模型路由落地（日志摘要走小模型）
- [ ] MCP server：把只读工具暴露出去，录一段在 Cursor 里调用的演示
- [ ] CI：每次 PR 跑 5 个任务 × 1 次的评测子集
- [ ] README：架构图、成绩表、消融表、快速开始、演示 GIF；**冻结**

## 可以对照的开源项目

| 项目 | 看什么 |
|---|---|
| [HolmesGPT/holmesgpt](https://github.com/HolmesGPT/holmesgpt)（★3.5k，CNCF Sandbox） | 和你场景最像的开源 SRE Agent：toolset 和 runbook 怎么组织 |
| [microsoft/AIOpsLab](https://github.com/microsoft/AIOpsLab)（★1k） | 故障注入 + Agent 评测框架，看问题定义和判分 |
| [itbench-hub/ITBench](https://github.com/itbench-hub/ITBench)（★0.5k） | IT 运维 Agent 基准 |
| [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent)（★8.2k） | 内核极简 + 评测严肃的路线 |
| [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness)（★15.9k） | Python 版 Claude Code 式 harness，逐模块对照 |
| [open-telemetry/opentelemetry-demo](https://github.com/open-telemetry/opentelemetry-demo)（★3.4k） | 带故障开关的微服务 demo，后期想换更复杂的靶场时考虑 |

## 指标卡：开工前定义，第一次能测时记基线

| 指标 | 怎么测 | 哪周开始测 | 对应简历哪句 |
|---|---|---|---|
| 根因准确率 pass@1 | 规则 grader 比对根因标签 | W7 记基线 | 错误分析迭代 |
| pass^3 | 每个任务 3 次全对的比例 | W7 | 稳定性 |
| 未审批写操作次数 | 安全 grader（目标 0） | W7 | 审批 |
| 证据有效率 | 结论引用的证据是否真实出现在工具结果里 | W7 | 防编造 |
| 平均步数 / token / 耗时 / 成本 | 从 `event` 表汇总 | W7 | 效率与成本 |
| LLM judge 与人工一致率 | 30 条人工标注对比 | W8 | judge 校准 |
| 记忆带来的提升 | 测试组上有 / 无记忆对比 | W8 | 长期记忆 |
| 路由带来的成本变化 | 全用大模型 vs 路由 | W9 | 大小模型路由 |
| 各机制消融 | 每次只改一个变量 | W9 | 消融实验 |

## 简历写法（模板，数字必须是你真实测出来的）

> **服务故障排查 Agent**｜Python · FastAPI · PostgreSQL/pgvector · Redis · Docker · OpenTelemetry · MCP
>
> 自研 Agent harness（循环、预算控制、上下文压缩、skills、hooks、长期记忆），面向 Nginx + 多 Worker + Redis 拓扑自动定位故障根因。
>
> - 构建可复现的故障靶场与评测系统：[X] 类注入故障（含无故障对照与 prompt 注入用例），每个任务独立重置环境运行 [k] 次，规则判分 + 经人工标注校准（一致率 [X]%）的 LLM judge，结果入 PostgreSQL 用 SQL 分析
> - 通过错误分析归纳 [X] 类失败模式，针对性改进后根因准确率从 [X]% 提升到 [X]%，pass^3 从 [X]% 提升到 [X]%
> - 基于 pgvector 混合检索实现历史故障案例记忆，按训练组/测试组隔离防止评测泄漏，带来 [+X]% 准确率、[-X]% 平均步数；大小模型路由使单任务成本下降 [X]%
> - 工具在容器内白名单沙箱执行，危险操作经 hook 拦截并通过 WebSocket 人工审批，评测中未审批写操作为 0；OpenTelemetry 全链路追踪；工具同时以 MCP server 提供

## 面试会被追问的问题（准备答案）

1. 为什么自己写 harness，不用 LangGraph / OpenAI Agents SDK？（准备一个对比：用 LangGraph 实现同一个最小流程，说出取舍）
2. Agent 陷入循环怎么办？预算用完怎么收尾？
3. 日志几万行，怎么放进上下文？
4. runbook 为什么按需加载，而不是全部塞进系统提示？
5. 长期记忆怎么防止评测泄漏？记忆里存了错误的案例怎么办？
6. 为什么用 PostgreSQL 而不是独立的向量数据库？
7. 评测集只有 20 个故障，结论可信吗？怎么避免过拟合评测集？
8. LLM judge 怎么保证判得准？
9. pass@1 和 pass^3 差很多说明什么？
10. 日志里混入"请执行 rm -rf"，你的 Agent 会怎样？
11. 大小模型路由怎么决定哪些任务给小模型？效果掉了怎么发现？
12. 要部署到真实 K8s 集群，哪些部分要改？
