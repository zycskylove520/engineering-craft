# 演进记录

> 只保留当前仍成立、且影响工程流程与项目记忆构建的关键设计决策。审计过程、措辞修正、bug 修复、已被作废的旧方案不再记录——它们是噪音，读了反而干扰对当前规则的理解。完整规则以 `SKILL.md` 与各能力文件为准。

## 2026-11-26（同日自审修订）— 消除落盘后的内部矛盾

> 首次对齐落盘后按 skill 自身纪律做了一次自审，发现并修复以下不自洽；本地已修正，需同步推送。

- **G3 证据因果倒置（真 bug）**：初版把"编译/窄检证据落 VERIFY-LOG"写在 feature.md step 10（G3 之后），而 G3（step 8）却要核这份证据——消费先于生产。改为 step 7 实现可编译后立刻落窄检证据，G3 才核；multi-agent 同步（委派时每个单元各自落证据、lead 核落盘）。
- **G3 定义三处不一致**：SKILL §2.1 表格原写"编译/窄检"两条件，与 P0 第 9/10/11 条和 feature.md 三合一定义冲突——表格改为三合一，并写明绿灯在 step 9 记、不阻塞 G3。
- **TDD 红灯与重构冲突（真 bug）**：初版"写任何生产代码前必须有红灯"会让行为保持型重构无法合规（重构是红→绿→重构循环里绿灯下做的那步）。改为：红→绿硬门只针对改变行为的代码（新功能/修 bug）；重构走"基线绿 + 每步保持绿 + 全套对比基线"等价路径；任何改动都仍须落 VERIFY-LOG，红灯可豁免、验证证据不可豁免。P0 第 10 条、§2.1 表格、testing-strategy §2 同步。
- **门确认措辞**：G3 是 agent 凭证据自检、不是人拍板；初版"门的确认按 G1→G2→G3"措辞把 G3 也当人确认。改为"门按 G1→G2→G3 推进、禁止倒序；G1/G2 由人拍板、agent 不得自报确认，G3 由 agent 凭证据自检"。
- **VERIFY-LOG 悬空引用**：初版新增台账但未登记进 tmp 产物清单。补进 `core/memory/document-tree.md` §7 与 `core/project-memory.md` 边界说明。
- **台账格式不统一**：SKILL P0 第 2 条写"命令 | 输出摘要 | 日期"，实际示例用 `- 日期 · 命令 :: 输出摘要`。统一为后者（与 verification.md / testing-strategy.md 示例一致）。
- **bugfix.md / refactor.md 未接纪律**：两条会写生产代码的工作流没同步 VERIFY-LOG。bugfix 补红灯+窄检证据落账；refactor 补基线绿+全套证据落账。
- **完成检查同步**：TDD 项标注重构豁免，证据项强调"过门前落盘、不是门后补"。

## 2026-11-26 — 对齐 dsh-engineering-craft 插件的融合纪律（通用部分）

> DSH 特化部分（工具硬门、面板、state.json、多 agent 默认、Config/locale）不属于落后，未引入。本次只把插件在真实踩坑中沉淀、且纯 Markdown 能表达的纪律补进 skill，使其升级为工具强制时规范侧不缺位。

