# 演进记录

## 2026-11-23 — 推送前终检：路由描述的两处漏修
- 用户要求"再检查是否有内容冲突、逻辑是否自洽，没问题可推送"。逐处核对（而非只 grep 关键词）发现上一轮全量审计的"形态 A 路由入口"修正**只落在 `workflows/multi-agent-feature.md` §2，漏同步 SKILL.md 两处**：
  - SKILL.md §2.2 路由句仍写"强依赖→单 agent"，与本段上文"岗位流水线可整条恒用（多 agent）"直接对撞——改为四态路由（① 紧耦合不可分单元→单 agent；② 角色间交接物依赖→形态 A；③ 同岗位独立单元→形态 B；④ 用户明确要求→workflow），并明确"强顺序指紧耦合不可分单元，不等于角色流水线"。
  - SKILL.md 导航协议第 4 步仍写"满足三道门才走 multi-agent-feature"（三道门只属形态 B，会把形态 A 排除）——改为"需要多 agent 编排时走该 workflow，其中并行单元才需过三道门"。
- README.en.md 补 "sequential role pipeline" 措辞与中文"岗位流水线"术语对齐。终检批量核查：旧三态路由/强依赖误导/三道门前置条件三类残留全清；grep 的两处"命中"核实为正则过宽误报（限定语后并非前置条件），无掩盖。

## 2026-11-23 — 全量审计（4 维并行）与修订后自洽性复核
- 用户要求"认真检查一遍：逻辑自洽、架构严谨、条理清晰、未落实项与实战经验是否真融入"。用 4 个独立只读 agent 并行审计（导航完整性 / 门与逻辑一致性 / 经验融入与待落实 / 术语与完成定义），lead 逐条核实后修复。
- 审计结果：7 条修正建议 checklist 全 true、每条新规带实测证据指针、无孤儿文件、无活跃 `../` 残留；但发现 8 项一致性问题，已全修：
  - 高：README.en.md 多 agent 段仍是旧"可选加速/非默认"定位且缺冻结回退与契约收敛——按中文版与 SKILL §2.2 重写，补 workflow 指针、统一 ①②③。
  - 中：multi-agent-feature.md ①L22 交叉引用错位（§2→§1.2）；② §3 缺 G3 命名硬门、用户验收、发布（按需）、追问（与另三份 workflow 及 CHANGELOG 历史声明不一致），补齐并扩到 14 步；③ 形态 A 无路由入口且与"强顺序→单 agent"对撞——§2 重写为"先定形态再定并行"四态，明确"强顺序指紧耦合不可分单元，不等于角色流水线"；④ 事实等级两套体系（结论级 vs 契约级）无作用域说明——api-design §6 加区别段与映射、verification §2 加交叉指针、multi-agent §4 改为只指针不复述（消除 L93 自相矛盾）。
  - 低：FINDINGS 悬空未定义——review §4.5 明确落 `FINDINGS.md` 与补记列；SKILL 完成检查漏 review 收口项，补；description 角色列表漏"评审"，补；3 处裸路径前缀（`verification.md`/`coding-standards.md`/`cpp.md`）补目录前缀；CHANGELOG 历史旧说法加就地"历史状态"标记、补回一条误并入的标题、补记 README 中英同步。
- 边界纪律：历史记录按不遗忘铁律保留原文，只加就地标记不改原句；审计 agent 只读，修复由 lead 逐条核实后再动手，不盲信自述（实测核了 README.en.md、multi-agent §3、design-principles 路径、retrospective 路径等原文）。

## 2026-11-23 — 多 agent 形态实测修订（顺序流水线 + 条件并行）

