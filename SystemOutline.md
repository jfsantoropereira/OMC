# OMC System Outline

Status: canonical system/design outline.

Repository: https://github.com/jfsantoropereira/OMC.git
Local path: `/Users/joaofelipe/Desktop/OMC`

## 1. Accepted Architecture

OMC is T3 Code plus agent-native spawn/control tools.

Accepted decisions:

- OMC is the UI/runtime host, not the master agent.
- OMC derives from T3 Code and should preserve its existing project/thread/chat UX.
- External CLI agents connect through adapters: Hermes, OpenClaw, Claude Code, Codex, and future compatible backends.
- Hermes is not installed, cloned, vendored, or bundled as part of OMC.
- Spawned agent work should appear in the UI using normal T3 Code primitives: projects, threads, turns, sessions, events, lifecycle status, archives, and worktrees.
- The OMC-native spawn/control surface is the product addition. Working name: `ospawn`; older docs may say `hatch`.
- No in-app Master Agent for MVP.
- No Admin Agent role for MVP.
- No role hierarchy for MVP.
- No major UI redesign before thread-native spawning works.

## 2. Design Principles

- T3-native UX first.
- External-agent neutral: no privileged Hermes dependency.
- CLI adapter boundary over vendored runtimes.
- Threads over roles.
- Reflect agent work in existing UI instead of inventing a separate dashboard.
- Local-first, explicit configuration.
- Isolate app identity, ports, and data dirs from existing T3 Code installs.
- Compact JSON command output for automation.
- Add auth, policy, hierarchy, and dashboards only after real usage pressure.

## 3. System Components

### 3.1 OMC/T3 Host

OMC owns the application/UI/runtime shell.

Responsibilities:

- projects;
- threads;
- chat UI;
- provider/session/event records;
- worktrees/bootstrap;
- archive/unarchive;
- interrupt/stop/resume surfaces;
- adapter registry/config;
- UI reflection of external agent activity.

Non-responsibilities:

- not a bundled Hermes install;
- not a single-agent orchestrator;
- not a master/admin hierarchy;
- not Telegram-first UX in MVP.

### 3.2 External CLI Agents

External agents are separately installed and configured tools.

Examples:

- Hermes;
- OpenClaw;
- Claude Code;
- Codex;
- other local or remote CLI-compatible agents.

Responsibilities of external agents:

- their own installation and updates;
- their own credentials and provider config;
- their own memory/tools/plugins;
- their own execution semantics;
- optional remote gateways outside OMC.

OMC should call them, supervise them, and display their work. It should not absorb their internals.

### 3.3 Agent Adapter Layer

The adapter layer normalizes external CLI agents into OMC thread operations.

Responsibilities:

- define per-agent launch commands;
- set explicit cwd/env/stdin/stdout/stderr contracts;
- map prompts/messages into backend-specific CLI invocations;
- stream output/events into OMC turns/events;
- map stop/interrupt/resume when supported;
- capture artifacts, logs, summaries, and exit status;
- keep backend-specific state behind the adapter boundary.

Adapter contract should include:

- adapter id/name;
- command template;
- required binaries/env vars;
- supported capabilities;
- working directory strategy;
- input protocol;
- output parsing/streaming protocol;
- interrupt/stop behavior;
- resume behavior;
- artifact/log locations;
- health check.

### 3.4 ospawn / hatch

`ospawn` is the preferred product name for OMC-native spawn/control tooling.

`hatch` may remain as a historical/internal alias, but docs should trend toward `ospawn` if that is the chosen name.

Responsibilities:

- expose project/thread operations;
- create agent-backed threads;
- message existing threads;
- configure adapter/model/runtime/worktree;
- lifecycle control;
- emit compact JSON for automation;
- stay close to T3 project/thread vocabulary.

Non-responsibilities:

- not a Hermes-specific control plane;
- not a policy engine in MVP;
- not a role hierarchy;
- not a separate agent runtime.

## 4. Existing T3 Capabilities to Reuse

Verify during Phase 0/1:

