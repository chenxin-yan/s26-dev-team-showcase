## Notes

### Introduction

- overview of project (**Cyan**)
  - agent orchestration TUI
- live demo by starting a ralph loop with PRD pre-made (**Cyan**)
- brief intro of the members
- introduction to ralph, the methodology (**Frank**)
  - some next.js app

### Problems & solutions (**Asia**)

> problems of opencode or agentic coding in general without ralph

- opencode
  - you cannot decouple the server and the tui. whenever you quit the TUI, the agent will stop too
    - solution: we built a TUI + daemon that allows the agent to run despite the tui is not running
  - Its hard to work on multiple projects at the same time even with an IDE
    - solution: ralph tui gives one single view for all the projects you are working
- methodology
  - the context problem: trying to fix everything into one session is bad for agent. more noices means more hallucination and worse performance
  - vagueness of the prompts: with normal prompting or plan mode. It is declarative instead of imperative so that the agent can take different paths to achieve the same thing which cloud go out of the hand

### Product overview (**Kevin**)

Product-wise, the important framing is that Ralph is not just another coding agent UI. The goal is to turn agentic coding into a structured workflow. If an agent can run for a long time, touch real files, and continue across sessions, the user needs more than a chat box. They need a way to define the work, manage the work, and review the work.

That is why Ralph is organized around three main tabs: Plan, Execute, and Review.

Plan is where the user turns a high-level idea or PRD into concrete project artifacts. Instead of starting with an open-ended instruction like “build this feature,” Ralph helps move the project into a shared `SPEC.md` and a task-focused `prd.json`.

Execute is where those plan files become active workstreams. The user can see what is currently running, what state each task is in, and where the agent is in the process. This turns agent execution from a black-box chat into something observable and manageable.

Review is where the user comes back into the loop. They inspect the output, understand what changed, and decide whether the work should be accepted, revised, or continued into the next task.

So the product flow is simple: define the work, manage the work, and review the work. Ralph gives structure to agentic coding so the agent can move fast without the human losing control of the process.

### Showcases (**Alex** & **Fahim**)

- TODO: add screenshots before presentation
- show plan view and working on multiple projects at once (**Fahim**)
  - run ralph in two different directories
- execution view & review view (**Alex**)

### Technical Details

- Tech stack (**Frank**)
  - Typescript
  - opentui
  - bun
- Daemon
  - daemon intro: what is it (**Fahim**)
  - Streaming service for background execution and live log (**Fahim**)
  - Shared opencode runtime (**Alex**)
    - we run one opencode process across every workspace
    - each instance gets its own event stream tagged by directory
- `.ralph/` workspace & files (**Asia**)
  - instead of treating an agent session as just one long conversation, we give each session its own scaffolded workspace on disk.
  - in the workspace, we create a few specific files with different roles
    - SPEC.md for requirements
    - PROMPT.md for execution rules
    - progress.md for memory and handoff
    - task files like prd.json for planning state
- CLI (**Patrick**)
- npm packaging (**Cyan**)

### Future plan (**Patrick**)

> things we didn't have time to get to but could potentially build in the future

- better review view
  - curr: just viewing diffs
  - want: interactive rebase
- on demand prompts during execution (steering)
- add more feature parity to opencode
  - manual skill activation
  - general commands: /reset, /undo, etc.

- show the final result for the loop started in the beginning (**Cyan**)
