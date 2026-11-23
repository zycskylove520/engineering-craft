# workflows 层导航（任务编排 · 操作序列）

具体任务类型明确时，按表取编排流程。每一步引用的规范在 `core/`，语言写法在 `languages/`。

| 任务类型 | 文件 | 什么时候用 |
|---|---|---|
| 修 bug | `bugfix.md` | 有明确故障/错误/错误行为 |
| 新功能 | `feature.md` | 从需求到交付的完整链路 |
| 多 agent 并行开发 | `multi-agent-feature.md` | 任务可拆成文件不重叠的独立单元 + 环境有委派工具（三道门全过），feature 的加速路径 |
| 重构 | `refactor.md` | 改善结构、不改变外部行为 |
| 问题排查 | `investigate.md` | 原因不明、需定位根因 |

编排层负责"先做什么后做什么"；具体"怎么做"看 core 对应文件。典型的跨层组合：
- 修 bug = workflows/bugfix + core/verification（验证与 unstick）+ core/testing-strategy（TDD）+ languages 对应语言
- 新功能 = workflows/feature + core/requirements + core/design-principles + core/delivery
- 多 agent 并行 = workflows/multi-agent-feature（三道门/三态路由）+ feature 的前四步（顺序过门）+ 各单元按 languages 规范落地 + 主 agent 集成验证
