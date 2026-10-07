# 备选项目：如果不想做现在这两个

所有仓库都在 2026-10-05 通过 GitHub API 核实过：存在、仍在维护（已归档的会标注），星标数为当天数据。

## 一、什么时候还能换

| 要换的项目 | 最晚决定时间 | 原因 |
|---|---|---|
| 项目 B（纯 Agent） | **W3 周日（10/25）** | W4 开工；之后再换会损失已经写好的靶场和工具 |
| 项目 A（Java） | **W9 周日（12/06）** | W10 开工；周六 Java 实验做的是通用技能（并发、双数据源、Redis 幂等、MQ、Spring AI），换方向基本不浪费 |

过了这两个时间点就不要再换项目，进度不够时按 PLAN.md 的"落后时怎么砍"缩小范围。

**换项目时不变的部分**：时间线、阶段闸门、PRACTICES.md、评测框架的设计（适配器接口）、八股周表、简历骨架的结构。**要重写的部分**：对应的 PROJECT.md 和架构图。决定换的时候告诉我，我按新方向重写这两样。

## 二、怎么判断一个项目值不值得做

| 标准 | 要问的问题 |
|---|---|
| 场景真实 | 有真实的人会用吗？直接问 ChatGPT 能不能解决？ |
| 能评测 | 有没有客观的标准答案，能自动判对错、出数字？ |
| 复用已有资产 | 能用上你的 harness 内核、通信层、PostgreSQL 知识、评测框架吗？ |
| 面试差异化 | 面试官是不是已经见过一百遍了？ |
| 本机能跑 | 14GB 内存能不能撑住？ |
| 和八股互相加强 | 做项目的过程是不是在顺便复习面试会问的知识？ |

下面每个备选都按这几条评估过。

---

## 三、Java 线备选（替换项目 A）

### J1：Java 版大模型网关 ★ 最推荐的备选

**一句话**：一个兼容 OpenAI 接口的大模型代理服务，业务方只接它一个地址，它负责多模型路由、故障切换、按 token 限流、语义缓存、计费和审计。

**为什么值得做**：这是后端味最浓的 AI 项目。高并发、限流、熔断、流式代理、计费一致性全部覆盖；国内 one-api / new-api 用得很广，面试官熟悉这个场景，能聊得很深。你学过的 SSE 和 Nginx 流式代理正好用上。

**核心功能**：

- `/v1/chat/completions` 代理，流式 SSE 原样透传，首字延迟不增加
- 多供应商适配：OpenAI 兼容格式和 Anthropic 格式之间互相转换
- **按 token 限流**：请求前按 `max_tokens` 预扣额度，流式结束后按上游返回的真实用量结算、多退少补（Redis Lua 脚本保证原子性）。这是最好的面试点
- 故障切换：上游超时或报 5xx 时按优先级切到备用模型，配合熔断（Resilience4j）
- 语义缓存：PostgreSQL + pgvector
- 多租户 API Key、配额、计费明细、审计（PostgreSQL）
- 可观测：Prometheus + Grafana

**怎么评测**：自己写一个模拟上游的服务（可控延迟和错误率，压测不花模型钱），然后测：网关额外增加的 P99 延迟、单机 QPS、故障切换耗时、限流精度、计费与上游用量的误差、缓存命中率和误命中率。

**参考**：

