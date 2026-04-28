# OMC — One Man Company

Status: canonical entrypoint.

Repository: https://github.com/jfsantoropereira/OMC.git
Local path: `/Users/joaofelipe/Desktop/OMC`

## Status

Accepted architecture:

- OMC is a fork/derivative of T3 Code with agent-native spawn/orchestration tools.
- OMC is the UI/runtime surface, not the master agent and not a bundled copy of any external agent runtime.
- OMC should connect to external CLI agents such as Hermes, OpenClaw, Claude Code, Codex, or other compatible tools through explicit command adapters.
- OMC should preserve the current T3 Code project/thread/chat UX as much as possible.
- Spawned/managed agent work should reflect in the UI as normal T3 Code threads, turns, sessions, status, archive/resume, and worktree activity.
- No Hermes install, memory layer, Telegram gateway, or Hermes-specific runtime should be vendored into this repo.
- No in-app Master Agent or Admin Agent hierarchy for MVP.

## Goal

Build OMC as T3 Code plus agent-native spawn tools.

The useful primitive is not an internal boss agent. The useful primitive is a UI-capable project/thread runtime that can start, inspect, message, stop, archive, and resume external CLI-agent sessions while showing their work in the existing T3 Code interface.

OMC should make external agents feel native to T3 Code:

- create agent-backed threads;
- route prompts/messages to a configured CLI agent;
- show responses, tool events, status, and artifacts in the thread UI;
- configure model/agent/runtime/worktree per thread;
- interrupt/stop/archive/unarchive/resume sessions;
- preserve normal T3 Code project and thread behavior.

## Core Thesis

Do not embed Hermes or any single orchestrator into OMC.

OMC is the host application and UI substrate. Hermes, OpenClaw, Claude Code, Codex, and similar tools are external execution backends reachable through CLI adapters. OMC should provide a clean agent-spawn/control abstraction over T3 Code primitives without privileging one agent runtime as part of the project.

## Control Model

### OMC/T3 UI

OMC owns the user-visible project/thread interface.

Responsibilities:

- preserve T3 Code's current project/thread/chat UX;
- represent external CLI-agent sessions as normal threads/turns where possible;
- expose lifecycle state: running, stopped, interrupted, archived, resumable;
- expose artifacts, summaries, worktree state, and command logs in the UI when available;
- avoid new dashboard/UI concepts until the existing thread UI proves insufficient.

### Agent adapters

Agent adapters are the boundary between OMC and external CLI agents.

Examples:

- Hermes adapter;
- OpenClaw adapter;
- Claude Code adapter;
- Codex adapter;
- future local/remote agent adapters.

Responsibilities:

- spawn the selected CLI agent with explicit environment and working directory;
- stream messages/events back into OMC;
- expose stop/interrupt/resume semantics where the backend supports them;
- keep backend-specific state out of OMC unless needed for thread/session mapping;
- avoid global installs or default paths unless explicitly configured.

### ospawn / hatch CLI

Working name: `ospawn` for the agent-native spawn/control tool. `hatch` may remain as an older planning name or internal alias, but the product concept is OMC-native spawning, not Hermes-specific control.

The CLI/API should expose T3-like operations:

- list/show projects;
- list/show threads;
- spawn/create an agent-backed thread;
- message an existing thread;
- configure agent/model/runtime/worktree;
- interrupt/stop/archive/unarchive/resume;
- emit compact JSON for automation.

## External Agent Boundary

OMC must not vendor or require Hermes.

Hermes may be one external controller/backend, but OMC should be equally capable of connecting to OpenClaw, Claude Code, Codex, or another CLI-compatible agent.

### OMC owns

- T3 Code-derived UI/runtime;
- thread/session/project records;
- adapter registry/config;
- lifecycle controls;
- UI reflection of agent activity;
- OMC-specific spawn tools.

### External agents own

- their own installs;
- their own credentials/config;
- their own memory systems;
- their own tool/plugin ecosystems;
- their own remote gateways, if any.

## MVP Scope

### In scope

1. Fork/derive from `pingdotgg/t3code` safely.
2. Preserve T3 Code UI behavior while adding agent-native spawning hooks.
3. Define the adapter contract for external CLI agents.
4. Implement read-only project/thread inspection.
5. Implement spawn/create of an agent-backed thread.
6. Implement message routing into an existing agent-backed thread.
7. Implement lifecycle controls: interrupt, stop, archive, unarchive, resume where supported.
8. Reflect agent activity in the existing T3 Code UI.
9. Keep all runtime installs/config for Hermes/OpenClaw/Claude Code/etc. external to OMC.

### Out of scope

- vendoring Hermes;
- requiring Hermes as the OMC master;
- in-app Master Agent;
- Admin Agent role;
- role-based permissions;
- Master/Admin UI;
- hidden `OMC Control` project;
- direct Telegram-to-OMC control in MVP;
- custom memory system replacing external agents' memory;
- global CLI installs without explicit opt-in;
- dashboard/registry UI before thread-native UX works.

## T3 Code Dual-Install Collision Check

Status: risky unless isolated. João already has `/Applications/T3 Code (Alpha).app` running on `127.0.0.1:3773` with default app identity `com.t3tools.t3code`, user data under `~/Library/Application Support/t3code`, and T3 data under `~/.t3`. Upstream `pingdotgg/t3code` uses the same product name, bundle id, default port `3773`, default `~/.t3` home, and `t3` CLI package/bin names.

Before running or installing T3 Code for OMC:

- do not overwrite `/Applications/T3 Code (Alpha).app`;
- set an OMC-specific `T3CODE_HOME`, e.g. `/Users/joaofelipe/Desktop/OMC/.omc-t3-home`;
- set a non-default port, not `3773`;
- avoid global `t3` CLI install/link;
- unset inherited `T3CODE_PROJECT_ROOT` unless intentional;
- if building an OMC desktop app, rename product/app id/userData/updater identifiers before install.

## Current Build Order

1. Confirm fork/derivative strategy for `pingdotgg/t3code`.
2. Clone/import the T3 Code base without colliding with the existing T3 Code Alpha install.
3. Rename/isolate OMC app identity, data dir, port, and package/bin names.
4. Verify install/dev/build/Electron packaging in isolation.
5. Map existing T3 project/thread/session internals.
6. Design the external CLI-agent adapter contract.
7. Build read-only project/thread inspection.
8. Add agent-backed thread spawn/create.
9. Add message routing and lifecycle controls.
10. Reflect spawned agent activity in the existing T3 Code UI.
11. Test at least two external backends if available, e.g. Hermes and Claude Code/OpenClaw/Codex.

## Canonical Docs

- `README.md` — short project entrypoint and execution orientation.
- `SystemOutline.md` — detailed architecture, command surface, build plan, and superseded concepts.

Historical planning notes are under `docs/planning/` and are not current source of truth.

## Next Action

Execute Phase 0 from `/Users/joaofelipe/Desktop/OMC`:

- choose fork/import strategy for `pingdotgg/t3code`;
- isolate app identity and runtime paths before running anything;
- verify dev/build/Electron packaging;
- document exact commands and blockers before implementing the agent adapter/spawn surface.
