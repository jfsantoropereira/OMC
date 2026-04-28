Status: superseded by canonical OMC docs.

Canonical docs:

- [../../README.md](../../README.md)
- [../../SystemOutline.md](../../SystemOutline.md)

This file is historical context only. Do not treat it as current architecture or build plan. Current architecture: Hermes is the external master/operator; OMC is a thin T3 Code thread substrate; `hatch` is the programmatic T3 control surface; no in-app Master Agent; no Admin Agent role; minimal/no UI changes for MVP.

---

# OMC on top of T3 Code — Gap Analysis

Research baseline: `pingdotgg/t3code` at commit `0d280262f6c9dcaf529376b2007dc7447f86fa3c`.

## What T3 Code already provides
- event-sourced orchestration core with projects and threads
- thread lifecycle primitives such as create, archive, unarchive, interrupt, and session stop
- bootstrap helpers for creating threads, worktrees, and setup flows
- remote pairing and remote environment access over HTTP/WebSocket
- archived-thread UI patterns
- context-compaction signals that can later feed master-memory ingestion

## What OMC still needs

### 1. Agent hierarchy model
T3 has projects and threads, but not the OMC agent graph. OMC still needs:
- agent hierarchy metadata
- parent/child relationships
- spawn-purpose and ownership metadata
- archive semantics that preserve operational clarity

### 2. Master control plane
T3 has no global privileged Master Agent abstraction above all projects. OMC still needs:
- a Master control surface
- cross-project visibility
- role-aware agent management commands

### 3. Admin designation model
The current OMC design requires:
- multiple Admin Agents per project
- project-level admin designation
- Admin spawn privilege only within project scope
- Admin archive authority only over non-admin agents in the same project

### 4. Hatch CLI
T3 has project and orchestration primitives, but not the Hatch CLI policy layer for Master/Admin/Regular interaction.

### 5. Archive-to-memory workflow
T3 has archiving, but OMC still needs:
- `thread_resume.md` generation
- long-term memory ingestion hooks
- admin-state cleanup on archive where applicable

### 6. Telegram bridge
T3 remote support is browser/app oriented. Telegram requires a separate ingress/egress bridge. MVP: all Telegram messages route to Master; Master relays downstream and returns results upstream.

### 7. Notification/wakeup system
T3 has no concept of agents notifying their parents on completion. OMC needs:
- turn-complete event reactor that emits signal to parent thread
- signal format: `[subagent:<threadId> completed]` (token-efficient, parent pulls details via CLI)
- auto-drain of upstream notifications to parent

### 8. Message queue
T3 delivers messages directly. OMC needs per-thread inbox queuing:
- queue when target is busy
- auto-drain downstream (parent→child)
- `--interrupt` flag to bypass queue
- prevents message loss during concurrent agent activity

### 9. Heartbeat scheduler
T3 has no periodic wakeup mechanism. OMC needs:
- configurable interval + injected prompt (stored in app settings)
- fires while app is running; stops on explicit quit
- primary use: Master periodic check-in

## Likely implementation surfaces once code is imported
- contracts for orchestration/thread metadata
- server-side decider/projector/projection logic
- CLI command layer
- sidebar/store/types on the frontend
- archive/memory reactor surfaces
- Telegram integration module
