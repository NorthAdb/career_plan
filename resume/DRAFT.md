# 简历骨架（以终为始）

先把简历最终长什么样写出来，项目做的每件事都朝这里的某一句对齐。`[X]` 是占位数字，W17 换成真实测出来的值；做不出来的那句就删掉，**不写没测过的数字**。

两个版本只是项目顺序和技能栏顺序不同：投 Java 后端岗用版本 A，投 AI Agent 应用岗用版本 B。

---

## 版本 A：Java 后端开发（AI 方向）

**[姓名]** ｜ [电话] ｜ [邮箱] ｜ GitHub: [链接]

### 教育经历

**[学校]** ｜ [专业] 硕士 ｜ [入学年月] – [毕业年月]（预计）
- [研究方向一句话；如有论文 / 竞赛 / 奖学金写在这里]

### 专业技能

- **Java 后端**：熟悉 Java 集合、并发与 JVM；熟悉 Spring Boot、MyBatis；了解 Spring Cloud Alibaba 微服务组件
- **数据与中间件**：熟悉 MySQL 索引、事务与锁；熟悉 PostgreSQL 与 pgvector 向量检索；熟悉 Redis 缓存、分布式锁与限流；熟悉 RabbitMQ 可靠投递与幂等消费；了解 Elasticsearch
- **AI 应用**：熟悉 Spring AI、Tool Calling、结构化输出、RAG 混合检索（向量 + 全文 + RRF）、Agent 评测；了解 MCP、OpenTelemetry
- **工程**：Docker Compose、Nginx（含 SSE / WebSocket 代理）、Git、GitHub Actions；熟练使用 Cursor / Claude Code 进行 AI 辅助开发

### 项目经历

**电商智能售后 Agent** ｜ Java 17 · Spring Boot 3 · Spring AI · MySQL · PostgreSQL/pgvector · Redis · RabbitMQ ｜ [GitHub 链接]

基于开源电商系统 mall 二次开发，新增自然语言售后能力（订单查询、政策问答、退货申请、客服预审），采用"MySQL 业务库 + PostgreSQL AI 库"双库架构。

- 设计 Agent 工具层安全机制：用户身份由服务端从 JWT 注入、工具参数不含用户标识，[X] 条越权与注入用例全部拦截；写操作"草稿 + 用户确认"两阶段提交
- Redis `SET NX` + MySQL 唯一索引两层幂等，并发重放 [X] 次仅生成 1 条申请；退款经 RabbitMQ 异步执行，生产者确认 + 手动 ack + 消费去重
- 基于 pgvector + 中文全文检索构建混合检索 RAG，RRF 融合排序，Recall@5 从纯向量的 [X]% 提升到 [X]%，回答附条款引用
- 小模型意图路由（准确率 [X]%）+ 语义缓存（命中率 [X]%），单次会话平均成本下降 [X]%
- 商品数据经本地消息表 + RabbitMQ 增量同步至 PostgreSQL，按版本号幂等更新并每日对账
- 构建 [X] 条售后评测集（判数据库终态、越权、政策遵守），错误分析迭代后任务成功率从 [X]% 提升到 [X]%

**服务故障排查 Agent** ｜ Python · FastAPI · PostgreSQL/pgvector · Redis · Docker · OpenTelemetry ｜ [GitHub 链接]

自研 Agent harness，面向 Nginx + 多 Worker + Redis 拓扑自动定位故障根因，并构建可复现的故障注入评测系统。

- 自研 Agent 内核：循环与预算控制、结构化诊断输出、上下文压缩、按需加载的 runbook skills、hooks 拦截与 WebSocket 人工审批
- 构建 [X] 类注入故障的评测系统（含无故障对照与 prompt 注入用例），规则判分 + 人工校准的 LLM judge（一致率 [X]%）；错误分析迭代后根因准确率从 [X]% 提升到 [X]%
- 基于 pgvector 实现历史故障案例记忆，训练组 / 测试组隔离防评测泄漏，带来 [+X]% 准确率；大小模型路由使单任务成本下降 [X]%

---

## 版本 B：AI Agent 应用开发

教育经历相同。改动两处：

1. **专业技能**第一行换成 AI 应用，并补充"熟悉 Agent loop、上下文工程、长期记忆、Agent 评测方法（pass@k、LLM judge 校准、错误分析）"
2. **项目经历**把"服务故障排查 Agent"放第一个，并在它的描述里加一句"评测框架同时复用于 Java 售后 Agent 的评测"，把两个项目串成"评测驱动的 Agent 开发"这条主线

---

## 自查清单（每次改简历都过一遍）

- [ ] 一页纸
- [ ] 每个数字都能说出怎么测的、测试集多大，并且和 GitHub README 里的成绩表一致
- [ ] 每个技术名词都能回答"为什么用它、不用会怎样"
- [ ] 项目 A 写清楚"基于 mall 二次开发"，不把原项目的功能算成自己的
- [ ] GitHub 链接可以打开，README 首屏有架构图和成绩
- [ ] 没有"精通"二字
