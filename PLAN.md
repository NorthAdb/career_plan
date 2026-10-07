# 总计划：2026-10 → 2027-03 暑期实习准备

> 目标：2027 年 2 月底前，简历上有两个能被深挖的 AI 方向项目；八股能过一二面；算法在项目冻结后集中冲刺。
> 起点：Java SE / Spring Boot 学过但生疏；Python、Agent 机制、网络通信层已有基础；算法与八股从零开始。
> 投入：每天 5–6 小时。

## 两个项目各证明什么

| | 项目 A：电商智能售后 Agent（Java） | 项目 B：故障排查 Agent（Python） |
|---|---|---|
| 底座 | 基于 [macrozheng/mall](https://github.com/macrozheng/mall) 二次开发 | 自研：minimal_harness 内核 + web_foundation 通信层 |
| 数据层 | 业务库 MySQL（mall 原有）+ AI 库 PostgreSQL / pgvector（新增） | 全部 PostgreSQL / pgvector + Redis |
| Agent 的角色 | 业务系统里的一个功能 | 项目本身 |
| 证明的能力 | 懂后端业务；能把不确定的 LLM 安全、低成本地接进有事务、有权限的系统 | 懂 Agent 内部怎么运转；会做评测、用数据迭代 |
| 面试主讲点 | 工具层安全（越权、幂等、审批、审计）、混合检索 RAG、意图与模型路由、语义缓存、护栏、跨库数据同步 | loop、上下文压缩、skills、长期记忆、工具沙箱、评测与错误分析、消融实验 |
| 详细方案 | [backend/PROJECT.md](backend/PROJECT.md) | [agent/PROJECT.md](agent/PROJECT.md)、[agent/EVAL.md](agent/EVAL.md) |

架构图：[电商后端 + 售后 Agent](diagrams/backend-stack/backend-stack.html)、[退货两阶段提交时序](diagrams/return-flow/return-flow.html)、[故障排查 Agent + 评测闭环](diagrams/agent-eval/agent-eval.html)。工作方式（资源约束、ADR、GitHub、AI 编程边界、讲述稿）见 [PRACTICES.md](PRACTICES.md)。

### 两个项目的连接点：一套评测框架，评两个 Agent

项目 B 里写的评测框架（任务 YAML、Runner、grader、PostgreSQL 结果库、SQL 报告）在设计时就抽象出 `AgentAdapter` 接口：项目 B 用"进程内适配器"直接驱动内核，项目 A 用"HTTP 适配器"通过接口驱动 Java 服务。这样：

- 项目 A 的评测不用重写，W15 只需要写一个 HTTP 适配器、40 条用例和几个数据库终态 grader
- 面试时两个项目共享一条主线：**"我用评测驱动 Agent 开发，同一套框架评测了两个不同语言、不同场景的 Agent"**。这比两个互不相干的项目更有说服力

## 每天怎么分配

| 时期 | 项目 | 八股 | 算法 |
|---|---|---|---|
| W1–W16（10/05–01/24） | **4–4.5 h** | 1–1.5 h（Java 主线 + AI 主线，见 [interview/PREP.md](interview/PREP.md)） | 不排 |
| W17–W22（01/25–03/07） | 只修 bug、准备讲述 | 1 h（第二遍） | **2.5 h** 集中冲刺 |

- **项目时段同一时间只做一个项目**，例外是下面的"周六 Java 手感实验"
- 周日 30 分钟复盘：勾选本文件、在 `log/` 写周记。进度以验收为准，落后就砍范围，不顺延

### 周六 Java 手感实验（W1–W9，每次 2 小时）

问题：算法推迟后，W1–W9 期间完全不写 Java，只读八股。读懂和写得出来是两回事，到 W10 才动手会很吃力。

做法：W1 先把 `macrozheng/mall` fork 到自己的 GitHub 并克隆到 `E:\mall-ai`。W1–W9 每周六从项目时段里拿出 2 小时，做一个和当周八股配套的小实验，代码放在 `E:\mall-ai` 的 `labs/` 目录。**每个实验都是项目 A 的预制件**，W10 以后直接复用。

| 周 | 实验 | 当周八股 | 之后用在项目 A 哪里 |
|---|---|---|---|
| W1 | Java 17 语法练手：record、switch 表达式、text block、Stream | Java 基础 | 全部代码 |
| W2 | 用 `LinkedHashMap` 写一个 LRU 缓存，打断点观察 `HashMap` 扩容 | 集合 | 会话记忆的最近 N 轮 |
| W3 | （本周项目时段已在跑 mall，跳过） | — | — |
| W4 | 线程池 + `CompletableFuture` 并发调用 3 个模拟接口，带超时 | 并发 | Agent 并行调用多个只读工具 |
| W5 | Spring Boot 3 小服务：REST + MyBatis 连 MySQL，再加第二个数据源连 PostgreSQL | MySQL | `mall-ai` 的双数据源 |
| W6 | Redis：`SET NX EX` 幂等 + Redisson 分布式锁，用 JMeter 并发 50 次验证 | MySQL 事务 | 退货确认的幂等 |
| W7 | RabbitMQ：生产者确认 + 手动 ack + 死信队列 + 消费幂等 | PostgreSQL | 退款异步、商品同步 |
| W8 | Spring AI：一次工具调用 + 一次结构化输出（OpenAI 兼容接口） | Redis | Agent 骨架 |
| W9 | Spring AI + PgVectorStore：10 条政策入库并检索 | Redis | 政策 RAG |

有了这些预制件，W10 读代码和打地基的压力会小很多。

## 阶段闸门：到点检查，不达标就调整

| 闸门 | 时间 | 检查什么 | 不达标怎么办 |
|---|---|---|---|
| G1 | W3 周日 | `minimal_harness` 全绿；mall 能跑起来下单 | 内核没完成：W4 继续写，项目 B 砍掉历史案例记忆。mall 跑不起来（多半是内存）：按 PRACTICES.md 只起必需容器 |
| G2 | W6 周日 | 靶场能注入并还原至少 6 个故障；浏览器能看事件流 | 故障目录先定在 6 个，W7 照常开始评测，不等靶场完美 |
| G3 | W9 周日 | 项目 B 的"必须保住"清单全部完成 | 最多顺延 1 周（占用 W10），项目 A 相应砍掉语义导购和压测 |
| G4 | W13 周日 | 项目 A 的越权、幂等、草稿确认、混合检索 RAG 都已完成 | W14 的加分项全部取消，只做护栏和限流 |
| G5 | W16 周日 | 项目 A 冻结 | 无论完成到哪里都冻结，W17 必须开始刷算法 |

关于算法的风险：从零开始 6 周刷 Hot 100 很紧，而大厂暑期实习的笔试基本都有算法题。W17 起每天 2.5 小时不能再压缩；如果项目提前完成，就提前开始刷题。

## 时间线总览

| 阶段 | 周次 | 日期 | 项目时段 | 八股时段 |
|---|---|---|---|---|
| 0 热身 | W1–W3 | 10/05–10/25 | Agent 内核练习；跑起 mall 认名词；PostgreSQL + pgvector 入门 | Java 基础、集合；LLM 基础 |
| 1 项目 B | W4–W9 | 10/26–12/06 | 故障排查 Agent + 评测 + 长期记忆 | 并发、MySQL、PostgreSQL、Redis；Agent 与 RAG |
| 2 项目 A | W10–W16 | 12/07–01/24 | mall 售后 Agent 改造 | JVM、Spring、计网/OS、MQ/分布式、场景题 |
| 3 包装投递 | W17–W22 | 01/25–03/07 | 简历、博客、模拟面试、投递 | 第二遍；算法冲刺 |

2027 年春节是 2 月 6 日（周六），W19 按半休息周算。参考往年，阿里春招实习 3 月中旬开放投递，各厂大致 2 月底到 3 月陆续开放。**项目必须在 1 月 24 日冻结。**

---

## 阶段 0：热身（W1–W3，10/05–10/25）

### W1–W2：Agent 内核练习

- [ ] 按 `E:\pi_north\agent-harness-course` 第 2–8 课对照 Pi 源码读，只读和练习相关的部分
- [ ] `practice/minimal_harness` 测试全绿（session → provider → loop/tools → hooks → compaction → skills）
- [ ] 每完成一个模块，用 3 句话写下"为什么这么设计"，放进 `log/`
- [ ] 额外：给 `provider.py` 写一个真实的 OpenAI 兼容实现（DeepSeek / Qwen 都兼容这个接口），跑通一次真实对话

验收：`pytest -q` 全绿；能脱稿讲清 loop 的退出条件、工具报错怎么回传、compaction 什么时候触发。

### W3：后端名词实物化 + PostgreSQL 入门

- [ ] 读 [backend/TECH-MAP.md](backend/TECH-MAP.md)
- [ ] 在 W1 已 fork 的 `E:\mall-ai` 里，用 `document/docker/docker-compose-env.yml` 起中间件（按 PRACTICES.md 控制内存），本地跑起 `mall-admin`、`mall-portal`，用 Swagger 调通"登录 → 下单 → 申请退货"
- [ ] 完成 TECH-MAP 末尾"名词实物观察清单"的 MySQL / Redis / RabbitMQ / ES / Nginx 部分
- [ ] 用 `pgvector/pgvector` 镜像起一个 PostgreSQL，完成清单里的 PostgreSQL 部分：建扩展、插向量、建 HNSW 索引、做一次"向量 + 全文"混合查询

验收：能对着自己画的图说出一次下单请求经过了哪些组件；能用一条 SQL 做向量相似度检索并解释 `<=>` 是什么。

---

## 阶段 1：项目 B 故障排查 Agent（W4–W9，10/26–12/06）

详细任务见 [agent/PROJECT.md](agent/PROJECT.md) 的"逐周任务"，评测见 [agent/EVAL.md](agent/EVAL.md)。

| 周 | 里程碑 | 验收 |
|---|---|---|
| W4 | M1 能跑：仓库工程化、内核接真实模型、5 个只读工具、结构化诊断输出 | CLI 下自己定位 F02，会话写入 PostgreSQL |
| W5 | M2 能看能管：接 Server、审批 hook、OpenTelemetry 追踪 | 浏览器看完整事件流；拒绝审批后 Agent 换方案；Phoenix 里能看到每一步 |
| W6 | M3 有靶场：Compose 化 + toxiproxy + 10 个故障 + 5 份 runbook + 工具沙箱 | `inject.py F01` / `restore.py F01` 可用；工具只能执行白名单命令 |
| W7 | M4 有成绩：评测 v0（Runner 通过 `AgentAdapter` 接口驱动 Agent，为项目 A 复用做准备）+ 第一次错误分析 | 第一张成绩表 + 失败分类表 |
| W8 | M5 有提升：修前 2 类失败；故障扩到 15–20 个；LLM judge 校准；历史案例记忆 | 成绩新旧对比；记忆检索可用且没有数据泄漏 |
| W9 | M6 可展示：消融实验、大小模型路由、MCP server、CI 回归、README | README 首屏有架构图和成绩表；**项目 B 冻结** |

---

## 阶段 2：项目 A 电商售后 Agent（W10–W16，12/07–01/24）

详细任务见 [backend/PROJECT.md](backend/PROJECT.md) 的"逐周任务"。

| 周 | 里程碑 | 验收 |
|---|---|---|
| W10 | 读懂 mall 三条链路；把周六实验的预制件（双数据源、Spring AI、PgVectorStore）并入 `mall-ai` 模块；设计文档 | 设计文档覆盖每个工具的权限、读写、幂等方式，以及 PostgreSQL 表结构 |
| W11 | 售后 Agent v0：只读工具 + 身份注入、SSE 聊天页、PostgreSQL 会话记忆、意图路由 | 越权测试通过；意图分类准确率有初值 |
| W12 | 写操作：草稿确认、幂等、审计、客服预审、转人工 | 并发重放 10 次只生成 1 条申请 |
| W13 | 混合检索 RAG（pgvector + 中文全文 + RRF）、语义缓存、商品增量同步 | 政策问答带条款引用；缓存命中率和省下的成本有数字 |
| W14 | 护栏（注入检测、PII 脱敏、输出校验）、限流与 token 预算、退款异步；语义导购（加分） | 攻击用例被拦截；脱敏前后的请求日志对比 |
| W15 | 复用项目 B 的评测框架：写 HTTP 适配器 + 40 条用例 + 数据库终态 grader；错误分析迭代；可观测看板 | 成绩表；按意图拆分的延迟和成本 |
| W16 | 工程化与包装：一键部署、压测、单测、README、演示视频 | 别人按 README 能跑起来；**项目 A 冻结** |

---

## 阶段 3：包装与投递（W17–W22，01/25–03/07）

- [ ] W17：把 [resume/DRAFT.md](resume/DRAFT.md) 里的占位数字换成真实数据，出简历 v1；算法冲刺开始
- [ ] W17–W18：两篇复盘博客：一篇讲评测与错误分析，一篇讲 Agent 接入业务系统时的越权、幂等与跨库同步
- [ ] W17：每个项目写 30 秒 / 2 分钟 / 10 分钟三版讲述稿（见 PRACTICES.md 第七节）
- [ ] W18：用 ADR 准备每个项目 30 个深挖问题，录音自答；模拟面试两次
- [ ] W19：春节，算法保持每天 1 小时
- [ ] W20–W22：盯各厂开放时间投递；每场面试当天写复盘

---

## 落后时怎么砍

| 项目 | 必须保住 | 按顺序砍 |
|---|---|---|
| B 故障排查 | 自研内核、10 个故障、评测成绩表、一轮错误分析、审批 | ① CI 回归 ② MCP ③ 大小模型路由 ④ 历史案例记忆 ⑤ 超过 15 个的故障 |
| A 售后 Agent | 越权防护、幂等、草稿确认、审计、混合检索 RAG、20 条评测 | ① 语义导购 ② 压测 ③ 可观测看板 ④ 语义缓存 ⑤ 客服预审 |

每周日复盘时，如果本周验收没达成，下周先补验收，不开新任务。连续两周落后，就按上表砍范围，不顺延截止日期。

## 周进度打卡

| 周 | 项目验收 | 八股本周主题 | 备注 |
|---|---|---|---|
| W1 | | | |
| W2 | | | |
| W3 | | | |
| W4 | | | |
| W5 | | | |
| W6 | | | |
| W7 | | | |
| W8 | | | |
| W9 | | | |
| W10 | | | |
| W11 | | | |
| W12 | | | |
| W13 | | | |
| W14 | | | |
| W15 | | | |
| W16 | | | |
| W17 | | 算法累计： | |
| W18 | | 算法累计： | |
| W19 | | 算法累计： | |
| W20 | | 算法累计： | |
| W21 | | 算法累计： | |
| W22 | | 算法累计： | |
