---
title: '<span style="color: palette:green">Ralph</span> · Agent orchestration in the terminal'
sub_title: '_Dev team showcase — S26_'
event: "Tech@NYU"
date: "Spring 2026"
authors:
  - "<NAME 1> — <ROLE>"
  - "<NAME 2> — <ROLE>"
  - "<NAME 3> — <ROLE>"
theme:
  name: catppuccin-mocha
  override:
    footer:
      style: progress_bar
      character: "▌"
    slide_title:
      separator: true
      padding_top: 1
      padding_bottom: 2
    default:
      margin:
        percent: 7
options:
  end_slide_shorthand: false
  list_item_newlines: 2
---

Meet the team
=============

<!-- alignment: center -->

<span style="color: palette:subtext0">Replace placeholders before you present.</span>

<!-- column_layout: [1, 1, 1] -->

<!-- column: 0 -->

<span style="color: palette:teal">01</span>

- **<NAME 1>**
- `<ROLE>`

<!-- column: 1 -->

<span style="color: palette:mauve">02</span>

- **<NAME 2>**
- `<ROLE>`

<!-- column: 2 -->

<span style="color: palette:peach">03</span>

- **<NAME 3>**
- `<ROLE>`

<!-- reset_layout -->

<!-- end_slide -->

What is Ralph?
==============

<!-- list_item_newlines: 1 -->

> [!tip]
> A **daemon-backed** TUI so agents keep working after you detach.

- <span style="color: palette:sky">Orchestration</span> — one loop across workspaces, not one-off chats
- <span style="color: palette:green">Resilience</span> — `ralphd` owns runs; the TUI is a thin client
- <span style="color: palette:yellow">Visibility</span> — single dashboard for every project in flight

<!-- end_slide -->

Ralph, the methodology
=======================

> [!important]
> **Small sessions, tight tasks** — less noise, fewer hallucinations, cleaner handoffs.

- <span style="color: palette:lavender">Plan → execute → review</span> as the default rhythm
- **One task per session** — select from `prd.json`, ship, signal completion
- **`progress.md` append-only** — each iteration leaves breadcrumbs for the next

<!-- end_slide -->

<!-- jump_to_middle -->

<!-- alignment: center -->

<span style="color: palette:yellow">Live</span>

Live demo
=========

- Spin up a **Ralph loop** with a pre-baked PRD
- Watch **plan → run → log** without babysitting the TUI

<!-- end_slide -->

Problems with agentic coding today
===================================

| <span style="color: palette:sky">Lens</span> | <span style="color: palette:text">What goes wrong</span> |
| --- | --- |
| **Tooling** | Server + TUI fused — quit the UI, kill the agent |
| **Tooling** | Multi-repo work — context scattered across windows |
| **Methodology** | Mega-sessions — context rot and weaker verification |
| **Methodology** | Vague / declarative goals — many paths, easy drift |

<!-- end_slide -->

Opencode pain #1: TUI = agent lifetime
========================================

> [!warning]
> The agent's lifetime is tied to the **foreground TUI** — no true background mode.

- Disconnect or close the UI → **run stops**
- Hard to pair a **lightweight client** with long-running work

<!-- end_slide -->

Our fix: TUI + daemon
======================

> [!tip]
> **Split control plane from data plane** — `ralphd` keeps state; the TUI attaches when you want.

- <span style="color: palette:green">Unix socket</span> `~/.ralph/ralphd.sock` — fast local IPC
- <span style="color: palette:green">SQLite</span> `~/.ralph/state.sqlite` — durable orchestration state
- **CLI** for health checks without opening the UI

```bash +line_numbers
bun run src/cli.ts daemon start
bun run src/cli.ts daemon health
bun run src/cli.ts daemon stop
```

<!-- end_slide -->

Opencode pain #2: multi-project workflow
========================================

> [!warning]
> Every repo becomes its **own island** of terminals, tabs, and agent state.

- Heavy **context switching** between codebases
- No **single pane of glass** for “what is every agent doing right now?”

<!-- end_slide -->

Our fix: one TUI for all projects
==================================

> [!tip]
> **One mental model** everywhere — same plan / execute / review flow per workspace.

- Unified **dashboard** across Ralph workspaces
- **At-a-glance** status: what’s running, what’s blocked, what’s next

<!-- end_slide -->

Methodology pain #1: the context problem
=========================================

> [!caution]
> Stuffing everything into **one session** trades convenience for **signal-to-noise**.