| 仓库 | ★ | 看什么 |
|---|---|---|
| [QuantumNous/new-api](https://github.com/QuantumNous/new-api) | 49.3k | 国内最流行的模型聚合分发系统（Go），看功能清单和计费设计 |
| [songquanpeng/one-api](https://github.com/songquanpeng/one-api) | 37.1k | new-api 的前身，渠道管理和额度模型 |
| [higress-group/higress](https://github.com/higress-group/higress) | 9.5k | 阿里开源的 AI 网关（Go），看 token 限流和缓存插件的设计 |
| [Portkey-AI/gateway](https://github.com/Portkey-AI/gateway) | 13.1k | TS 写的 AI 网关，看路由、重试、护栏 |
| [BerriAI/litellm](https://github.com/BerriAI/litellm) | 60.1k | Python 网关，看多供应商格式转换 |
| [apache/shenyu](https://github.com/apache/shenyu) | 8.8k | Java 原生 API 网关，看插件架构 |

**风险**：Agent 成分少，**必须和一个纯 Agent 项目搭配**（比如现在的项目 B）。流式代理最好用 WebFlux，响应式编程有学习成本。

### J2：运营数据问答（安全 NL2SQL / ChatBI），仍基于 mall

**一句话**：运营人员问"上周退货率最高的三个品类"，系统生成 SQL、安全执行、返回表格和图表，并解释用的是哪条 SQL。

**和现在方案的关系**：底座还是 mall，只是把"售后 Agent"换成"运营数据 Agent"，双库、周六实验全部照用。

**核心设计**：

- **语义层**：用 YAML 定义指标和维度（"退货率 = 退货申请数 / 订单数"），模型基于语义层生成 SQL，而不是直接猜表结构
- **SQL 安全**：用 JSqlParser 把 SQL 解析成语法树后校验：只允许 SELECT、表和字段白名单、强制 LIMIT、禁止访问用户隐私字段；用只读账号 + 语句超时执行
- 结果解释：返回 SQL 和指标口径，让人能核对

**怎么评测**：50 个问题 + 标准 SQL，比较两条 SQL 的**执行结果**是否一致（执行准确率）；再加一批攻击用例（DROP、注入、查隐私字段）统计拦截率。评测非常客观。

**参考**：

| 仓库 | ★ | 看什么 |
|---|---|---|
| [tencentmusic/supersonic](https://github.com/tencentmusic/supersonic) | 5.1k | 腾讯音乐开源的 Java ChatBI，语义层设计的最好参考 |
| [spring-ai-alibaba/DataAgent](https://github.com/spring-ai-alibaba/DataAgent) | 2.7k | Spring AI Alibaba 官方的 Java 数据 Agent |
| [Canner/WrenAI](https://github.com/Canner/WrenAI) | 17.8k | 语义层驱动的 Text-to-SQL |
| [JSQLParser/JSqlParser](https://github.com/JSQLParser/JSqlParser) | 6.0k | **直接用**：SQL 解析与校验 |
| [xlang-ai/Spider2](https://github.com/xlang-ai/Spider2) | 0.9k | 企业级 Text-to-SQL 基准，看评测方法 |

**风险**：Text-to-SQL 是热门赛道，差异化必须落在"安全校验 + 语义层 + 评测"上，而不是"能生成 SQL"。

### J3：分布式任务调度平台 + 运维 Agent（基于 xxl-job）

**一句话**：在 xxl-job 上加任务失败自动诊断（读执行日志、判断原因、建议重试或修改参数，重跑需审批）和自然语言创建任务（生成 cron 表达式并校验）。

**参考**：[xuxueli/xxl-job](https://github.com/xuxueli/xxl-job)（★30.6k，代码量适中，分布式调度是面试常考点）、[PowerJob/PowerJob](https://github.com/PowerJob/PowerJob)（★7.8k）。

**风险**：xxl-job 官方已经带了 `xxl-job-executor-sample-springboot-ai` 示例，原创性要打折扣；而且和项目 B 一样都是"运维诊断"，**不能和项目 B 同时做**。更适合作为"项目 B 的 Java 版本"来替换项目 B。

### J4：审批流预审 Agent（基于 ruoyi-vue-pro 的 BPM）

**一句话**：在报销、采购等审批流程里加一个 Agent 节点：读取单据和附件、对照报销制度检索、给出预审意见和风险点，审批人参考后决定。

**参考**：[YunaiV/ruoyi-vue-pro](https://github.com/YunaiV/ruoyi-vue-pro)（★39.5k，自带基于 Flowable 的 BPM 工作流）。

**风险**：ruoyi-vue-pro 已有 AI 模块（对话、知识库等），你要讲清楚自己做的是"审批流程里的 Agent 节点"，不是聊天；代码库体量大，读懂成本高。

### J5：Halo 建站平台的 AI 插件

**一句话**：给 Halo 写一个"站内知识库问答"插件，发布到 Halo 应用市场，可能有真实用户。

**参考**：[halo-dev/halo](https://github.com/halo-dev/halo)（★39.9k，Java）、官方 [halo-dev/plugin-ai-foundation](https://github.com/halo-dev/plugin-ai-foundation)（AI 基础插件，还很早期）。

**风险**：插件 API 有学习成本，需要先调研官方 AI 基础插件已经覆盖了什么，避免重复。真实用户是最大亮点，但不确定性也最大。

---

## 四、纯 Agent 线备选（替换项目 B）

### P1：数据库慢 SQL 诊断与索引优化 Agent ★ 最推荐的备选

**一句话**：给它一条慢 SQL 或一句"订单列表接口变慢了"，它自己查执行计划、找出问题、用"假设索引"验证优化效果，给出索引建议或 SQL 改写，建索引前需要审批。

**为什么值得做**：

- **和八股深度绑定**：索引失效、最左前缀、回表、深分页这些 MySQL / PostgreSQL 高频面试题，就是这个 Agent 要诊断的故障。做项目就是在复习八股
- **评测完全客观**：优化前后的执行时间是真实测出来的
- **有现成的基线可以对比**：拿你的 Agent 和传统自动索引工具 Dexter 比，这个数字非常有说服力
- 复用你的 harness 内核、评测框架、PostgreSQL 知识

**工具**：读 `pg_stat_statements` 找最慢的 SQL；`EXPLAIN (ANALYZE, BUFFERS)`；读表结构、索引和统计信息；**HypoPG 创建假设索引后再看执行计划**（不真正建索引就能验证效果）；改写 SQL 后比对结果集是否一致；`CREATE INDEX` 需审批。

**靶场与故障**：PostgreSQL + 造百万行数据（可以把 mall 的订单表导进来），然后注入：缺索引、函数包裹列导致索引失效、隐式类型转换、前导 `%` 的 LIKE、OR 条件不走索引、深分页 OFFSET、统计信息过期、N+1 查询。每个故障的最优修复是已知的。

**怎么评测**：执行时间提升倍数、建议的索引是否最优（有没有冗余索引）、改写后结果集是否一致、是否出现未审批的 DDL；和 Dexter 的建议做对比。

**参考**：

| 仓库 | ★ | 看什么 |
|---|---|---|
| [HypoPG/hypopg](https://github.com/HypoPG/hypopg) | 1.7k | **直接用**：假设索引 |
| [ankane/dexter](https://github.com/ankane/dexter) | 2.1k | PostgreSQL 自动索引工具，**做评测基线** |
| [supabase/index_advisor](https://github.com/supabase/index_advisor) | 1.7k | 基于 HypoPG 的索引建议，看思路 |
| [TsinghuaDatabaseGroup/DB-GPT](https://github.com/TsinghuaDatabaseGroup/DB-GPT) | 0.7k | 清华的数据库诊断 Agent（D-Bot），有论文，看故障分类和诊断流程 |
| [eosphoros-ai/DB-GPT](https://github.com/eosphoros-ai/DB-GPT) | 20.1k | 数据库 AI 助手平台，了解 |

**和项目 A 搭配很好**：项目 A 的业务库是 MySQL，这个项目用 PostgreSQL，两个数据库都有深入实践。

### P2：Java 代码安全审计 Agent

**一句话**：先用 Semgrep 扫出候选漏洞（召回高、误报多），再让 Agent 读代码上下文（调用链、有没有过滤和校验）判断真假，输出报告和修复补丁，并跑测试验证补丁。

**为什么值得做**：安全方向的 Agent 项目很少；和 Java 结合；**有标准评测集**：OWASP Benchmark Java v1.2 有 2740 个标注好的测试用例（其中 1415 个是真漏洞，其余是故意构造的"看起来像漏洞但其实安全"的代码），能直接算真阳性率和误报率。核心数字是"只用 Semgrep"和"Semgrep + Agent"的误报率对比。

**参考**：

| 仓库 | ★ | 看什么 |
|---|---|---|
| [OWASP-Benchmark/BenchmarkJava](https://github.com/OWASP-Benchmark/BenchmarkJava) | 0.8k | **评测集** |
| [semgrep/semgrep](https://github.com/semgrep/semgrep) | 16.9k | **直接用**：静态分析，产生候选 |
| [The-PR-Agent/pr-agent](https://github.com/The-PR-Agent/pr-agent) | 13.3k | 开源 PR 审查 Agent，看怎么组织代码上下文 |
| [Aider-AI/aider](https://github.com/Aider-AI/aider) | 49.4k | 看代码仓库地图（repo map）怎么做 |

**风险**：OWASP Benchmark 是合成代码，要补几个真实开源项目的案例；全量跑 2740 个用例成本高，评测时按漏洞类别分层抽样 300 个即可。

### P3：MCP 安全网关

**一句话**：架在 Agent 和多个 MCP 服务之间的代理：统一认证、按工具设策略（允许 / 需审批 / 禁止）、参数级规则、审计、限流，检测工具描述投毒和返回内容里的 prompt 注入。

**怎么评测**：自建攻击集（投毒的工具描述、越权参数、带注入指令的返回内容），统计拦截率、误报率和网关增加的延迟。

**参考**：[IBM/mcp-context-forge](https://github.com/IBM/mcp-context-forge)（★4.6k）、[docker/mcp-gateway](https://github.com/docker/mcp-gateway)（★1.6k）、[snyk/agent-scan](https://github.com/snyk/agent-scan)（★3.1k，原 invariantlabs 的 mcp-scan）。

**风险**：MCP 规范还在快速变化；Agent 本身的成分不多，偏基础设施。不要和 J1 网关同时做。

### P4：Agent 长期记忆层（高风险）

**一句话**：给 Agent 做跨会话记忆：抽取、存储、检索、更新、遗忘，用公开基准评测。

**参考**：[mem0ai/mem0](https://github.com/mem0ai/mem0)（★66.6k）、[getzep/graphiti](https://github.com/getzep/graphiti)（★31.4k）、[letta-ai/letta](https://github.com/letta-ai/letta)（★25.0k）；评测基准 [LongMemEval](https://github.com/xiaowu0162/LongMemEval)（★1.1k，ICLR 2025）、[LoCoMo](https://github.com/snap-research/locomo)（★1.2k）。

**为什么标高风险**：[那篇模拟面试复盘](https://www.joyehuang.me/blog/20260512---agentmockinterview/post)里被问穿的正是"长期记忆引擎"项目。只有在公开基准上跑出可信数字、能和 mem0 对比时才值得做。

### P5：基于 Pi 二次开发（TypeScript）

**一句话**：用 Pi 的 RPC 模式加上你在 web_foundation 写的 SSE / WebSocket 服务，做一个"Pi 远程控制台"：浏览器里多会话管理、事件时间线、工具审批、断线续传；或者给 Pi 写审批 / 沙箱 / 记忆扩展。

**优点**：直接复用 `E:\pi_north` 的学习成果；如果能被 Pi 官方采纳或进入它的扩展生态，就是一段开源经历。

**风险**：用 TypeScript；核心是别人的，你的贡献边界要讲清楚；Pi 有贡献门槛（见它的 `CONTRIBUTING.md`），当前没有 good first issue。

### P6：面向你自己研究方向的文献调研 Agent

**一句话**：输入一个研究问题，Agent 检索 arXiv / Semantic Scholar，读论文，输出带引用的综述。

**为什么可以考虑**：研究生身份是天然优势，你就是这个领域的专家，能判断输出好坏，还能顺便帮到自己的科研。

**怎么评测**：引用核查（引用的论文是否真实存在、是否支持对应论点）、和人工综述对比覆盖率。

**参考**：[assafelovic/gpt-researcher](https://github.com/assafelovic/gpt-researcher)（★29.9k）；LangChain 的 `open_deep_research` 已归档，不建议参考。

**风险**："Deep Research"本身很热门，差异化必须来自垂直领域和引用核查。

---

## 五、怎么组合

两个项目要一个偏业务后端、一个偏 Agent 内部，并且不能重复。

| 组合 | 评价 |
|---|---|
| **A 售后 + B 故障排查**（现方案） | 业务 + 运维；复用资产最多 |
| **A 售后 + P1 慢 SQL 诊断** | 推荐的替换。MySQL + PostgreSQL 都有深度实践，八股加成最大 |
| **J1 网关 + B 故障排查** | 两个都偏基础设施，后端岗非常吃香，但"Agent 应用"成分弱一些 |
| **J1 网关 + P1 慢 SQL 诊断** | 硬核后端路线，面试官会很喜欢；缺少面向用户的 Agent 应用 |
| **J2 ChatBI + P2 代码审计** | 两个评测都非常客观；工作量偏大 |
| ~~J3 xxl-job + B 故障排查~~ | ✗ 都是运维诊断 |
| ~~J2 ChatBI + P1 慢 SQL~~ | ✗ 都围绕 SQL 生成和分析 |
| ~~J1 网关 + P3 MCP 网关~~ | ✗ 都是网关 |

## 六、补充路线：开源贡献

一两个被合并的 PR 能给简历加分，适合在项目之外穿插做，不替代项目。

我查了几个相关仓库当前带 `good first issue` 标签的开放问题数量：langchain4j 5 个，spring-ai-alibaba、HolmesGPT、inspect_ai 各 1 个，spring-ai 和 Pi 都是 0 个。这个标签的问题很少，所以更现实的做法是：

1. **在做项目的过程中，把遇到的真实问题提成 issue 或 PR**：文档错误、示例跑不通、边界情况的 bug。这类 PR 最容易被合并，也最好讲
2. 优先选你项目里真正用到的库：Spring AI、pgvector、langchain4j、HolmesGPT、inspect_ai
3. 提 PR 之前先读该仓库的 `CONTRIBUTING.md`，小改动先开 issue 确认维护者愿意接受

## 七、不推荐的方向

| 方向 | 为什么不推荐 |
|---|---|
| 通用知识库问答 / RAG 平台 | [Dify](https://github.com/langgenius/dify)（★158k）、[RAGFlow](https://github.com/infiniflow/ragflow)（★92k）、[MaxKB](https://github.com/1Panel-dev/MaxKB)（★23k）已经做得很成熟，面试官会直接问"和 Dify 比有什么不同" |
| 多 Agent 堆叠 | 说不清多个 Agent 相比单 Agent 的实际收益，是最常见的被问穿点 |
| 浏览器自动化 Agent | [browser-use](https://github.com/browser-use/browser-use)（★117k）很成熟；评测贵、不稳定；浏览器加模型在 14GB 内存上吃力 |
| 通用 Text-to-SQL | [vanna](https://github.com/vanna-ai/vanna) 已归档，WrenAI、DB-GPT 很成熟；除非像 J2 那样在安全和语义层上做出差异 |
| 通用编程 Agent | [OpenHands](https://github.com/OpenHands/OpenHands)（★90k）、[aider](https://github.com/Aider-AI/aider)（★49k）、Claude Code 都在；无法回答"为什么不直接用它们" |
| 套壳聊天应用 | 没有技术深度 |
