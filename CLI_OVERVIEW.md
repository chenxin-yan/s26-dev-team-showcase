# Ralph CLI: Comprehensive Technical Overview

> **Package:** `@techatnyu/ralph`  
> **Branch:** `cli` (extends `main`)  
> **Purpose:** Command-line interface for Ralph — a coding agent orchestration system

---

## 1. Executive Summary

The Ralph CLI is the primary user-facing interface for managing coding agent loops via the Ralph daemon (`ralphd`). It supports two modes:

- **Interactive Mode:** Launches the TUI when run without arguments (`ralph`)
- **Command Mode:** Executes one-shot CLI commands for automation, scripting, and headless environments

The CLI is built on `@crustjs/core` (a command-line framework) and communicates with `ralphd` via Unix domain sockets using a typed JSON-RPC protocol.

---

## 2. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     USER INTERFACE LAYER                     │
│  ┌──────────────┐  ┌──────────────────────────────────────┐ │
│  │   ralph CLI  │  │  ralph TUI (React + @opentui)        │ │
│  │  (apps/tui)  │  │  Interactive chat interface          │ │
│  └──────┬───────┘  └──────────────────────────────────────┘ │
└─────────┼───────────────────────────────────────────────────┘
          │ Unix Domain Socket (JSON-RPC)
          ▼
┌─────────────────────────────────────────────────────────────┐
│                    RALPH DAEMON (ralphd)                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Job Queue    │  │ Instance Mgr │  │ OpenCode Runtime │  │
│  │ Management   │  │ (start/stop) │  │ (SDK Bridge)     │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Communication Flow

1. CLI validates and parses user input via `@crustjs/validate/zod`
2. CLI sends JSON-RPC request over Unix socket to `ralphd`
3. `ralphd` processes request (queue job, manage instance, etc.)
4. `ralphd` returns structured response
5. CLI formats output (pretty-printed tables, colored text, or JSON)

---

## 3. Command Reference (Complete)

### 3.1 Core Commands

| Command | Description | Interactive |
|---------|-------------|-------------|
| `ralph` | Launch the TUI | Yes |
| `ralph setup` | Run guided first-time setup | Yes (optional `--non-interactive`) |
| `ralph setup --json` | Output setup diagnostics as JSON | No |
| `ralph setup --non-interactive` | Skip all prompts, exit codes reflect status | No |
| `ralph doctor` | Check prerequisites (OpenCode, auth, daemon) | No |
| `ralph doctor --json` | JSON output of prerequisite checks | No |
| `ralph --version` | Show package version | No |
| `ralph help` | Show command help (alias for `--help`) | No |

### 3.2 Daemon Lifecycle

| Command | Description | Flags |
|---------|-------------|-------|
| `ralph daemon serve` | Run daemon in foreground (blocks) | — |
| `ralph daemon start` | Start daemon in background (idempotent) | `--json` |
| `ralph daemon stop` | Stop the running daemon | `--json` |
| `ralph daemon health` | Show PID, uptime, queue stats | — (always JSON) |

**Idempotency:** `daemon start` is safe to run multiple times. If already running, it reports the existing PID.

### 3.3 Job Management

| Command | Description | Required Args | Optional Flags |
|---------|-------------|---------------|----------------|
| `ralph daemon submit <prompt...>` | Submit a prompt job | `--instance <id>` | `--session <id>` |
| `ralph daemon list` | List all jobs | — | `--instance`, `--state` |
| `ralph daemon get <jobId>` | Get job details | `jobId` | — |
| `ralph daemon cancel <jobId>` | Cancel a queued/running job | `jobId` | — |

**Job States:** `queued`, `running`, `succeeded`, `failed`, `cancelled`

### 3.4 Instance Management

| Command | Description | Required Args | Optional Flags |
|---------|-------------|---------------|----------------|
| `ralph daemon instance create <name>` | Register a new workspace instance | `--directory <path>` | `--max-concurrency <n>` |
| `ralph daemon instance list` | List all registered instances | — | — |
| `ralph daemon instance get <id>` | Get instance details | `instanceId` | — |
| `ralph daemon instance start <id>` | Start an instance | `instanceId` | — |
| `ralph daemon instance stop <id>` | Stop an instance (fails if jobs running) | `instanceId` | — |
| `ralph daemon instance remove <id>` | Remove an instance (fails if active jobs) | `instanceId` | — |