- More tokens → more **noise** → more **hallucination risk**
- Harder to **verify**, **review**, and **hand off** cleanly

<!-- end_slide -->

Methodology pain #2: declarative prompts drift
===============================================

> [!caution]
> “Build the feature” has **many valid implementations** — agents wander without guardrails.

- Plan-only / goal-only prompts → **under-constrained** search space
- Without **task boundaries**, reviews get fuzzy

<!-- end_slide -->

Product overview: Plan → Execute → Review
==========================================

<!-- column_layout: [1, 1, 1] -->

<!-- column: 0 -->

### <span style="color: palette:sapphire">Plan</span>

- Shape **tasks** + acceptance in `prd.json`
- Ground truth in **`SPEC.md`**

<!-- column: 1 -->

### <span style="color: palette:green">Execute</span>

- **`ralphd`** drives the loop
- **Streaming logs** while work runs in the background

<!-- column: 2 -->

### <span style="color: palette:peach">Review</span>

- Inspect **output** before the next iteration
- **Roadmap:** richer diff / test surfacing in-review

<!-- reset_layout -->

<!-- end_slide -->

Tech stack
==========

| Layer | Choice |
| --- | --- |
| Language | **TypeScript** |
| UI | **OpenTUI** — `@opentui/core`, `@opentui/react` |
| Runtime | **Bun** |
| Monorepo | **Turborepo** (dev orchestration) |

<!-- end_slide -->

The daemon
==========

> [!note]
> **`ralphd`** is the source of truth for sessions, sockets, and persistence.

| Resource | Path / role |
| --- | --- |
| Socket | `~/.ralph/ralphd.sock` |
| State DB | `~/.ralph/state.sqlite` |
| Role | Spawn runs, stream output, survive without the TUI |

<!-- end_slide -->

Streaming service
=================

- **Background execution** — detach the TUI without losing the agent
- **Live log fan-in** — stdout/stderr surfaces in the client
- **Multi-project fan-out** — same pipeline for every workspace

<!-- end_slide -->

`.ralph/` workspace
===================

> [!note]
> The **contract** between humans and agents lives on disk — inspectable, diffable, versioned.

| File | Job |
| --- | --- |
| `SPEC.md` | What you are building |
| `prd.json` | Task list + pass / fail |
| `progress.md` | Append-only iteration log |
| `PROMPT.md` | One-task-per-session agent rules |

```markdown +line_numbers
1. SPEC.md — what you're building
2. prd.json — task list
3. progress.md — append-only log
```

<!-- end_slide -->

npm packaging
=============

> [!tip]
> Ship it like any other CLI — **`@techatnyu/ralph`** on the public registry.

- **Package:** `@techatnyu/ralph` · `publishConfig.access: public`
- **UX:** end users run `ralph`; packaged builds can **spawn `ralphd`** automatically
- **Workspace link:** `@techatnyu/ralphd` via `workspace:*` in dev

<!-- end_slide -->

<!-- jump_to_middle -->

<!-- alignment: center -->

<span style="color: palette:mauve">Showcase</span>

Showcase: Plan view
===================

- Trace **PRD → task pick → spec context** in the TUI
- Call out how **`prd.json`** steers the next session

<!-- end_slide -->

<!-- jump_to_middle -->

<!-- alignment: center -->

<span style="color: palette:mauve">Showcase</span>

Showcase: Multiple projects at once
====================================

- Flip between workspaces **without losing narrative**
- Compare **agent state** side-by-side in one surface

<!-- end_slide -->

<!-- jump_to_middle -->

<!-- alignment: center -->

<span style="color: palette:mauve">Showcase</span>

Showcase: Execution & Review views
===================================

- **Execution** — streaming output, background-friendly runs
- **Review** — what landed this iteration + what’s queued next

<!-- end_slide -->

What's next
===========

> [!note]
> Honest backlog — the **next layer of polish**, not vapor.

- **Review surface** — richer diffs, tests, approvals inline
- **Prompt injection mid-run** — steer without restarting the loop
- **Opencode parity** — Q&A flows, manual skill activation

<!-- end_slide -->

<!-- column_layout: [1, 3, 1] -->

<!-- column: 1 -->

<!-- jump_to_middle -->

<!-- alignment: center -->

<!-- no_footer -->

<span style="color: palette:green">Thank you</span>

Thanks / Q&A
============

- Repo: **`TechAtNYU/ralph`**
- <span style="color: palette:subtext0">We’d love your questions.</span>

<!-- reset_layout -->

<!-- end_slide -->
