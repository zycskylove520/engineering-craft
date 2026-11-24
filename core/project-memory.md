# 工程记忆能力簇 · 索引（约定式）

> 本文件不再是规格正文，而是 `core/memory/` 能力簇的**入口索引**。
> 约定式导航：本簇每个 `.md` 是一种能力，文件名即适用信号；新增能力只新建文件，**不改本索引**。

## 盘点原则（簇内共用 · 不随能力演化而改）

- **只存规范，不遗忘**：存"这个工程应该长什么样、为什么这样设计、大家怎么约定"；不记录会话流水。
- **每条带证据 + 日期**：文件路径或用户明确约定，"我觉得"不算。
- **旧决策保留**：被取代的标日期与原因、保留原文，不静默删除；不设 archive、不做过期清理。

## 能力清单（按文件名自描述，按当前需要读）

- `core/memory/content-model.md` —— 一条记忆长什么样（四种 kind、五件套、写规则不写清单、边界声明）。**写记忆前必读**。
- `core/memory/document-tree.md` —— 记忆目录怎么划（6 标准职责目录 + 3 根文件、目录内按子系统分文件、必挖清单、status.md 状况快照、open-questions.md 问题清单、build/ 按构建方式分文件）。**建库规划时必读**。
- `core/memory/retrieval-and-writeback.md` —— 写码前检索套用、新模式验证后回灌。**写业务/核心代码前必读**。

建库的操作序列在 `workflows/init-project-memory.md`（不属本簇，属编排层）。

## 读取协议

按当前任务选能力文件：写记忆 → content-model；规划文档树/通读 → document-tree；写码 → retrieval-and-writeback。多件事叠加就读多篇。不必全读。

## 写回时机（跨能力共用规则）

- 首次建库 → 写全部规范（见 `workflows/init-project-memory.md`）。
- 架构/规范新增或变更（用户确认）→ 写回对应文档，标日期带证据。
- 发现不合理设计 → 记入 `open-questions.md`（属记忆目录，随修复递减），协商修复后把形成的约定移入对应职责目录，问题标已解决（按不遗忘原则保留痕迹，不静默删）。
- 门反复拦截同类缺失（见 `core/gate-metrics.md` GATE-LOG 回看）→ 视为规范缺陷信号，写回结论标日期与证据指针；原始流水不写。
- 日常需求不改变规范 → 不写。

> 边界：PLAN.md / PROGRESS.md / GATE-LOG.md / FINDINGS.md 等执行产物统一放 `.project_memory/tmp/`（临时区，不进规范层、INDEX 不登记、可清理——见 `core/memory/document-tree.md` §7），**不散落在项目根**；`open-questions.md` 与 `status.md` 属工程记忆规范层（前者框架问题清单随修复递减，后者项目状况快照），都不放执行任务流水。临时产物完成后只把形成的长期约定写回职责目录。