### 3.5 Provider Management

| Command | Description | Flags |
|---------|-------------|-------|
| `ralph provider list` | List configured providers in a table | `--refresh`, `--directory`, `--json` |

**Provider Status:** Shows `connected` (green) or `disconnected` (red) with model counts.

### 3.6 Model Management

| Command | Description | Flags |
|---------|-------------|-------|
| `ralph model set <provider/model>` | Set active model (e.g., `anthropic/claude-sonnet-4-5`) | `--json` |
| `ralph model get` | Show currently selected model | `--json` |

---

## 4. Code Structure

### 4.1 Key Files

```
apps/tui/src/
├── cli.ts                      # Main CLI definition (Crust framework)
├── index.tsx                   # TUI entry point (React renderer)
│
├── lib/
│   ├── cli-output.ts           # Output formatters (tables, colors, JSON)
│   ├── cli-version.ts          # Version resolution from package.json
│   ├── onboarding.ts           # Prerequisite checks (OpenCode, auth, daemon)
│   ├── onboarding.test.ts      # Unit tests for onboarding logic
│   ├── providers.ts            # Provider sorting, model listing utilities
│   ├── store.ts                # Local config persistence (model selection)
│   ├── worktree.ts             # Git worktree utilities
│   └── scaffold.ts             # Workspace scaffolding
│
└── __tests__/
    └── cli.e2e.test.ts         # End-to-end CLI tests
```

### 4.2 Framework Dependencies

| Package | Purpose |
|---------|---------|
| `@crustjs/core` | CLI framework (command routing) |
| `@crustjs/plugins` | Built-in plugins (help, version, autocomplete) |
| `@crustjs/validate` | Zod-based flag validation |
| `@crustjs/progress` | Loading spinners for long operations |
| `@crustjs/prompts` | Interactive model selection |
| `@crustjs/style` | Terminal colors, bold, tables |

---

## 5. Features & Capabilities

### 5.1 JSON API Mode

Nearly every command supports `--json` for scripting and CI/CD integration:

```bash
# Machine-readable setup check
ralph doctor --json
# → { "ok": true, "checks": [...] }

# Programmatic daemon startup
ralph daemon start --json
# → { "ok": true, "alreadyRunning": false, "health": { "pid": 12345, ... } }

# Get model selection for scripts
ralph model get --json
# → { "model": "anthropic/claude-sonnet-4-5" }
```

### 5.2 Interactive Onboarding (`ralph setup`)

1. **Check OpenCode Installation:** Verifies `opencode --version` is in PATH
2. **Check OpenCode Auth:** Runs `opencode auth list` to confirm credentials
3. **Auto-start Daemon:** Calls `ensureDaemonRunning()` if `--non-interactive` not set
4. **Refresh Providers:** Fetches latest provider metadata from OpenCode SDK
5. **Model Selection:** If no model is set, presents an interactive picker of connected models

**Exit Codes:**
- `0` — All checks passed, daemon running, ready to use
- `1` — One or more checks failed (see `--json` for details)

### 5.3 Typo Detection & Autocomplete

The CLI includes fuzzy matching for typos:

```bash
$ ralph deamon
# → Did you mean "daemon"?
# → Exits with code 1
```

### 5.4 Pretty Output Formatting

Human-facing commands use formatted tables and colors:

```
Ralph Setup
OK   OpenCode installed
OK   OpenCode authenticated
OK   Daemon running (pid 12345, uptime 42s)

Active model: anthropic/claude-sonnet-4-5
1 connected provider(s), 4 total provider(s) detected.
Ralph is ready.
Run `ralph` to launch the TUI.
```

**Provider Table:**
```
Providers
┌────────────────────┬─────────────┬────────┐
│ Provider           │ Status      │ Models │
├────────────────────┼─────────────┼────────┤
│ Anthropic (anthrop)│ connected   │ 4      │
│ OpenAI (openai)    │ disconnected│ 12     │
└────────────────────┴─────────────┴────────┘
Connected: anthropic
```

---

## 6. Daemon Protocol

The CLI communicates with `ralphd` via a typed JSON-RPC protocol over Unix sockets.

### 6.1 Request Structure

```typescript
{
  id: string;        // UUID
  method: "job.submit" | "instance.create" | "provider.list" | ...;
  params: { ... };   // Method-specific parameters
}
```

### 6.2 Response Structure