- 触发：用本 skill 完整做了一个 DSH 插件工程（六岗位 subagent 顺序流水线实跑），回头复盘 skill 的 multi-agent 编排。原 `multi-agent-feature.md` 定位为"可选并行加速、三道门、单 agent 一等路径、尚未实战"——与真实高价值形态不符。
- 核心判断：**岗位之间按交接物依赖严格顺序，只有研发岗内部、测试准备阶段允许条件并行**。产品→架构是因果、评审依赖最终 diff、发布依赖人拍板，均无并行前提。并行只在"同一交接物版本 + 文件不重叠 + 三道门全过"内。
- 修订：① `multi-agent-feature.md` 重写为双形态（A 顺序岗位流水线 / B 同阶段条件并行）+ 岗位并行分类表 + 三种回退（失败单元 / **上游交接物缺陷→冻结下游退回上游** / **契约不可测→收敛到已验证替代并在 ADR 留痕**）+ 事实等级标注 + 完成定义；去掉"未实战"免责声明。② `review.md` 新增 §4.5/§4.6——APPROVE 附带 should/nit 的收口责任与 lead 收尾硬约束。③ `integration.md` 新增 §2.1"写后能读到"端到端硬检查 + 契约收敛交叉指针。④ SKILL.md §2.2、description、完成检查、workflows/INDEX、README.md 与 README.en.md 的多 agent 段同步措辞。
- 证据来源（实测，非推断）：CONTRACT 4 处 API 假设被推翻并退回架构岗（回退上游）；cordis 带参查询被 harness 层拒后收敛到 state.json+webServer/fetch（契约收敛）；5 个缺陷（指针未登记/裸路径串/两处确认卡恒假/handoffs 键错位）全为单测绿、端到端才暴露（写后能读到）；评审 R-01 should 由 lead 收尾修复并补记 FINDINGS（收口责任）。
- 边界诚实：这些是**通用工程模式**，不把 DSH 特例（具体 API/单 host row 等）写进通用 skill——那属于项目自己的 CONTRACT/工程记忆。形态 A 恒用时仍允许岗位内部按三道门启用形态 B；单 agent 在特定岗位内仍是一等路径。
- 自查补漏（同日）：用户追问"7 条建议是否逐条落实"，核对发现第 7 条（事实等级标注）**最初只写进 `multi-agent-feature.md` §4，落错属地**——它是 API 设计的横切规范，不该只活在编排文件里。已补到 `topics/api-design.md` §6（主规范）与 `core/design-principles.md` §5.1（决策留痕交叉指针）。其余 6 条逐条落盘核对无误。

## 2026-11-23 — 最终闭环检查修复（历史条目，标题此前误并入上条，现补回）
- 用户要求"审计→发现即修→再审，直到无问题"。全量审计脚本报一处 FAIL，核实为脚本自身解析 bug（`../core/review.md` 实际存在）；顺线发现真实不一致：languages 层 9 处用 `../core/`（相对当前文件）与 SKILL.md 第 37 行"所有路径相对于 skill 目录"的声明语义冲突。
- 修复：9 处 `../core/` 统一为 `core/`（_TEMPLATE/bash/cpp/cuda/go/java/powershell/rust，共 8 文件 + 模板）；bash.md 顺手修一处双空格排版。
- 证据：全量审计 A–H 八项重跑，引用完整性、孤儿、步骤连续、陈旧措辞、frontmatter、标题、数据污染全部通过。


## 2026-11-23 — 克制版补验收/联调/度量反馈环
- 用户担心补方案 2/3 会让 skill 膨胀并耦合项目记忆——确认担忧成立但可规避：关键是「规则留 skill、数据留工作区」与「常驻 vs 按需」。实测常驻仅 SKILL.md 111 行，其余 1353 行按需读取。
- 新增三个**按需** core 文件：`acceptance.md`（交付前用户验收，与 requirements.md 的开工前验收标准互补）、`integration.md`（跨模块联调，与 testing-strategy 的集成测试分层：代码层/改动层/系统层）、`gate-metrics.md`（门拦截度量：6 字段最小 schema，数据留工作区 GATE-LOG.md，**不进 skill/工程记忆**，只有长期教训回写规范层）。
- 常驻增量压到最小：core/INDEX 加 3 行导航；SKILL 主流程骨架加"（多模块时联调）（交付前用户验收）"两词、阶段清单加两项、完成检查加一条；feature.md 加两个"按需"步骤（步骤号顺延至 16）。
- 方案 3 用复用钩子实现：`project-memory.md` §5 加一行——"门反复拦截同一类缺失/纪律屡次被绕过→视为规范缺陷信号，写回规范层标日期与证据指针"，接上规范自我进化环，零新增体系。
- 边界诚实：度量是"让规范自我修正"，不是 KPI 考核；GATE-LOG 是临时产物，遵循不存流水铁律。证据：改动逐处回读、行数与引用实测。


