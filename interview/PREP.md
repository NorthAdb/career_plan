# 八股与笔试准备

- **W1–W16**：每天 1–1.5 小时八股，分两条线：Java 后端主线（约 1 小时）+ AI 主线（约 30 分钟）。不刷算法，时间留给项目
- **W17–W22**：项目冻结后每天 2.5 小时算法冲刺 + 1 小时八股第二遍

## 一、八股

### 主资料

- [JavaGuide](https://github.com/Snailclimb/JavaGuide)（★159k）：Java 后端八股主线，也覆盖 AI 应用开发
- [小林 coding / CS-Base](https://github.com/xiaolincoder/CS-Base)（★18.5k）：计网、操作系统、MySQL、Redis 的图解，比 JavaGuide 好懂
- [advanced-java](https://github.com/doocs/advanced-java)（★79k）：高并发、分布式、微服务进阶
- AI 主线：两个项目文档里的技术点 + [RESOURCES.md](../RESOURCES.md) 的必读文章 + PostgreSQL / pgvector 官方文档

### 方法

- 每个知识点看完后**合上资料，用自己的话说一遍**（可以录音），说不出来就是没懂
- 每周日用 15 分钟抽查本周内容
- **和项目挂钩**：学到 Redis 分布式锁，就想项目 A 的幂等怎么做；学到 MQ 可靠投递，就想商品同步怎么保证不丢；学到 RAG 分块，就回项目里试一种新切法

### 周表

| 周 | Java 主线（约 1 h） | AI 主线（约 30 min） |
|---|---|---|
| W1 | Java 基础：面向对象、`equals`/`hashCode`、`String`、异常、泛型擦除、反射 | LLM 基础：Token、上下文窗口、温度与采样、为什么会幻觉 |
| W2 | 集合：`HashMap` 底层与扩容、`ConcurrentHashMap`、`ArrayList` | Transformer 与注意力机制（讲到能画图的程度） |
| W3 | 并发（上）：线程池七参数与拒绝策略、`synchronized` 锁升级、`volatile` | Embedding 与向量检索；pgvector 的 HNSW / IVFFlat |
| W4 | 并发（下）：AQS、`ReentrantLock`、CAS 与 ABA、`ThreadLocal`、`CompletableFuture` | Function Calling 原理；Agent loop 与死循环治理 |
| W5 | MySQL（上）：B+ 树、聚簇/非聚簇、回表、覆盖索引、最左前缀、索引失效、explain | 上下文工程：压缩、截断、按需加载（对照你的内核） |
| W6 | MySQL（下）：ACID、隔离级别、MVCC、行锁/间隙锁/临键锁、redo/undo/binlog | Agent 记忆：短期 / 长期 / 摘要，各自的实现方式 |
| W7 | **PostgreSQL**：和 MySQL 的区别、PG 的 MVCC 与 VACUUM、GIN/GiST 索引、JSONB | Agent 评测：task / trial / grader、pass@k 与 pass^k、LLM judge 校准 |
| W8 | Redis（上）：数据结构与底层编码、单线程为什么快、RDB/AOF、过期与淘汰 | RAG 全链路：分块、召回、混合检索与 RRF、重排、引用 |
| W9 | Redis（下）：穿透/击穿/雪崩、缓存一致性、Redisson 分布式锁、主从/哨兵/集群 | MCP 协议；工具安全与 prompt 注入防护 |
| W10 | JVM：内存区域、GC 与 G1、类加载与双亲委派、OOM 排查 | Spring AI 的核心抽象（ChatClient、Advisor、Tool、VectorStore） |
| W11 | Spring：IoC、AOP（动态代理）、Bean 生命周期、循环依赖、事务传播与失效、自动配置 | 意图识别与模型路由；结构化输出 |
| W12 | 计网 + OS：TCP 握手挥手、TIME_WAIT、HTTPS；进程/线程、虚拟内存、IO 多路复用 | 语义缓存；成本优化手段汇总 |
| W13 | MQ：消息丢失/重复/顺序/积压；本地消息表、CDC | 可观测：OpenTelemetry GenAI 语义约定、trace 里该记什么 |
| W14 | 分布式：CAP、分布式事务方案、分布式 ID、限流算法、熔断降级 | 护栏：PII 脱敏、输出校验、越权防护的通用模式 |
| W15 | 场景题：秒杀、短链、Feed 流、排行榜、延迟任务 | 多 Agent 的适用场景与代价（为了答"为什么不用"） |
| W16 | 场景题：每题按"需求 → 估算 → 设计 → 瓶颈"答 | 回顾两个项目所有 AI 技术点，每个准备 1 分钟讲解 |
| W17–W18 | 第二遍：自测说不出来的点回炉 | 第二遍 |

## 二、算法（W17 起）

### 原则

- **用 Java 写**，面 Java 后端岗必须用 Java
- 国内大厂笔试多是 **ACM 模式**：自己用 `Scanner` / `BufferedReader` 读输入。LeetCode 是核心代码模式，要专门练输入输出
- 20 分钟没思路就看题解，**看懂后关掉自己重写**，第二天再写一遍
- 错题记在 `log/leetcode-wrong.md`

### 节奏（每天 2.5 小时）

| 周 | 内容 | 目标 |
|---|---|---|
| W17 | [Hot 100](https://leetcode.cn/studyplan/top-100-liked/)：哈希、双指针、滑动窗口、子串、数组、矩阵 | 约 20 题 |
| W18 | 链表、二叉树 | 约 20 题 |
| W19（春节） | 每天 1 小时：图论、回溯 | 约 10 题 |
| W20 | 二分、栈、堆、贪心 | 约 20 题 |
| W21 | 动态规划、多维 DP | 约 15 题 |
| W22 起 | 错题二刷；牛客往年真题练 ACM 模式；每周一次限时模拟 | 能稳定 A 出 2 题 |

6 周大约 85 题，覆盖 Hot 100 的大部分。专题薄弱处对照 [代码随想录](https://github.com/youngyangyang04/leetcode-master)（★62.6k）补。

### 第一天就要熟的 Java API

`HashMap`、`HashSet`、`ArrayList`、`ArrayDeque`（当栈和队列）、`PriorityQueue`、`Arrays.sort` 与自定义比较器、`StringBuilder`、`Collections`。

## 三、项目深挖准备（W18）

两个项目各准备 30 个问题：

1. 各项目文档末尾的"面试会被追问的问题"
2. 简历上每一个技术名词都问自己"为什么用它、不用会怎样、换成别的行不行"
3. 每一个数字都问自己"怎么测的、样本多大、可信吗"

对着手机录音回答，回放时听哪里卡壳、哪里含糊。

## 四、投递与面试节奏（W20 起）

- 先投 2–3 家非首选公司练手，再投目标公司
- 同时投"Java 后端开发"和"AI Agent 应用开发"：面后端岗把项目 A 放第一，面 Agent 岗把项目 B 放第一
- 每场面试当天写复盘，追加到本文件末尾

## 面试复盘记录

（面试后在这里追加）
