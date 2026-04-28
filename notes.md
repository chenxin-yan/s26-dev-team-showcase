## Notes

### Introduction

- overview of project (**Cyan**)
  - agent orchestration TUI
- brief intro of the members
- introduction to ralph, the methodology (**Frank**)
- live demo by starting a ralph loop with PRD pre-made (**Cyan**)

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

- Plan
- execute
- review

### Technical Details

- Tech stack (**Frank**)
  - Typescript
  - opentui
  - bun
- Daemon (**Cyan**)
- npm packaging (**Cyan**)
- Streaming service for background execution and live log (**Fahim**)
- `.ralph/` workspace & files (**Asia**)
  - instead of treating an agent session as just one long conversation, we give each session its own scaffolded workspace on disk.
  - in the workspace, we create a few specific files with different roles
    - SPEC.md for requirements
    - PROMPT.md for execution rules
    - progress.md for memory and handoff
    - task files like prd.json for planning state
- CLI (**Patrick**)
- ...alex could pick up another technical details to talk about (**Alex**)

### Showcases (**Alex** & **Fahim**)

- show plan view
- show working on multiple projects at once
- execution view & review view
- show the final result for the loop started in the beginning

### Future plan (**Patrick**)

> things we didn't have time to get to but could potentially build in the future

- better review view
- on demand prompts during execution
- add more feature parity to opencode
  - questions
  - manual skill activation