## 2026-11-23 — 阶段门硬约束（顺序问题专项修正）
- 用户指出真实例子：产品没给需求时研发不能提前写代码——审计确认旧版只有软纪律（§2.1"达标才进下一环节"），P0 无硬门，requirements.md 仅"写不出第三句就只问一个问题"雏形；**上一轮"逻辑自洽/可靠"审计漏掉了约束强度维度**（已在回复中承认）。
- 档 1：SKILL §2.1 表格扩为「进入条件（硬门）/交接物/准出/谁拍板」双向门——产品行明确"未确认验收前禁止写实现代码"；研发行进入条件 G2 未过不得写代码；纪律段强调"裁缩短板不跳过未满足的进入条件"。
- 档 2：P0 新增第 9 条「阶段门硬约束」——G1 需求未澄清禁止写代码 / G2 设计未确认禁止开工 / G3 检查未过禁止送审，宁停不带病推进。
- 档 3：feature.md 插入 G1 需求门（第 2 步）/G2 设计门（第 5 步）/G3 开工送审门（第 8 步）三处显式检查，失败即停；multi-agent-feature.md 前置流水线补 G1/G2（未过禁止并行委派与子 agent 写代码）；bugfix/refactor 第 1 步补 G1 入口条件。完成检查加阶段门项。
- 诚实边界：这些仍是**纪律强化非机制强制**——无 hook 无法物理阻止违反（P0 第 8 条），能做到的是把纪律写到最强、违反显眼可查。证据：改动逐处回读、引用路径实测。


## 2026-11-23 — 全量一致性审计与修正
- 审计范围：SKILL.md 全文 + 4 个 INDEX + 全部 5 个 workflows + core 关键文件 + CHANGELOG（实测，逐文件读取）；另核对记忆库 README。
- 修 1 个真冲突：交付→发布→追问顺序矛盾——旧 SKILL §6 与 feature/bugfix 写"交付后立刻追问并停下"，会把方案一加的发布环节堵死。改为"走完本次适用的完整流程（交付；涉及上线先发布）后才追问"，并给 feature/bugfix/multi-agent-feature/refactor 四个 workflow 统一补上发布（按需）与追问步骤。
- 修 2 个不同步：① SKILL L1 阶段清单补"发布运维"；② 执行计划存放指代统一——PLAN.md 明确放工作区（临时产物），不进工程记忆；open-questions.md 只放未决问题。project-memory.md 加边界条、operations.md 的 PROGRESS.md 落点同步。
- 修 1 处 skill 外部陈旧引用：记忆库 README 第 10 行的 `layered-codebase-memory`/`.dsh/memory/` 更新为 `engineering-craft`/`.project_memory/` 并标改名日期；temporary/ 日记里的旧名是历史记录，按不遗忘铁律保留。
- 设计复核结论：四层正交 + 正交视图叠加的架构自洽；证据分级/能力诚实/单点定义/失败停止条件均达标；保留的观察项（评审行横跨两环节、收益门定性、多 agent 未实战）均为已知且已诚实标注，非缺陷。（历史状态："多 agent 未实战"已由本文件顶部 2026-11-23 形态实测修订条作废）证据：改动逐处回读、引用路径实测存在。


