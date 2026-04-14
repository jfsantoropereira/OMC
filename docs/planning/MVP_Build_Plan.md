# OMC MVP Build Plan

## Phase 0 — Foundation
- **fork** `pingdotgg/t3code` (MIT license — rebrand and commercial use allowed, keep LICENSE with original copyright)
- diverge from upstream — no ongoing sync, treat as one-time snapshot
- keep the T3 orchestration/event model intact
- add OMC-specific contracts instead of forking core semantics unnecessarily
- inherit: Bun, Effect-TS, Turborepo, React+Vite, SQLite event store, Electron packaging

## Phase 1 — Agent Graph + Message Queue
- add thread metadata for:
  - agent kind
  - parent thread
  - root thread
  - owner thread
  - spawn purpose
  - spawn model (`provider/version`, e.g. `claude/latest`, `codex/gpt-5.4-high`)
  - lifecycle status
- add migrations + projections + selectors
- surface parent/child relationships in UI
- add **per-thread message queue** infrastructure:
  - inbox queue for incoming messages from other agents
  - if thread idle: deliver immediately (inject as user-turn via existing T3 mechanism)
  - if thread busy: queue message
  - auto-drain downstream messages (parent→child) when thread becomes idle
  - `--interrupt` flag support: bypass queue, interrupt current generation

## Phase 2 — Admin Designation Model
- add a project-level field or equivalent event flow for `adminThreadIds`
- add commands for:
  - grant admin
  - revoke admin
  - list admins
  - enforce: admins may archive only non-admin agents in their own project
  - auto-remove admin on archive of that thread
- render Admin Agents pinned near the top of the project section in the sidebar

## Phase 3 — Hatch CLI
- **system-wide binary on PATH** (like `notes` CLI)
- **token-efficient output** — terse, no banners, minimal whitespace
- **`-help` on every command** for agent self-discovery
- add command family for:
  - `whoami`
  - `context`
  - `agents list` / `agents running` / `agents waiting` / `agents archived` / `agents show <id>` / `agents tree`
  - `message send <threadId> [--interrupt] "message"`
  - `spawn create --model <provider/version>` (e.g. `--model claude/latest`)
  - `request spawn`
  - `admin list/grant/revoke`
  - `interrupt <threadId>`
  - `archive`
  - `unarchive`
  - `stop`
  - `self status/handoff/request-help`
- ensure CLI can target remote/headless T3 server state cleanly
- enforce server-side authorization by actor role, including the rule that Admins cannot archive fellow Admins
- add **notification/wakeup routing**:
  - on subagent turn-complete → emit signal `[subagent:<threadId> completed]` to parent thread
  - signal delivered via message queue (auto-drains upstream to parent)
  - parent uses `hatch agents show <threadId>` to pull details if needed

## Phase 4 — Master Agent
- create a hidden "OMC Control" project with `workspaceRoot: ~` (user home directory)
- Master thread lives in this project, rendered above Projects in sidebar
- Master can access any project, any file on the machine
- no separate dashboard view — Master is a normal chat thread; agent status lives in `hatch` CLI
- add **heartbeat scheduler**:
  - configurable in app settings (interval + injected prompt)
  - e.g. every 4 hours: inject "check on things" into Master thread
  - only fires while app is running (app stays active even with screen locked, stops on explicit quit)
- add orchestration helpers so the master can operate on child threads directly

## Phase 5 — Archive and Memory
- on `thread.archived`, generate `thread_resume.md`
- if archived thread is an Admin, clear that designation
- persist memory artifacts through `notes`
- add a compaction hook to write high-value summaries to master memory

## Phase 6 — Telegram MVP
- Telegram bot receives inbound text/commands
- bridge routes messages into the master thread or targeted agent thread
- bridge supports Admin-aware command routing
- return assistant results back to Telegram
- add simple allowlist/authz for bot users

## Phase 7 — Hardening
- permissions model for who can operate master controls
- failure recovery for bot/websocket disconnects
- registry cleanup for stale threads
- memory dedupe and summarization quality controls

## Suggested First Code Spike
Start with:
- fork T3 Code, verify build + Electron packaging works
- thread metadata for agent hierarchy (including `spawnModel` field)
- per-thread message queue infrastructure
- Admin designation model
- `hatch whoami`
- `hatch agents list` / `hatch agents running`
- `hatch admin list`
- `hatch spawn create --model <provider/version>`
- `hatch request spawn`
- `hatch message send <threadId> "message"`