```typescript
// Success
{
  id: string;
  method: string;
  ok: true;
  result: { ... }
}

// Error
{
  id: string;
  method: string;
  ok: false;
  error: {
    code: "invalid_json" | "invalid_request" | "not_found" | "conflict" | "instance_unavailable" | "shutdown" | "internal";
    message: string;
    issues?: Array<{ path: string; message: string; code: string }>;
  }
}
```

### 6.3 Methods

| Method | Description |
|--------|-------------|
| `daemon.health` | Get daemon PID, uptime, queue stats |
| `daemon.shutdown` | Gracefully stop daemon |
| `instance.create` | Register a new workspace instance |
| `instance.list` | List all instances |
| `instance.get` | Get single instance |
| `instance.start` | Start instance runtime |
| `instance.stop` | Stop instance runtime |
| `instance.remove` | Delete instance |
| `provider.list` | Query OpenCode SDK for providers |
| `job.submit` | Enqueue a new prompt job |
| `job.list` | List jobs (filterable) |
| `job.get` | Get job details |
| `job.cancel` | Cancel a queued/running job |

### 6.4 Job Execution Flow

```
User: ralph daemon submit "Refactor auth" --instance <id>
  │
  ▼
CLI → daemon.submitJob({ instanceId, session: { type: "new" }, task: { type: "prompt", prompt: "..." } })
  │
  ▼
ralphd queues job → starts instance → creates OpenCode session → calls session.prompt() → stores result
  │
  ▼
User polls: ralph daemon get <jobId>
Response: { state: "succeeded", outputText: "I've refactored..." }
```

---

## 7. Setup & Onboarding Flow (Detailed)

### 7.1 Prerequisites Check

The onboarding system (`lib/onboarding.ts`) validates three things:

| Check | Command Used | Failure Message |
|-------|--------------|-----------------|
| OpenCode Installed | `opencode --version` | "`opencode` is not installed or not in PATH" |
| OpenCode Authenticated | `opencode auth list` | "No auth configured. Run `opencode auth login`" |
| Daemon Running | `daemon.health()` or `ensureDaemonRunning()` | "ralphd is not running" |

### 7.2 Setup Variants

**Interactive (default):**
```bash
$ ralph setup
# → Runs checks with spinner animations
# → If connected: "Choose a default model" interactive picker
# → Pretty-printed summary
```

**Non-interactive (CI/CD):**
```bash
$ ralph setup --non-interactive
# → No prompts, no spinners
# → Exit code 0 or 1
```

**JSON (automation):**
```bash
$ ralph setup --json
# → Full machine-readable output including check details
```

---

## 8. Comparison: CLI Branch vs. Main Branch

| Feature | `main` Branch | `cli` Branch |
|---------|---------------|--------------|
| `ralph` (no args) | Launches TUI | Launches TUI |
| `ralph setup` | **Does not exist** | Full onboarding + model picker |
| `ralph doctor` | **Does not exist** | Prerequisite checks |
| `ralph provider list` | **Does not exist** | Table-formatted provider list |
| `--version` | **Does not exist** | Shows package version |
| `--json` flags | Rare | Most commands support it |
| Typo handling | Falls through to TUI | Autocomplete suggestions + exit 1 |
| Validation | Basic | Zod schema validation |
| Output formatting | Inline `console.log` | Centralized formatters |
| `help` alias | **Does not exist** | `ralph help` works |
| Daemon `start`/`stop` | Plain text only | Added `--json` support |
| Model `set`/`get` | Plain text only | Added `--json` support |

**Key Principle:** The CLI branch is purely additive — no commands were removed from `main`.

---

## 9. Usage Examples

### 9.1 First-Time Setup

```bash
# Full interactive setup
ralph setup

# Verify everything is working
ralph doctor

# Set a model manually
ralph model set anthropic/claude-sonnet-4-5
```

### 9.2 Running Jobs (Headless/Scripting)

```bash
# Start daemon
ralph daemon start

# Create an instance for your project
ralph daemon instance create my-app --directory ./my-app

# Submit a task
JOB=$(ralph daemon submit "Refactor auth to use JWT" --instance <id> --json)
JOB_ID=$(echo $JOB | jq -r '.job.id')

# Poll for completion
while true; do
  STATE=$(ralph daemon get $JOB_ID --json | jq -r '.job.state')
  [ "$STATE" = "succeeded" ] && break
  [ "$STATE" = "failed" ] && exit 1
  sleep 2
done

# Get result
ralph daemon get $JOB_ID --json | jq -r '.job.outputText'
```

