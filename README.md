# engineering-craft · 分层代码工程规范

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](./LICENSE)
[![Pure Markdown](https://img.shields.io/badge/skill-pure%20Markdown-blue.svg)](#为什么通用)
[![Agent Agnostic](https://img.shields.io/badge/agent-agnostic-orange.svg)](#为什么通用)
[![Trigger: 开始工程](https://img.shields.io/badge/trigger-%E5%BC%80%E5%A7%8B%E5%B7%A5%E7%A8%8B-purple.svg)](#触发方式)

> 语言：[中文](./README.md) · [English](./README.en.md)

一个通用的 agent skill：把代码工程做成**有纪律、可验证、能积累**的过程——从需求澄清到发布运维，每一阶段都有进入条件、交接物和准出标准，不带着未确认的上游产物往下走。

**不绑定任何 agent 软件。** 它是一组纯 Markdown 规范，不调用任何专有工具或私有 API——任何能加载 skill / 指令目录的 agent 都能使用。

## 它解决什么

- 需求还没澄清就开始写代码，做到一半发现方向错了
- 改完代码只说"应该没问题"，没有真实命令和输出作证据
- 断言分不清是实测、推断还是猜测，出了错追不回来
- 项目规范散在聊天里，换个会话就丢，新人接手无从下手
- 小任务和大任务用同一套流程，要么太重要么太糙

## 触发方式

说**「开始工程」**（精确四个字）进入工程记忆流程；日常写代码、改 bug、调试、重构、审查代码同样适用本 skill。

进入后按任务类型判断 → 读对应规范层 → 按流程编码 → 验证 → 交付 →（涉及上线则发布运维）→ 追问下一需求。

## 适用范围

**适合**：
- 有一定规模或需要多人 / 多 agent 协作的代码工程任务——新功能、重构、跨模块改动、需要长期维护的项目。
- 需要"可审计"的过程：交接物落盘、门有确认、断言有证据、规范能跨会话积累。
- 代码质量与过程纪律比"最快出活"更重要的场景（生产代码、开源项目、团队项目）。
- 已接入 subagent 的 agent 环境，想让产品 / 架构 / 研发 / 测试 / 评审 / 运维按交接物顺序协作。

**不适合 / 按需裁剪**：
- 一次性小改动（改个错别字、调个常量）：不必留全套交接物，按 ACDL 裁剪到最小——但需求未明确（G1）仍不许动手。
- 纯闲聊、查资料、写文档、非代码任务：本 skill 不触发。
- 强顺序的紧耦合任务（如 TDD 红→绿→重构）：走单 agent，不强行并行。
- **没有 hook 的系统**：本 skill 是纪律与可验证性，不是物理强制——它能让违规显眼可查，但不能阻止 agent 违反门。别把它当"系统级护栏"。

## 使用方法

1. **安装**：见下方「安装」，把目录放进 agent 的 skills 位置并重载。
2. **触发**：说「开始工程」（精确四字）；或在日常写代码 / 改 bug / 重构 / 审查时按 skill 描述自动加载。
3. **定任务类型**：需求澄清 → 新功能（`workflows/feature.md`）/ 修 bug（`bugfix.md`）/ 重构（`refactor.md`）/ 排查（`investigate.md`）。
4. **过门再动手**：先确认 G1（需求 / PRD 可判定验收）、G2（设计 / ADR + 契约确认）；没过就先补上游，不写实现代码。
5. **按需读层**：进入某阶段读 `core/` 对应文件；识别语言后**写码前**读 `languages/` 对应一篇；涉及安全 / 性能 / 并发 / API 设计叠加 `topics/`。不要一次性全读。
6. **多 agent 协作**（可选）：角色间按交接物顺序走形态 A 岗位流水线；只有研发岗内部、测试准备阶段在文件不重叠 + 三道门全过时启用形态 B 并行。上游交接物有缺陷就冻结下游退回该岗。详见 `workflows/multi-agent-feature.md`。
7. **编码与验证**：小步提交、公开 API 写注释；改完跑真实命令贴真实输出，结论标「实测 / 推断 / 猜测」；失败两次攻前提。
8. **联调与交付**：多模块拼起来做端到端验证，每类状态写入都要验"写后能读到"；然后评审自审 → 交付说明 →（涉及上线由人做 go/no-go）；规范有变写回 `.project_memory/`。
9. **收尾**：追问下一需求 + 是否需要调整修正，然后停下等回复。

> 全程记住入口 `SKILL.md` 是导航：具体规范按导航按需读取，不凭记忆跳过查阅。

## 它怎么工作

四层架构，**薄入口 + 按需查阅**——`SKILL.md` 只常驻导航与红线，具体规范按导航表读对应文件，不凭记忆跳过查阅。

| 层 | 目录 | 何时读 |
|---|---|---|
| L1 通用规范 | `core/` | 进入对应工程阶段（需求/设计/编码/测试/联调/验收/验证/评审/交付/发布运维/复盘）时 |
| L2 编排流程 | `workflows/` | 具体任务类型明确时（修 bug / 新功能 / 重构 / 排查） |
| L3 语言规范 | `languages/` | 识别到项目主语言后、写任何代码前必读对应一篇 |
| L4 横切专题 | `topics/` | 任务涉及安全/性能/并发/API 设计/重构时叠加 |

### 六大阶段（角色 × 阶段 × 交接物）

叠在四层架构之上的是一条**六阶段流水线**——产品 → 架构 → 研发 → 测试 → 评审 → 运维。每个阶段都有明确的**进入条件**、**结构化交接物**和**准出标准**：

| 阶段 | 角色 | 进入条件（硬门） | 交接物 | 准出标准 |
|---|---|---|---|---|
| ① 需求澄清 | 产品 | 拿到需求输入（含可判定验收的雏形或待澄清原料） | **PRD**（目标 / 用户故事 / 优先级 / 开放问题） | 人确认验收标准（G1）→ 可进设计 |
| ② 方案设计 | 架构 | G1 过：PRD 存在、验收标准已确认落盘 | **ADR**（是什么/为什么/取舍）+ 接口契约 + 双草图 | 人确认技术选型（G2）→ 可开工 |
| ③ 编码实现 | 研发 | G2 过：ADR + 契约已落盘、选型已确认 | 可编译的小步提交 + **PLAN.md** | 窄检/类型检查通过（G3）→ 可测试 |
| ④ 测试验证 | 测试 | G3 过：代码可编译、窄检通过、交接物齐 | 测试代码 + 覆盖率/边界异常报告 + 真实输出 | 全绿且证据分级齐全 → 可评审 |
| ⑤ 评审自查 | 评审 | 测试全绿、证据齐全；没过检查的改动不送审 | **Brief** + **findings**（id/严重度/文件:行/问题/修法） | VERDICT: APPROVE、4 轮封顶 → 可交付 |
| ⑥ 交付发布 | 运维 | 评审 APPROVE、PR/合并说明齐 | PR/合并说明 → 发布清单 + 回滚预案 + 观察项 | 人做 go/no-go → 可发布 |

**双向硬门纪律**：每个阶段同时卡进入条件和准出条件——进入条件不满足就停在本阶段，明确说出"缺什么、向谁要"，绝不带着未确认的上游交接物往下走；准出不满足不放行下游。架构选型、go/no-go、事件指挥归人，agent 只做生成/分析/辅助。小任务可按 ACDL 裁剪"哪些阶段留交接物"，但**裁剪的不是进入条件**——需求没明确（G1 未过）就不许动手写实现代码。

### 三道阶段门（硬约束）

- **G1**：需求（PRD）未确认到可判定验收 → 禁止写实现代码
- **G2**：设计（ADR + 接口契约）未确认 → 禁止开工
- **G3**：检查未过 → 禁止送审

### 多 agent 协同

多 agent 是组织方式，不只是加速。两种形态：**岗位流水线**（角色按交接物依赖严格顺序，可整条恒用）与**同岗位内的条件并行**。**岗位之间按交接物依赖严格顺序**（产品→架构因果、评审依赖最终 diff、发布依赖人拍板，均无并行前提）；**只有研发岗内部、测试准备阶段允许条件并行**——需三道门全过：① 能力门（环境有 subagent 且策略允许）；② 任务形状门（≥2 个归属不重叠的独立单元，各有可判定验收）；③ 收益门（任务够大能摊平协调成本）。上游交接物有缺陷 → 冻结下游退回该岗；契约因环境测不了 → 收敛到已验证替代并在 ADR 留痕。详见 `workflows/multi-agent-feature.md`。

### 工程记忆

`「开始工程」` 会在项目根建 `.project_memory/`（隐藏目录），沉淀该项目的规范、约定与决策——只存规范，不写会话流水；带标记 `.engineering-craft` 识别，名冲突时改用备选目录，绝不写入别人的目录。

### P0 铁律（常驻）

1. 写码前必读对应语言规范。
2. 改完必验证：跑真实命令、贴真实输出，未经验证不宣称完成。
3. 公开 API 必有注释。
4. 证据分级：实测 / 推断 / 猜测；不编造接口、库、文档、测试结果。
5. 一次一件事；失败两次攻前提。
6. 安全默认：输入不可信、最小权限、敏感信息不硬编码。
7. 破坏性操作先确认目标路径。
8. 能力诚实：hook 才能强制的规则本 skill 只能写成纪律；无工具时如实降级。
9. 阶段门硬约束：上游进入条件未满足，禁止进入下游。

## 为什么通用

- 全部内容是 Markdown 文本，没有代码、没有构建步骤、没有平台专有 API。
- 引用的方法论（MetaGPT、Agentic SDLC、ACDL 等）都是公开通用框架。
- 工程记忆存在项目目录的普通文件里（`.project_memory/`），不依赖任何平台的会话存储。
- 入口文件是 `SKILL.md`，内含 `name` / `description` 的 YAML frontmatter——多数 agent 的通用约定。

## 安装

这个 skill 就是一个目录，没有运行时依赖。把它放到你的 agent 软件加载 skill 的位置，重启会话（或触发技能重载），说「开始工程」即可。

常见 skills 目录（以各软件实际版本为准）：

| Agent | 常见位置 |
|---|---|
| Claude Code | `~/.claude/skills/` 或项目的 `.claude/skills/` |
| Cline / Roo Code | 项目根目录的自定义指令目录（如 `.clinerules/`） |
| 其他支持 skill 的 agent | 该软件的 skills / 指令目录 |

> 不同 agent 对入口的约定不同：有的读 `SKILL.md` + frontmatter，有的用别的文件名或格式。本 skill 入口是 `SKILL.md`；如你的 agent 用别的约定，只需按它要求指定入口文件，四层内容无需改动。

从仓库安装：

```bash
git clone https://github.com/zycskylove520/engineering-craft.git
# 然后把 engineering-craft/ 整个目录复制到你的 agent 的 skills 目录
```

## 目录结构

```
engineering-craft/
├── SKILL.md              # 薄入口：导航、红线、阶段门、工程记忆
├── CHANGELOG.md          # 演进记录
├── README.md             # 中文文档（默认）
├── README.en.md          # 英文文档
├── core/                 # L1 通用规范（按阶段读）
│   ├── INDEX.md
│   ├── requirements.md / design-principles.md / coding-standards.md
│   ├── testing-strategy.md / verification.md / review.md
│   ├── integration.md / acceptance.md / delivery.md
│   ├── release-and-operate.md / retrospective.md
│   ├── operations.md / project-memory.md / gate-metrics.md
├── workflows/            # L2 编排流程（按任务类型读）
│   ├── INDEX.md
│   ├── feature.md / bugfix.md / refactor.md / investigate.md
│   └── multi-agent-feature.md
├── languages/            # L3 语言规范（写码前必读对应一篇）
│   ├── INDEX.md / _TEMPLATE.md
│   ├── typescript.md / python.md / go.md / java.md / rust.md
│   └── cpp.md / cuda.md / bash.md / powershell.md
└── topics/               # L4 横切专题（涉及时叠加）
    ├── INDEX.md
    ├── security.md / performance.md / concurrency.md
    ├── api-design.md / refactoring.md
```

## 许可

[MIT](./LICENSE)
