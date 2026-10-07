# 研二 → 2027 春招实习：双项目作战手册

> 生成于 2026-10-06。目标岗位：AI Agent 开发 + Java 后端实习。

## 文档导航

| 文件 | 内容 |
|---|---|
| [01-JD调研.md](01-JD调研.md) | 两类岗位的硬性要求、加分项、面试高频题（含来源链接） |
| [02-开源项目清单.md](02-开源项目清单.md) | 经 GitHub API 核实的仓库清单（精读/二开/提PR 四组）+ llmwiki 与 DeepWiki 平替核实 |
| [03-项目构思.md](03-项目构思.md) | 项目 A/B 的候选方向、推荐组合、简历 bullet 示意、开源策略、面试叙事 |
| [04-行动路线图.md](04-行动路线图.md) | 20 周周程表 + 本周 7 件事 + 焦虑缓解 |

## 一图流结论

- **项目 A（纯 Agent）**：tiny-agent——上下文工程优先的编码 Agent 引擎（TS，对标 pi / openai-agents，带评测）
- **项目 B（Java × Agent）**：「智识」企业知识库智能问答平台（Spring Boot 3 + LangChain4j + PostgreSQL pgvector，带消融评测）
- **联动**：B 支持"双内核"（LangChain4j ↔ 自研 tiny-agent），两个项目一个故事
- **开源**：二开为主菜，副业给 a2a-java → langchain4j 提 PR

## 已产出的可运行资产

| 资产 | 位置 | 说明 |
|---|---|---|
| tiny-agent 种子 demo | [demo-mini-agent/agent.mjs](../demo-mini-agent/agent.mjs) | ~380 行零依赖 Node 脚本：agent loop + 工具层 + 截断 + auto-compact + reminder 注入。`node agent.mjs` 即可跑（mock 模式免 API key）；`node agent.mjs --real "任务"` 接 OpenAI 兼容 API |
| 架构图① | [.archify/architecture-tiny-agent-20261006/candidate-tiny-agent.html](../.archify/architecture-tiny-agent-20261006/candidate-tiny-agent.html) | tiny-agent 引擎架构（浏览器打开，含 PNG 导出） |
| 架构图② | [.archify/architecture-zhishi-20261006/candidate-zhishi.html](../.archify/architecture-zhishi-20261006/candidate-zhishi.html) | 智识平台架构（浏览器打开，含 PNG 导出） |

两张图均通过 archify 的 validate / deliver / check / browser-check 全部门禁（showcase 档）。
