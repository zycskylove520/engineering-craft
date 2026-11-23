# engineering-craft · 分层代码工程规范

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](./LICENSE)
[![Pure Markdown](https://img.shields.io/badge/skill-pure%20Markdown-blue.svg)](#安装)
[![Agent Agnostic](https://img.shields.io/badge/agent-agnostic-orange.svg)](#安装)
[![Trigger: 开始工程](https://img.shields.io/badge/trigger-%E5%BC%80%E5%A7%8B%E5%B7%A5%E7%A8%8B-purple.svg)](#怎么触发)

> 语言：[中文](./README.md) · [English](./README.en.md)

一个通用的 agent skill：让代码工程变成**有纪律、可验证、能积累**的过程。需求没确认不写码、设计没确认不开工、检查没过不送审；每一步都有真实命令和输出作证据，项目的规范与决策跨会话沉淀在 `.project_memory/` 里。

**不绑定任何 agent 软件**：全部是 Markdown，没有代码、构建或私有 API——任何能加载 skill 目录的 agent 都能用。

## 它解决什么

- 需求没澄清就写码，做到一半发现方向错了
- 改完只说"应该没问题"，拿不出真实命令和输出
- 项目规范散在聊天里，换个会话就丢
- 小任务太重、大任务太糙，一套流程走天下

## 安装

把整个目录放到 agent 加载 skill 的位置，重启会话（或触发技能重载）即可。

| Agent | 常见 skills 目录 |
|---|---|
| Claude Code | `~/.claude/skills/` 或项目 `.claude/skills/` |
| Cline / Roo Code | 项目的自定义指令目录（如 `.clinerules/`） |
| 其他支持 skill 的 agent | 该软件的 skills / 指令目录 |

```bash
git clone https://github.com/zycskylove520/engineering-craft.git
# 把 engineering-craft/ 整个目录复制到你的 agent 的 skills 目录
```

> 入口是带 YAML frontmatter 的 `SKILL.md`；若你的 agent 用别的入口约定，只需指向入口文件，内容无需改动。

## 怎么触发

说**「开始工程」**（精确四个字）；日常写代码、改 bug、调试、重构、审查代码时它也会按描述自动加载。

## 三种使用场景

### 1. 初次使用（空项目）

对 agent 说「开始工程」，告诉它你要做什么。它会先和你商谈**架构、语言、命名、技术栈**——不凭空替你决定——把约定落成项目里的规范文件，再进入需求澄清。你只需要回答它的问题、在关键处拍板。

### 2. 融入已有项目

- **已有代码、尚无记忆**：它会先通读源码，沉淀出这个项目的架构、命名和习惯（每条带文件路径作证据），主动指出不合理的设计并和你协商，然后建起 `.project_memory/`。不擅自大改。
- **代码和记忆都在**：它先读 `INDEX.md` 和各层规范，按项目既有约定干活，不重写你的规矩。
- **不想全量启用**：小改动可裁剪流程，但"需求没明确就不写码"这条不变。

### 3. 下一次使用

不用重新讲一遍项目背景。直接说「开始工程」+ 新需求，它会读已有记忆接着上次的约定走；每次交付后如果有长期约定变化，会写回规范层，下次自动生效。

## 运行原理（概览）

复杂任务交给多个角色 agent 协作——**产品 → 架构 → 研发 → 测试 → 评审 → 运维**六个岗位。它们按交接物依赖**严格保序**：上游交付落盘、下游才启动；同一岗位内部若有文件互不重叠的独立单元，才会**条件并行**。上游交接物出问题会冻结下游、退回重做，不让错误往下传。

只有三个地方需要**你**拍板：确认需求验收、确认技术选型、涉及上线时的 go/no-go；其余由 agent 按纪律推进。

它是一套**纪律与可验证性**，不是系统级强制hook：能让违规和未经验证的断言显眼可查，但不能物理阻止 agent 绕过流程。

运行原理与工程记忆的完整规范见 `SKILL.md`；六岗位的细节见 `workflows/multi-agent-feature.md`。

## 适用范围

- **适合**：有规模或多人/多 agent 协作、需要可审计过程、质量重于速度的代码工程。
- **不适合**：纯闲聊、查资料、非代码任务；一次性小改动按需裁剪即可。

## 目录结构

```
engineering-craft/
├── SKILL.md              # 薄入口：导航、红线、工程记忆
├── README.md / README.en.md
├── core/                 # 通用规范：需求、设计、编码、测试、验证、评审、交付…
├── workflows/            # 编排流程：feature / bugfix / refactor / investigate / multi-agent
├── languages/            # 语言规范：写码前读对应一篇
├── topics/               # 横切专题：安全 / 性能 / 并发 / API 设计 / 重构
└── CHANGELOG.md / LICENSE
```

## 许可

[MIT](./LICENSE)
