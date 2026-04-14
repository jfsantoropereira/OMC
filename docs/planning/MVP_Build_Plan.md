# OMC MVP Build Plan

## Phase 0 — Foundation
- fork or clone `pingdotgg/t3code`
- keep the T3 orchestration/event model intact
- add OMC-specific contracts instead of forking core semantics unnecessarily

## Phase 1 — Agent Graph
- add thread metadata for:
  - agent kind
  - parent thread
  - root thread
  - owner thread
  - spawn purpose
  - lifecycle status
- add migrations + projections + selectors
- surface parent/child relationships in UI

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
- add command family for:
  - `whoami`
  - `context`
  - `agents list/show/tree`
  - `message send`
  - `spawn create`
  - `request spawn`
  - `admin list/grant/revoke`
  - `interrupt`
  - `archive`
  - `unarchive`
  - `stop`
  - `self status/handoff/request-help`
- ensure CLI can target remote/headless T3 server state cleanly
- enforce server-side authorization by actor role, including the rule that Admins cannot archive fellow Admins

## Phase 4 — Master Agent
- create a master control project/thread model
- add a master dashboard / registry of active agents
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
- thread metadata for agent hierarchy
- Admin designation model
- `hatch whoami`
- `hatch agents list`
- `hatch admin list`
- `hatch spawn create`
- `hatch request spawn`