## 2026-11-23 — 多 agent 条件启用编排（方案二）
- 用户拍板方案二的设计（三道门 + 三态路由）：能力感知、优雅降级，但不写死"有工具就并行"。
- SKILL.md 新增 §2.2「何时启用多 agent」：三道门（能力门/任务形状门/收益门）全过才并行；明确**角色是流水线（顺序过门）、并行只在同阶段内的独立任务单元**——不把五角色拆成五个 agent 同时跑；路由三态（单 agent / subagent 并行 / workflow 脚本，workflow 仅用户明确要求或大规模编排时启用）。
- 新增 `workflows/multi-agent-feature.md`：作为 `feature.md` 的可选加速路径（历史表述，已由本文件顶部形态实测修订条改为"岗位流水线 + 条件并行"双形态），含三道门判据、三态路由、9 步执行流程（前置顺序过门→按文件归属拆单元→一条消息并行委派→主 agent 集成准备→收结果→主 agent 重跑全套→评审→按单元回退→交付）、交接物契约（复用 §2.1，落盘不口述）、6 条失败模式清单。
- 更新导航协议第 4 步、`workflows/INDEX.md`、description、完成检查同步。
- 关键取舍：单 agent 是一等路径不道歉；子 agent 自述"我测过了"不算数，主 agent 集成后重跑；失败按单元回退不整链崩。证据：工具集与 workflow 使用规则实测；三道门/三态路由为推断（基于已读的 MetaGPT/Agentic SDLC/ACDL/OpenHands 横评）；**尚未在真实多 agent 任务上验证**。（历史状态，已由本文件顶部 2026-11-23 形态实测修订条作废：已由真实工程任务完整实跑验证）


