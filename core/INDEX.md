# core 层导航（通用工程规范 · 语言无关）

按工程阶段取对应文件，不必全读。

| 阶段/场景 | 文件 | 内容 |
|---|---|---|
| 需求含糊、要收窄验收 | `requirements.md` | 需求澄清四问、三句复述、验收标准 |
| 设计、架构决策、数据建模 | `design-principles.md` | SOLID/高内聚、先减后加、数据形状、草图先行 |
| 编码通用规矩、注释 | `coding-standards.md` | 命名/结构/错误处理/依赖/**注释分层** |
| 写测试、TDD、覆盖率 | `testing-strategy.md` | 测试金字塔、TDD、固件与 mock 边界、覆盖率 |
| 改完验证、贴证据、卡住 | `verification.md` | 基线→窄检→全套、读报错、爆炸半径、失败两次攻前提 |
| 评审、自审、看 findings | `review.md` | 范围/diff、findings 结构、4 轮封顶、多方案 |
| 交付前请用户/需求方确认"做完了" | `acceptance.md` | 对照 PRD 验收标准逐条核、谁验、偏差处理 |
| 多模块/服务拼起来验证 | `integration.md` | 契约对齐、两两联通+端到端、联调问题定位 |
| 看规范好不好用、门是否形同虚设 | `gate-metrics.md` | 6 字段最小 schema、数据留工作区、只回写长期教训 |
| 交付、PR、合并门 | `delivery.md` | PR 标题/描述/评论、合并门、接手 PR |
| 发布、部署、上线后、回滚、事件 | `release-and-operate.md` | 发布三件套、部署前门、上线观察、人主导的事件响应 |
| 长任务、上下文过半、接力 | `operations.md` | PROGRESS.md、ledger、重置信号、回来接力 |
| 会话末复盘、文案去 AI 味 | `retrospective.md` | 教训升级层级、去 AI 味清单 |
| 「开始工程」工程记忆目录 | `project-memory.md` | 目录结构、分层职责、只存规范不遗忘、写回时机 |

**通用原则**：这些是语言无关的规范；语言特有的落地方式（文档工具、测试框架、并发模型）在 `languages/` 对应文件里补充，冲突时以语言文件为准。
