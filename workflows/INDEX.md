# workflows 层导航（任务编排 · 操作序列 · 约定式）

**读取协议**：本目录每个 `.md` 是一个工作流，**文件名即任务类型**（`init-project-memory.md`=工程记忆入口、`feature.md`=新功能、`bugfix.md`=修 bug、`refactor.md`=重构、`investigate.md`=排查、`multi-agent-feature.md`=多 agent 协同）。遇到任务先列出目录，选名字最匹配的那篇读。**新增工作流只新建文件，不改本导航。**

编排层负责"先做什么后做什么"；具体"怎么做"看 `core/` 对应主题，语言写法看 `languages/`。跨层组合知识写在各工作流自己文件里，不在此登记。

常见组合（举例，非清单）：
- 首次建库 = `init-project-memory.md`（操作序列）+ `core/memory/`（能力簇：内容模型/目录组织/使用闭环）
- 修 bug = `bugfix.md` + `core/verification.md` + `core/testing-strategy.md`
- 新功能 = `feature.md` + `core/requirements.md` + `core/design-principles.md` + `core/delivery.md`
- 多 agent = `multi-agent-feature.md` + 岗位流水线 + 条件并行

以上只是举例；新工作流按其文件名判断用法，不在此登记。
