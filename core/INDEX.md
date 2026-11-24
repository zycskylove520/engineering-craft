# core 层导航（通用工程规范 · 语言无关 · 约定式）

**读取协议**：本目录每个 `.md` 是一条通用规范，**文件名即主题**（`requirements.md`=需求、`verification.md`=验证、`gate-metrics.md`=门度量……）。按当前工程阶段选对应主题读，不必全读。**新增规范只新建文件，不改本导航。**

子目录 `memory/` 是工程记忆能力簇（文件名即能力），入口见 `project-memory.md`；它同样约定式，新增能力不改导航。

## 常见阶段 → 主题（举例，非清单）

需求含糊 → `requirements.md`；设计/决策 → `design-principles.md`；编码通用/注释 → `coding-standards.md`；测试/TDD → `testing-strategy.md`；验证/卡住 → `verification.md`；评审 → `review.md`；验收 → `acceptance.md`；联调 → `integration.md`；门度量 → `gate-metrics.md`；交付/PR → `delivery.md`；发布运维 → `release-and-operate.md`；长任务接力 → `operations.md`；复盘 → `retrospective.md`；工程记忆 → `project-memory.md`（其下 `memory/`）。

以上只是常见映射；遇到新主题直接按文件名判断，不在此登记。

**通用原则**：这些是语言无关的规范；语言特有的落地方式在 `languages/` 对应文件补充，冲突时以语言文件为准。