### 9.3 CI/CD Integration

```bash
# In a GitHub Action — non-interactive, JSON output
ralph setup --non-interactive || exit 1
ralph doctor --json | jq '.ok' | grep -q true

# Submit a review job
ralph daemon submit "Review this PR for security issues" \
  --instance production-instance
```

### 9.4 Provider Inspection

```bash
# See what's available
ralph provider list

# Refresh if you just added credentials
ralph provider list --refresh

# Get raw data
ralph provider list --json | jq '.connected[]'
```

---

## 10. Testing

### 10.1 E2E Tests (`apps/tui/src/__tests__/cli.e2e.test.ts`)

Tests run the actual CLI with `Bun.spawn` in isolated temp directories:

| Test | Validates |
|------|-----------|
| Daemon start idempotency | Same PID across multiple starts |
| Version flag | Outputs `ralph v0.0.1` |
| Help command alias | Contains "Coding agent orchestration TUI and CLI" |
| JSON output | `daemon start --json` returns valid JSON |
| Model CRUD | `model set` + `model get --json` round-trip |
| Typo detection | `ralph deamon` suggests "daemon" |

### 10.2 Onboarding Tests (`lib/onboarding.test.ts`)

Unit tests with mocked command runners:
- OpenCode installed but not authenticated
- Daemon auto-start behavior
- All-checks-passing scenario
- Timeout handling for commands

---

## 11. Error Handling & Exit Codes

| Scenario | Exit Code | Behavior |
|----------|-----------|----------|
| Success | `0` | Normal completion |
| Setup/doctor failure | `1` | One or more checks failed |
| Unknown command | `1` | Typo detected, suggestions shown |
| Daemon not running | **Error thrown** | Message: "ralphd is not running. Start it with: ralph daemon start" |
| Invalid model format | **Error thrown** | Message: "Invalid model format. Use provider/model" |
| Instance conflict | **Error thrown** | Message: "instance has running jobs and cannot be stopped" |

---

## 12. Future Capabilities (What Exists in Protocol But Not CLI)

The daemon protocol supports features not yet exposed in the CLI:

| Protocol Feature | CLI Exposure | Potential Command |
|------------------|--------------|-------------------|
| Job task `agent` field | Hidden | `ralph daemon submit --agent <name>` |
| Job task `system` prompt | Hidden | `ralph daemon submit --system "You are a..."` |
| Job task `variant` | Hidden | `ralph daemon submit --variant <variant>` |
| Session title on creation | Hidden | `ralph daemon submit --title "Bug fix"` |
| Chat/REPL loop | **Not implemented** | `ralph chat --instance <id>` |
| Stream job output | **Not implemented** | Real-time logs instead of polling |
| Job retry | **Not implemented** | `ralph daemon retry <jobId>` |
| Batch job submission | **Not implemented** | `ralph daemon batch jobs.json` |

---

## 13. Key Design Decisions

1. **No TUI Required for Automation:** Every meaningful operation is available as a CLI command with JSON output, enabling CI/CD pipelines and editor integrations.

2. **Daemon-Centric Architecture:** The CLI is stateless. All persistent state (instances, jobs, sessions) lives in `ralphd`, enabling multiple CLI/TUI clients to interact with the same daemon.

3. **Idempotent Operations:** `daemon start`, `instance start`, and `setup` are all safe to run repeatedly.

4. **Graceful Degradation:** `setup` can run in `--non-interactive` mode. `doctor` never auto-starts the daemon.

5. **Pretty by Default, JSON on Demand:** Human users get colored tables and summaries; scripts get `--json`.

---

## 14. Package Metadata

```json
{
  "name": "@techatnyu/ralph",
  "version": "0.0.1",
  "type": "module",
  "dependencies": {
    "@crustjs/core": "^0.0.16",
    "@crustjs/plugins": "^0.0.21",
    "@crustjs/progress": "^0.0.2",
    "@crustjs/prompts": "^0.0.12",
    "@crustjs/validate": "^0.0.15",
    "@techatnyu/ralphd": "workspace:*",
    "zod": "^4.3.6"
  }
}
```

---

*Document generated from branch `cli`, commit `ba1fa4e` — `apps/tui/src/cli.ts` and related modules.*
