# engineering-craft · Layered Software Engineering Discipline

[![MIT License](https://img.shields.io/badge/license-MIT-green.svg)](./LICENSE)
[![Pure Markdown](https://img.shields.io/badge/skill-pure%20Markdown-blue.svg)](#installation)
[![Agent Agnostic](https://img.shields.io/badge/agent-agnostic-orange.svg)](#installation)
[![Trigger: 开始工程](https://img.shields.io/badge/trigger-%E5%BC%80%E5%A7%8B%E5%B7%A5%E7%A8%8B-purple.svg)](#how-to-trigger)

> Language: [中文](./README.md) · [English](./README.en.md)

A general-purpose agent skill that makes code engineering **disciplined, verifiable, and cumulative**. No implementation code before requirements are confirmed; no work before the design is confirmed; no review before checks pass. Every step is backed by real commands and output, and the project's rules and decisions accumulate in `.project_memory/` across sessions.

**Not tied to any agent software**: pure Markdown, no code, no build, no private API — any agent that can load a skill directory can use it.

## What it solves

- Writing code before requirements are clear, then finding the direction was wrong halfway through
- Saying "should be fine" with no real commands or output to back it up
- Project conventions scattered across chats, lost on the next session
- One heavyweight process for tiny tasks, one too loose for large ones

## Installation

Put the whole directory where your agent loads skills from, then restart the session (or reload skills).

| Agent | Common skills directory |
|---|---|
| Claude Code | `~/.claude/skills/` or the project's `.claude/skills/` |
| Cline / Roo Code | the project's custom instruction directory (e.g. `.clinerules/`) |
| Other skill-capable agents | that software's skills / instructions directory |

```bash
git clone https://github.com/zycskylove520/engineering-craft.git
# copy the whole engineering-craft/ directory into your agent's skills directory
```

> The entry is `SKILL.md` with YAML frontmatter; if your agent uses another entry convention, just point it at the entry file — no content changes needed.

## How to trigger

Say **"开始工程"** (the exact four Chinese characters); it also auto-loads during day-to-day coding, bug fixing, debugging, refactoring, and code review per its description.

## Three usage scenarios

### 1. First use (empty project)

Tell the agent "开始工程" and what you want to build. It first **discusses architecture, language, naming, and stack with you** — never decides for you — persists the conventions as project rules, then enters requirements clarification. You just answer its questions and make the key calls.

### 2. Joining an existing project

- **Code exists, no memory yet**: it reads the source, distills the project's architecture, naming, and habits (each with file-path evidence), flags questionable designs and negotiates with you, then builds `.project_memory/`. No unasked rewrites.
- **Code and memory both present**: it reads `INDEX.md` and the layers first, works within existing conventions, and doesn't rewrite your rules.
- **Don't want the full process**: small changes can trim the flow, but "no code before requirements are clear" never changes.

### 3. Next time

No need to re-explain the project. Say "开始工程" plus the new requirement — it reads the existing memory and picks up where the conventions left off. When long-term conventions change during delivery, they're written back to the rules layer and apply automatically next time.

## How it works (overview)

Complex tasks advance through six roles, each with a clear job:

| Role | What it does for you | What you get when it's done | Who decides |
|---|---|---|---|
| Product | Turns your vague idea into clear requirements with acceptance criteria | A requirements doc (goals / user stories / priorities) | **You**: confirm acceptance |
| Architect | Designs the initial structure from the confirmed requirements, picks language and stack | Architecture decisions + interface contract | **You**: confirm tech choices |
| Developer | Writes code in small, compilable steps against a plan | Working code + execution plan | Agent self-check |
| Tester | Writes tests, probes edge cases, pastes real command output | Test report + verifiable run evidence | Agent self-check |
| Reviewer | Reviews the diff and evidence, finds issues and fixes | Review verdict + issue list | Agent self-audit; you only if disputed |
| Ops | Prepares release material and a rollback plan | Release checklist + rollback plan + watch items | **You**: go/no-go |

The six roles run in **strict artifact-dependency order**: a downstream role starts only after the upstream handoff is on disk; conditional parallelism happens only inside one role when its units own non-overlapping files. A defect in an upstream artifact freezes downstream and returns that artifact for revision, so errors don't propagate.

Only **three decisions are yours** (marked in the table): confirm acceptance criteria, confirm tech choices, and go/no-go when deployment is involved; the agent drives the rest by discipline.

**If your agent software has no multi-agent support**: no problem. A single agent plays the six roles in order, writes each intermediate artifact to a file, and the next phase reads it to continue. Ordering, gates, verification, and memory accumulation all stay the same — you just lose the speed of parallelism. This is not a degraded mode; it's the single-threaded form of the same discipline.

This is **discipline and verifiability, not a system-enforced hook**: it makes violations and unverified claims visible and checkable, but cannot physically stop an agent from bypassing the flow.

Full runtime rules and engineering memory: `SKILL.md`. Six-role detail: `workflows/multi-agent-feature.md`.

## When to use it

**Good fit:**
- Code-engineering work with real scale or multi-person / multi-agent collaboration: new features, refactors, cross-module changes.
- Long-lived projects where rules and decisions must accumulate across sessions instead of getting lost in chat.
- Work that needs an auditable process: who confirmed requirements, who signed off on the stack, whether claims have real output behind them.
- Cases where quality and process discipline matter more than "fastest possible output": production code, open source, team projects.
- Agents with subagent support can let the six roles truly collaborate in parallel; without it, the same flow still runs single-threaded (see "How it works").

**Poor fit / trim as needed:**
- Pure chat, research, or documentation tasks — non-code work doesn't trigger this skill.
- One-shot tiny edits (typo, constant tweak): skip the full handoff set, trim to the minimum.
- Strongly sequential tightly coupled work (e.g. TDD red→green→refactor): single agent, no forced parallelism.
- Expecting system-level enforcement: this is discipline, not a hook — it cannot physically stop an agent from bypassing the flow.

## Directory structure

```
engineering-craft/
├── SKILL.md              # thin entry: navigation, red lines, engineering memory
├── README.md / README.en.md
├── core/                 # general rules: requirements, design, coding, test, verify, review, delivery…
├── workflows/            # feature / bugfix / refactor / investigate / multi-agent
├── languages/            # language rules — read the relevant one before coding
├── topics/               # cross-cutting: security / performance / concurrency / API design / refactoring
└── CHANGELOG.md / LICENSE
```

## License

[MIT](./LICENSE)
