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
T3 remote support is browser/app oriented. Telegram requires a separate ingress/egress bridge.

## Likely implementation surfaces once code is imported
- contracts for orchestration/thread metadata
- server-side decider/projector/projection logic
- CLI command layer
- sidebar/store/types on the frontend
- archive/memory reactor surfaces
- Telegram integration module