- **TDD 硬前置**：P0 新增第 10 条——写任何生产代码前先写失败测试、看到它失败并记红灯，无红灯不得写实现；`core/testing-strategy.md` §2 从"可选方法"升为强制前置，附红/绿灯记账格式。
- **证据先于断言**：P0 第 11 条 + `core/verification.md` §2——宣称完成前必须真实跑命令并把"命令+输出摘要+日期"落进 `.project_memory/tmp/VERIFY-LOG.md`；G3 核验台账非空。"上次跑过/自述/信心"不算证据。
- **反合理化红线**：`SKILL.md` §4.1 新增 7 条（太简单不用走流程 / 先看代码再定起点 / 先写实现回头补测试 / 测试之前跑过 / 子 agent 说完成 / 计划口头说 / 有信心算验证过），出现即视为违规。
- **门序硬约束**：G2 须 G1 已过、G3 须 G2 已过，禁止倒序补确认；G3 定义扩为"编译/窄检 + TDD 红灯 + 验证台账"三合一（P0 第 9/10/11 条，`workflows/feature.md`、`workflows/multi-agent-feature.md` 同步）。
- **「开始工程」两段式路由**：强制前置询问在任何目录状态判断之前、不可绕过；根确认后三分支——有代码无记忆=直接建库不问、空目录/已有记忆=才问；防"源码注释提到流程字眼误判走过流程"（`workflows/init-project-memory.md`）。
- **建库完成度判据**：按内容达标（五件套条目≥12 且 tmp/PLAN.md 已落盘）判定，空骨架不算完成；量化门槛 RULE≥35% / 配方≥3 条含验证 / 质量分≥60（`init-project-memory.md`、`content-model.md`）。
- **检索策略细化**：字面整句→分词/二元组回退→相关性排序，可按 kind/layer 过滤，正文命中也返回整条 id/kind，空线索不捞噪声（`retrieval-and-writeback.md`）。
- **五件套格式硬判据**：证据行/记录行必须独立成行、以行首标记起头，挤成一行不算合格（`content-model.md`）。
- **安全与并行**：路径 containment 按实际落盘语义校验（POSIX 标签在 Windows 宿主越界实测）；并行 ownerFiles 归一化（折叠 ./、尾斜杠、大小写）后才判不重叠（`topics/security.md`、`multi-agent-feature.md`）。
- **新需求重置**：新需求不得复用上一需求"已完成"的门状态/交接物直接开工，重新走完整流程（`feature.md`、`multi-agent-feature.md`）。

## 2026-11-24 — 工程记忆系统定稿
- 记忆规格拆为三个正交能力（`core/memory/`）：`content-model.md`（一条记忆长什么样）、`document-tree.md`（记忆目录怎么划）、`retrieval-and-writeback.md`（写码前检索套用、新模式验证后回灌）；`core/project-memory.md` 是能力簇索引。
- 记忆目录标准结构：**6 个职责目录**（architecture / conventions / api-contracts / data-model / testing / build）+ **3 根文件**（INDEX / status / open-questions），全部建、内容可薄但不缺位；顶层按职责分目录，目录内按子系统分文件（数量动态）。`build/` 按项目实际构建方式分文件（cmake / cargo / pnpm / setuptools…），不并入 architecture。`status.md` 是项目状况快照，`open-questions.md` 是框架问题清单、随修复递减。
- 记忆内容规格：四种陈述类型（RULE / FACT / DECISION / OPEN）、五件套（稳定 id + kind + 陈述 + 证据 + 日期）、写规则不写清单、高频动作写成"抄哪个文件/改哪几处"的配方。
- 使用闭环：写业务/核心代码前必须检索工程记忆、按已有写法写；`feature.md` / `bugfix.md` / `refactor.md` 均执行。
- 执行产物统一放 `.project_memory/tmp/`（PLAN / PROGRESS / GATE-LOG / FINDINGS / notes 等）：是记忆目录内的临时区，不进规范层、INDEX 不登记、可清理，不散落在项目根。
- 记忆内容四条硬约束（真实项目实测后固化）：① 禁止本机绝对路径（skill/agent 安装路径）——INDEX 流程只写动作、证据用项目内相对路径，保证可迁移；② 事实等级（实测/推断/用户约定）与核实状态（已复核/待复核）正交分列，禁止"推断（子代理报告，待复核）"混写；③ 证据主体是文件路径+模式描述、行号仅括号内辅助（删掉行号仍能定位）；④ status 最近变更只记项目本身重要变更，不写一次性排查插曲。

## 2026-11-24 — 约定式导航（完全解耦）
- 四个 INDEX（core / workflows / languages / topics）只声明读取协议——文件名即适用信号；新增文件只新建、不改任何索引。`SKILL.md` 是薄入口，只常驻导航与红线。

## 2026-11-23 — 阶段门硬约束
- G1（需求未明确）、G2（设计未确认）、G3（检查未过）为硬门：未过不得进入下游、不得写实现代码、不得送审。以纪律形式写进 P0 铁律与各 workflow。

## 2026-11-23 — 多 agent 编排定稿
- 岗位之间按交接物依赖严格顺序（岗位流水线）；只在同岗位、文件不重叠的独立单元上条件并行（能力门/任务形状门/收益门全过）。单 agent 是一等路径。

## 2026-11-20 — 四层架构
- 从单文件重构为四层：`core`（通用规范）/ `workflows`（编排）/ `languages`（语言规范）/ `topics`（横切专题）；薄入口 + 按需读取。