- project/thread orchestration;
- thread creation;
- turn/session lifecycle;
- archive/unarchive;
- interrupt/stop;
- thread metadata updates;
- worktree/bootstrap support;
- event streams or logs;
- archive UI;
- compaction/activity/status signals;
- desktop packaging and app identity configuration.

These are substrate facts. Do not revive old Master/Admin recommendations.

## 5. Minimal OMC Additions

### Required

- OMC app identity isolation from T3 Code Alpha.
- External CLI-agent adapter registry/config.
- `ospawn` command/API surface.
- Read-only project/thread inspection.
- Agent-backed thread create/spawn.
- Message routing to existing agent-backed thread.
- Lifecycle controls.
- UI reflection of adapter/session status.
- Resume/artifact capture or link.

### Optional neutral metadata

Only add if it is cheap and directly useful:

- `adapterId`;
- `externalSessionId`;
- `agentCommand` or command template id;
- `parentThreadId`;
- `rootThreadId`;
- `spawnPurpose`;
- `createdBy: user | cli | adapter:<id> | thread:<id>`.

### Explicitly not required

- Hermes source tree;
- Hermes memory layer;
- Hermes gateway/Telegram code;
- `agentKind: master/admin/regular`;
- `adminThreadIds`;
- pinned project master;
- role-based spawn permissions;
- grant/revoke admin flows.

## 6. ospawn Command Surface

Canonical namespace should use normal T3 concepts and external-agent adapters.

### 6.1 Context

```bash
ospawn context --json
ospawn adapters list --json
ospawn adapters show <adapter-id> --json
```

### 6.2 Projects

```bash
ospawn projects list --json
ospawn projects show <project-id> --json
```

### 6.3 Threads: Read

```bash
ospawn threads list --project <project-id> --state <state> --json
ospawn threads show <thread-id> --json
```

### 6.4 Threads: Spawn / Message / Configure

```bash
ospawn threads spawn   --project <project-id>   --adapter hermes|openclaw|claude-code|codex   --title "..."   --prompt-file task.md   --workdir <path>   --json

ospawn threads message <thread-id> --text "..." --json
ospawn threads message <thread-id> --prompt-file followup.md --json

ospawn threads configure <thread-id>   --adapter <adapter-id>   --model <provider/model>   --worktree <strategy>   --json
```

### 6.5 Threads: Lifecycle

```bash
ospawn threads interrupt <thread-id> --json
ospawn threads stop <thread-id> --json
ospawn threads archive <thread-id> --reason "done" --json
ospawn threads unarchive <thread-id> --json
ospawn threads resume <thread-id> --json
```

### 6.6 JSON Envelope

Target shape:

```json
{
  "ok": true,
  "actor": "omc-local",
  "result": {
    "threadId": "thr_123",
    "projectId": "proj_abc",
    "adapterId": "claude-code",
    "state": "running"
  }
}
```

On failure:

```json
{
  "ok": false,
  "error": {
    "code": "adapter_not_available",
    "message": "Adapter claude-code is configured but the CLI binary was not found"
  }
}
```

## 7. UI Reflection Model

OMC should make external CLI-agent work visible using existing T3 UI patterns.

MVP UI behavior:

- spawned agent = thread/session, not special role;
- agent output = turns/messages/events;
- tool/command output = event/log blocks where T3 supports it;
- running/stopped/interrupted/archived = existing lifecycle state where possible;
- artifacts/resume summaries = linked from the thread;
- adapter identity = lightweight metadata/badge only if needed.

Avoid:

- Master tab;
- Admin sidebar section;
- separate org-chart dashboard;
- role-based thread sorting;
- UI that implies Hermes is the built-in master.

## 8. External Agent Boundary

OMC must treat Hermes, OpenClaw, Claude Code, Codex, etc. as external dependencies.

Rules:

- Do not clone or vendor Hermes into the OMC repo.
- Do not require Hermes for OMC to run.
- Do not assume Hermes memory/notes/gateway are available.
- Do not store external-agent secrets in OMC.
- Keep adapter configs explicit and local/user-controlled.
- If an adapter requires a binary, detect it and fail with a clear setup error.