## 2026-11-23 — 补五环节视图与运维缺口（方案一）
- 调研市面多 agent 协同开发（实测，已读全文 4 份 + 摘要 3 份）：[Agentic SDLC Handbook ch4](https://danielmeppiel.github.io/agentic-sdlc-handbook/handbook/ch04-the-reference-architecture.html)（Human 五角色 Product/Architect/Dev/QA/SRE-Ops、8 阶段归 Intent/Build/Operate、三层参与）、[MetaGPT/IBM](https://www.ibm.com/think/topics/metagpt)（五角色 + SOP 结构化交接物）、[OpenHands 工作流横评](https://github.com/ketan1741/OpenHands-MultiAgent-Workflows)（五列含 Deployment）、[ACDL 蜂巢体系](https://www.bluexiii.com/acdl/)（REQ/DESIGN/BUILD/VERIFY 按范围裁剪）。
- SKILL.md 新增 §2.1「角色×阶段×交接物视图」：叠加在四层之上的正交视图，把产品/架构/研发/测试/运维五环节映射到已有文件，每环节定义交接物/准出/谁拍板；明确架构选型、go/no-go、事件指挥归人（agent 只生成/分析/辅助，能力诚实）。
- 新增 `core/release-and-operate.md`：补运维缺口——发布清单/回滚预案/观察项三件套、部署前门、上线观察、人主导事件响应；按 Agentic SDLC 成熟度标注，发布/运维为 Emerging/Directional，不幻想自动化运维。
- 主流程骨架接上发布/上线后观察（可裁剪，小改动止于交付说明）；完成检查加一条发布运维项；`core/INDEX.md` 加入新文件导航。
- 未改四层骨架（core/workflows/languages/topics），未做多 agent 编排（那是方案二）。证据：改动前后均回读文件与交叉引用，无猜测接口。


## 2026-11-20 — 更名 layered-codebase-memory → engineering-craft
- 目录名、`name:` 字段、工程记忆标记文件（`.engineering-craft`）、AGENT.md 引用全部同步更新。
- skill 定位不变：分层代码工程规范与工程记忆（代码任务专用，「开始工程」口令触发）。


## 2026-11-20 — 官方规范审计与修订
- 用 9 个子代理逐门查官方一手资料（PEP8/Google Style Guide 系列/Effective Go/Oracle JEP/Rust API Guidelines/Bash Manual/Microsoft Learn/NVIDIA Guide）核查准确性。
- 修订 9 处确定错误（均附官方出处）：C++ 普通函数改 mixed case（非 snake）、强制 #define guard（禁 #pragma once）；Rust edition 2024 门槛改 1.85+；Bash shebang 改 #!/bin/bash、文件名改下划线、set -e 在 &&/|| 链规则精确化；PowerShell 删除 C# 的 await/|=、清理 CXX/[nullable]；CUDA 区分 nsys/ncu 与已弃用的 nvprof。
- 更新过时项：C++20 基线、Rust edition 门槛、Bash 5.3、CUDA C++20/23 + Compute Sanitizer、TS 版本基线标注、Java 虚拟线程 pinning 按 JDK 24 (JEP 491) 分版本。
- 存疑项已标注来源属性（19 处）：Python（CapWords 术语/docstring 风格非 PEP 257/Result 非官方）、Go（MixedCaps/I 前缀/go doc 查看 vs 生成/fake 非逐字）、C++（absl::Status 属 Abseil/DCHECK 属 glog/Doxygen 非官方）、Rust（C-CONV 的 as_/to_/into_/rustdoc 不强制参数返回/CancellationToken 属 tokio-util）、CUDA（命名与 .cuh 属社区约定/lambda 需 --extended-lambda）、TypeScript（命名无官方标准）。均只加"社区/项目约定，非官方强制"说明，不删内容。

## 2026-11-20 — 补齐 CUDA C++ 规范
- 新增 cuda.md（GPU 并行编程，12 章）：线程层次/同步、内存层级与合并访问、cudaError 查错、CPU 参考比对测试、nsys/ncu/compute-sanitizer、占用率与 bank conflict。
- 正名：CUDA 是 C++ 扩展（nvcc 编译 C++，kernel 支持模板/类/lambda）；补充 device 端 C++ 特性（模板 kernel、`__host__ __device__` 双版本函数、Thrust/CUB/libcudacxx、异常/RTTI/虚函数限制）。
- 约定：整体遵循 cpp.md 的 Google 风格，cuda.md 写 device 端与 CUDA 特有内容。
- languages/ 现覆盖九种：Python、TypeScript、Go、Java、C++、Rust、Bash、PowerShell、CUDA C++。

## 2026-11-20 — 补齐 Shell 规范
- 新增 Bash（Linux/POSIX）与 PowerShell（Windows）两门 Shell 规范，12 章结构。
- Bash：set -euo pipefail、引号与参数展开、trap 清理、shellcheck/bats、破坏性操作安全。
- PowerShell：动词-名词命名、comment-based help、$ErrorActionPreference、Pester/PSScriptAnalyzer、退出码语义。
- languages/ 现覆盖八种：Python、TypeScript、Go、Java、C++、Rust、Bash、PowerShell。

## 2026-11-20 — 补齐语言规范
- 新增 Go / Java / C++（Google C++ Style Guide）/ Rust 四门语言规范，均为 12 章标准结构。
- 更新 languages/INDEX.md 的识别信号与状态（现六门语言：Python、TypeScript、Go、Java、C++、Rust 全部已填）。

## 2026-11-20 — 四层架构重构
- 从单文件 + 单一子文件，重构为四层：`core/`（通用规范）/`workflows/`（编排）/`languages/`（语言规范）/`topics/`（横切专题）。
- SKILL.md 瘦身为薄入口（导航 + P0 铁律 + 主流程），正文按导航按需读取。
- 原「执行纪律工具箱」按主题拆入 core 各文件（verification/operations/review/retrospective 等）。
- 注释规范改为分层：结构声明必须有良好注释、函数体逻辑只写为什么。
- 首批语言规范：Python、TypeScript；其余留 `_TEMPLATE.md` 占位。
- 设计原则：常驻薄、按需详；分级定位可靠；INDEX 精度优先。

## 之前 — 单文件阶段
- 初版：触发流程 + 工程记忆主线 + 代码规范。
- 融入执行纪律（需求收窄/验证/unstick/评审/长任务/交付/复盘等主题）。
- 第八节曾抽为 `references/execution-discipline.md` 按需子文件（本次重构已按主题拆分归位）。
