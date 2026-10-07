# 开源项目与资料精选

> 本文件是现有两个项目要用到的参考。如果想换项目，备选方向和对应的参考仓库见 [ALTERNATIVES.md](ALTERNATIVES.md)。

所有仓库已在 2026-10-04 通过 GitHub API 核实存在且仍在维护，star 数为当天数据。每条都写了"怎么用它"，**不要通读源码**，按用途取用。

## 一、Java 业务底座

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [macrozheng/mall](https://github.com/macrozheng/mall) | 84.8k | **项目 A 的底座**。Spring Boot 3.5 + Java 17，买家端和客服端都有退货模块，带 MySQL、Redis、RabbitMQ、ES、MongoDB 的完整 Docker 环境 |
| [macrozheng/mall-learning](https://github.com/macrozheng/mall-learning) | 13.4k | mall 的配套教程，W3 跑项目、W10 读链路时按需查 |
| [macrozheng/mall-swarm](https://github.com/macrozheng/mall-swarm) | 13.2k | mall 的微服务版（Spring Cloud Alibaba）。**只用来看微服务长什么样**，不做项目 |
| [YunaiV/ruoyi-vue-pro](https://github.com/YunaiV/ruoyi-vue-pro) | 39.5k | 企业级脚手架，带商城和 AI 模块。参考它的 AI 模块怎么在 Spring 里组织 |
| [YunaiV/yudao-cloud](https://github.com/YunaiV/yudao-cloud) | 19.6k | 上面那个的微服务版，同样只参考 |
| [linlinjava/litemall](https://github.com/linlinjava/litemall) | 20.4k | 更轻的商城，备选 |
| [newbee-ltd/newbee-mall](https://github.com/newbee-ltd/newbee-mall) | 11.6k | 最简单的商城，如果 mall 跑起来太吃力，可以先拿它热身 Spring Boot |

## 二、Java + AI

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [spring-projects/spring-ai](https://github.com/spring-projects/spring-ai) | 9.5k | **项目 A 用它**。Spring 官方，工具调用、结构化输出、JDBC 会话记忆（支持 PostgreSQL）、PgVectorStore、MCP、Micrometer 可观测都有 |
| [alibaba/spring-ai-alibaba](https://github.com/alibaba/spring-ai-alibaba) | 11.0k | 阿里基于 Spring AI 的 Agent 框架，国内面试会被问到，能对比即可 |
| [langchain4j/langchain4j](https://github.com/langchain4j/langchain4j) | 13.2k | Java 版 LangChain，能对比即可 |
| [agentscope-ai/agentscope-java](https://github.com/agentscope-ai/agentscope-java) | 5.9k | 阿里 AgentScope 的 Java 版，了解 |
| [nageoffer/ragent](https://github.com/nageoffer/ragent) | 4.2k | Java 写的企业级 Agentic RAG（文档解析、多路检索、意图识别、MCP）。做项目 A 的 RAG 部分时参考它的模块划分 |

## 三、PostgreSQL 与向量检索

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [pgvector/pgvector](https://github.com/pgvector/pgvector) | 23.2k | **两个项目都用**。README 本身就是最好的教程：向量类型、距离运算符、HNSW / IVFFlat 索引、混合检索示例 |
| [pgvector/pgvector-python](https://github.com/pgvector/pgvector-python) | 1.5k | 项目 B 用：psycopg / SQLAlchemy 里怎么读写向量 |
| [amutu/zhparser](https://github.com/amutu/zhparser) | 0.9k | PostgreSQL 中文分词扩展，混合检索的关键词那一路 |
| [paradedb/paradedb](https://github.com/paradedb/paradedb) | 9.3k | 在 PostgreSQL 里做 BM25 全文检索的方案，了解；面试被问"PG 全文检索不够好怎么办"时可以提 |
| [supervc-stack/VectorChord](https://github.com/supervc-stack/VectorChord) | 1.8k | pgvector 的高性能替代，了解；被问"数据量大了 pgvector 扛不住怎么办"时可以提 |
| [dimitri/pgloader](https://github.com/dimitri/pgloader) | 6.5k | MySQL → PostgreSQL 一键迁移工具。当前方案用不到，被问"为什么不全迁 PG"时知道有这个工具和代价 |

## 四、中间件（看官方文档，不读源码）

| 项目 | ★ | 用途 |
|---|---|---|
| [apache/rocketmq](https://github.com/apache/rocketmq) | 22.6k | 阿里系 MQ，面试和 RabbitMQ、Kafka 对比 |
| [alibaba/nacos](https://github.com/alibaba/nacos) | 33.4k | 注册中心 + 配置中心 |
| [alibaba/Sentinel](https://github.com/alibaba/Sentinel) | 23.1k | 限流熔断 |
| [apache/incubator-seata](https://github.com/apache/incubator-seata) | 26.0k | 分布式事务 |
| [alibaba/spring-cloud-alibaba](https://github.com/alibaba/spring-cloud-alibaba) | 29.2k | 上面几个的整合 |

## 五、Agent harness 参考

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) | 78.0k | 你已经学过（agent_l） |
| [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | 8.2k | 约 100 行的 Agent + SWE-bench 评测。看"内核极简、评测严肃"怎么配合 |
| [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) | 15.9k | Python 版 Claude Code 式 harness，和你的内核逐模块对照 |
| [openai/openai-agents-python](https://github.com/openai/openai-agents-python) | 29.8k | 看它的 tracing、guardrails、handoff 设计；面试时作为"为什么不用框架"的对比对象 |
| Pi（`E:\pi_north`） | — | 你正在读的样本，重点看 `packages/agent`、`packages/coding-agent`，评测看 `packages/evals` |

## 六、运维 / SRE Agent（项目 B 的场景参考）

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [HolmesGPT/holmesgpt](https://github.com/HolmesGPT/holmesgpt) | 3.5k | CNCF Sandbox 的 SRE Agent，和项目 B 最像。看 toolset 和 runbook 的组织方式 |
| [k8sgpt-ai/k8sgpt](https://github.com/k8sgpt-ai/k8sgpt) | 8.2k | K8s 诊断工具（Go），看它的"分析器 + LLM 解释"思路 |
| [microsoft/AIOpsLab](https://github.com/microsoft/AIOpsLab) | 1.0k | 故障注入 + AIOps Agent 评测框架，看问题定义和判分 |
| [itbench-hub/ITBench](https://github.com/itbench-hub/ITBench) | 0.5k | IT 自动化 Agent 基准 |
| [Shopify/toxiproxy](https://github.com/Shopify/toxiproxy) | 12.4k | **项目 B 直接用**：在服务之间注入延迟、断连 |
| [open-telemetry/opentelemetry-demo](https://github.com/open-telemetry/opentelemetry-demo) | 3.4k | 带故障开关的微服务 demo，后期想换复杂靶场时考虑 |
| [chaos-mesh/chaos-mesh](https://github.com/chaos-mesh/chaos-mesh) | 7.9k | K8s 上的混沌工程，了解即可 |

## 七、评测、可观测与模型网关

| 项目 | ★ | 怎么用它 |
|---|---|---|
| [sierra-research/tau2-bench](https://github.com/sierra-research/tau2-bench) | 2.2k | **项目 A 评测的蓝本**：retail 领域是客服 Agent + 退换货政策 + 数据库终态判分 |
| [langfuse/langfuse](https://github.com/langfuse/langfuse) | 35.4k | 开源 trace 与评测平台，**项目 A 接入**（支持 OpenTelemetry 上报） |
| [Arize-ai/phoenix](https://github.com/Arize-ai/phoenix) | 11.7k | 开源 trace 与评测平台，**项目 B 接入**，本地起一个就能用 |
| [Arize-ai/openinference](https://github.com/Arize-ai/openinference) | 1.2k | OpenTelemetry 的 AI 埋点库，项目 B 参考它记录哪些字段 |
| [traceloop/openllmetry](https://github.com/traceloop/openllmetry) | 7.5k | 另一套 OpenTelemetry AI 埋点库，和上面二选一参考 |
| [open-telemetry/semantic-conventions](https://github.com/open-telemetry/semantic-conventions) | 0.7k | OpenTelemetry 官方语义约定，里面的 GenAI 部分定义了模型调用该记哪些属性 |
| [BerriAI/litellm](https://github.com/BerriAI/litellm) | 60.1k | 统一调用上百种模型的网关。项目 B 自己写 provider 抽象，拿它做参考和面试对比 |
| [UKGovernmentBEIS/inspect_ai](https://github.com/UKGovernmentBEIS/inspect_ai) | 2.9k | 设计很干净的 Agent 评测框架，参考它的 task / solver / scorer 抽象 |
| [promptfoo/promptfoo](https://github.com/promptfoo/promptfoo) | 25.7k | prompt 回归和红队测试，参考它的攻击用例 |
| [confident-ai/deepeval](https://github.com/confident-ai/deepeval) | 18.6k | LLM 评测框架，了解 |
| [vibrantlabsai/ragas](https://github.com/vibrantlabsai/ragas) | 15.9k | RAG 评测指标，项目 A 政策问答参考 |

## 八、协议

| 项目 | ★ | 用途 |
|---|---|---|
| [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | 24.5k | 项目 B 写 MCP server |
| [ag-ui-protocol/ag-ui](https://github.com/ag-ui-protocol/ag-ui) | 16.3k | Agent 前端事件协议，你在 web_foundation 已对齐过 |

## 九、面试

| 项目 | ★ | 用途 |
|---|---|---|
| [Snailclimb/JavaGuide](https://github.com/Snailclimb/JavaGuide) | 159k | 八股主线 |
| [doocs/advanced-java](https://github.com/doocs/advanced-java) | 79.1k | 高并发、分布式进阶 |
| [youngyangyang04/leetcode-master](https://github.com/youngyangyang04/leetcode-master) | 62.6k | 代码随想录，刷题专题路线 |
| [xiaolincoder/CS-Base](https://github.com/xiaolincoder/CS-Base) | 18.5k | 图解计网、OS、MySQL、Redis |

## 十、必读文章

| 文章 | 为什么读 |
|---|---|
| [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)（Anthropic） | Agent 评测的术语和方法论，EVAL.md 的基础 |
| [AI Evals FAQ](https://hamel.dev/blog/posts/evals-faq/)（Hamel Husain） | "先做错误分析再写评测"，以及怎么校准 LLM judge |
| [Your AI Product Needs Evals](https://hamel.dev/blog/posts/evals/)（Hamel Husain） | 评测体系在真实产品里长什么样 |
| [一场 1 小时 19 分钟的 Agent 工程师模拟面试](https://www.joyehuang.me/blog/20260512---agentmockinterview/post) | 面试官视角：堆技术的项目怎么被问穿 |
