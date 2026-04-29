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

- diagram:
- Plan
- execute
- review

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