## 9. T3 Code Dual-Install Collision Check

Status: risky unless isolated. João already has `/Applications/T3 Code (Alpha).app` running on `127.0.0.1:3773` with default app identity `com.t3tools.t3code`, user data under `~/Library/Application Support/t3code`, and T3 data under `~/.t3`. Upstream `pingdotgg/t3code` uses the same product name, bundle id, default port `3773`, default `~/.t3` home, and `t3` CLI package/bin names.

Before running or installing T3 Code for OMC:

- do not overwrite `/Applications/T3 Code (Alpha).app`;
- set an OMC-specific `T3CODE_HOME`, e.g. `/Users/joaofelipe/Desktop/OMC/.omc-t3-home`;
- set a non-default port, not `3773`;
- avoid global `t3` CLI install/link;
- unset inherited `T3CODE_PROJECT_ROOT` unless intentional;
- rename product/app id/userData/updater identifiers before installing an OMC desktop build.

## 10. MVP Build Plan

### Phase 0 — Foundation and Isolation

- choose fork/import strategy for `pingdotgg/t3code`;
- isolate OMC identity from T3 Code Alpha;
- verify install/dev/build/Electron packaging in isolation;
- document exact commands and blockers.

Exit criteria:

- OMC can be developed/run without touching the existing T3 Code Alpha app/data/port;
- repo layout is clear;
- no external agent backend has been vendored into OMC.

### Phase 1 — T3 Thread/UI Substrate Reconnaissance

- map project/thread/session/event APIs;
- map UI components that display turns/status/artifacts;
- map lifecycle controls;
- identify clean extension points for adapter-backed sessions.

### Phase 2 — Adapter Contract

- define adapter config schema;
- define command launch contract;
- define input/output streaming contract;
- define capability detection;
- define error model.

### Phase 3 — ospawn Read-Only Skeleton

- context;
- adapters list/show;
- projects list/show;
- threads list/show.

### Phase 4 — Agent-Backed Thread Spawn

- spawn thread using selected adapter;
- route initial prompt;
- persist adapter/session metadata;
- reflect status in UI.

### Phase 5 — Message and Lifecycle

- message existing agent-backed thread;
- interrupt;
- stop;
- archive;
- unarchive;
- resume where supported.

### Phase 6 — Multi-Backend Validation

Test at least two adapters if available, such as:

- Claude Code;
- Codex;
- Hermes;
- OpenClaw.

The point is backend neutrality. OMC should not accidentally become a Hermes-only shell.

### Phase 7 — Optional UI Polish

Only after thread-native spawning works:

- adapter badges;
- artifact links;
- lightweight status indicators;
- parent/child visibility if useful.

## 11. Removed / Superseded Concepts

These are not part of the canonical MVP:

- Hermes bundled into OMC;
- Hermes as required OMC runtime;
- Master Agent inside T3;
- hidden `OMC Control` project;
- Master Agent tab/sidebar/dashboard;
- Admin Agents;
- Master/Admin/Regular hierarchy;
- role-based spawn authorization;
- grant/revoke admin flows;
- project-master promotion/demotion;
- pinned master thread;
- Admin badges/sorting/styling;
- UI-first control surfaces;
- heartbeat scheduler/message queue/wakeup system as MVP scope;
- direct Telegram-to-OMC bridge;
- WhatsApp integration;
- custom vector DB;
- complex remote/multi-tenant auth before local MVP works.

Historical notes may mention these concepts, but canonical planning should not depend on them.

## 12. Open Questions

- Exact fork/import strategy for `pingdotgg/t3code`.
- Final name: `ospawn`, `hatch`, or another CLI/API name.
- Adapter protocol details for each backend.
- Whether adapters are configured through files, UI settings, or both.
- Minimum event stream format needed to reflect external agent output in T3 UI.
- Minimum resume/artifact implementation.

## 13. Immediate Next Action

Execute Phase 0.

Do not start Hermes integration, Master/Admin concepts, or dashboard work. First make OMC a safely isolated T3 Code derivative with a clear external-agent adapter boundary.
