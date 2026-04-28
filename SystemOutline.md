# OMC System Outline

Status: canonical system/design outline. Supersedes the prior OMC planning notes unless a note is explicitly kept as historical research.

Repository: https://github.com/jfsantoropereira/OMC.git
Local path: `/Users/joaofelipe/Desktop/OMC`

## 1. Accepted Architecture

OMC is a thin T3 Code thread substrate controlled by Hermes through `hatch`.

Accepted decisions:

- Hermes is the external master/operator.
- OMC/T3 provides persistent projects, threads, sessions, events, lifecycle state, and visible history.
- `hatch` is the programmatic control surface Hermes uses to inspect and operate T3/OMC.
- No in-app Master Agent for MVP.
- No Admin Agent role for MVP.
- No Master/Admin/Regular role hierarchy for MVP.
- No meaningful UI work before the CLI substrate works.

## 2. Design Principles

- T3-native semantics first.
- Threads over roles.
- CLI/API before UI.
- Local-first trusted Hermes operator.
- Minimal metadata.
- Compact JSON output for machine control.
- Add auth, policy, UI, and hierarchy only if real usage creates pressure.
- Do not duplicate Hermes inside OMC.

## 3. System Components

### 3.1 Hermes

Hermes owns high-level operation.

Responsibilities:

- user/operator interface;
- global reasoning and planning;
- memory and notes access;
- Telegram/remote routing;
- choosing direct answer vs native `delegate_task` vs persistent OMC thread;
- calling `hatch` to operate OMC/T3.

Non-responsibilities:

- Hermes should not be embedded inside T3 as a special Master thread.
- Hermes should not require OMC to implement an agent org chart.

### 3.2 OMC/T3

OMC/T3 owns persistent execution substrate.

Responsibilities:

- projects;
- threads;
- provider sessions;
- worktrees/bootstrap;
- event/state model;
- visible message history;
- archive/unarchive;
- interrupt/stop;
- resume/handoff artifacts where possible.

Non-responsibilities:

- not a master/admin hierarchy;
- not the final orchestrator;
- not Telegram-first UX;
- not a new custom agent runtime if T3 primitives already suffice.

### 3.3 hatch

`hatch` is the narrow programmatic interface between Hermes and OMC/T3.

Responsibilities:

- expose project/thread operations;
- produce stable compact JSON;
- let Hermes inspect and mutate OMC/T3 state;
- stay close to T3 vocabulary.

Non-responsibilities:

- not a policy engine in MVP;
- not a role/permission framework in MVP;
- not a separate agent runtime.

## 4. Existing T3 Capabilities to Reuse

From prior T3 research, preserve these as likely useful substrate facts to verify during Phase 0/1:

- event-sourced project/thread orchestration;
- `thread.create`;
- `thread.archive` and `thread.unarchive`;
- `thread.turn.start`;
- `thread.turn.interrupt`;
- `thread.session.stop`;
- `thread.meta.update`;
- bootstrap/worktree/setup support;
- remote access foundations;
- archive UI;
- compaction/activity signals.

These are substrate facts, not a mandate to preserve old Master/Admin recommendations.

## 5. Minimal OMC Additions

### Required

- `hatch` CLI.
- Read-only project/thread inspection.
- Thread create/message/configure.
- Lifecycle commands.
- Resume artifact command or hook.

### Optional neutral metadata

Only add if it is cheap and directly useful:

- `parentThreadId`;
- `rootThreadId`;
- `spawnPurpose`;
- `createdBy: hermes | user | thread:<id>`.

### Explicitly not required

- `agentKind`;
- `adminThreadIds`;
- `ownerThreadId` for role hierarchy;
- pinned project master;
- role-based spawn permissions;
- grant/revoke admin flows.

## 6. hatch Command Surface

Canonical namespace: `threads`.

Optional compatibility alias: `spawn`, but docs and implementation should prefer `threads` because this is programmatic T3 usage, not an agent hierarchy.

### 6.1 Context

```bash
hatch context --json
hatch whoami --json
```

### 6.2 Projects

```bash
hatch projects list --json
hatch projects show <project-id> --json
```

### 6.3 Threads: Read

```bash
hatch threads list --project <project-id> --state <state> --json
hatch threads show <thread-id> --json
hatch threads tree --project <project-id> --json
```

`tree` is optional. Do not implement hierarchy UI before basic thread operations work.

### 6.4 Threads: Create / Message / Configure

```bash
hatch threads create   --project <project-id>   --title "..."   --prompt-file task.md   --model <provider/model>   --json

hatch threads message <thread-id> --text "..." --json
hatch threads message <thread-id> --prompt-file followup.md --json

hatch threads configure <thread-id>   --model <provider/model>   --worktree <strategy>   --json
```

### 6.5 Threads: Lifecycle

```bash
hatch threads interrupt <thread-id> --json
hatch threads stop <thread-id> --json
hatch threads archive <thread-id> --reason "done" --json
hatch threads unarchive <thread-id> --json
hatch threads resume <thread-id> --json
```

### 6.6 JSON Envelope

Target shape:

```json
{
  "ok": true,
  "actor": "hermes-local",
  "result": {
    "threadId": "thr_123",
    "projectId": "proj_abc",
    "state": "running"
  }
}
```

On failure:

```json
{
  "ok": false,
  "error": {
    "code": "thread_not_found",
    "message": "No thread found for id thr_123"
  }
}
```

## 7. Archive and Resume

A resume artifact should let Hermes or a human restart context without scraping an entire thread.

Suggested `thread_resume.md` fields:

- goal;
- current status;
- relevant decisions;
- important messages or links;
- files changed/artifacts created;
- tests/commands run;
- blockers;
- next actions.

Hermes decides what to persist to notes.

## 8. UI Stance

MVP:

- no meaningful UI changes;
- no Master tab;
- no Master sidebar row;
- no dashboard/registry;
- no Admin UI;
- no role/permission UI;
- preserve normal T3 project/thread UX.

Possible later, only after CLI proves useful:

- parent/child thread display;
- created-by-Hermes indicator;
- resume artifact link;
- lightweight status indicators.

## 9. Telegram Stance

Telegram routes to Hermes, not directly to OMC/T3.

Hermes decides whether to:

- answer directly;
- use native `delegate_task`;
- operate a persistent OMC/T3 thread via `hatch`.

OMC/T3 does not need a direct Telegram bridge for MVP.

## 10. Delegation Substrate Split

### Hermes `delegate_task`

Use for:

- ephemeral clean-context subagents;
- independent review;
- codebase inspection;
- synthesis;
- tasks where a final summary is sufficient.

Properties:

- isolated context;
- no persistent T3-visible thread;
- fast to spawn;
- good for review and bounded implementation.

### OMC/`hatch` threads

Use for:

- persistent T3-visible work;
- project/worktree tasks;
- sessions that need archive/resume;
- long-running or follow-up-heavy work;
- cases where lifecycle control matters.

Properties:

- visible in T3;
- persistent history;
- explicit lifecycle;
- operated by Hermes through `hatch`.

## 11. MVP Build Plan

### Phase 0 — Foundation

- choose local repo path;
- fork/clone `pingdotgg/t3code`;
- install dependencies;
- verify dev server;
- verify production build;
- verify Electron packaging if supported;
- document repo layout and exact commands.

Exit criteria:

- T3 Code runs locally;
- build/package status is known;
- blockers are documented;
- no OMC abstraction work starts before this is clear.

### Phase 1 — T3 Thread Substrate Reconnaissance

- map existing project/thread APIs and event names;
- identify where CLI can call into T3 safely;
- verify archive/unarchive, interrupt, stop, meta update, worktree/bootstrap behavior;
- decide whether `hatch` should call a local HTTP API, internal package, local DB, or existing CLI/server entrypoint.

### Phase 2 — hatch Read-Only Skeleton

Implement:

- `hatch context --json`;
- `hatch whoami --json`;
- `hatch projects list/show --json`;
- `hatch threads list/show --json`;
- optional `hatch threads tree --json`.

### Phase 3 — Thread Create / Message / Configure

Implement:

- `hatch threads create`;
- `hatch threads message`;
- `hatch threads configure`;
- prompt-file support;
- model/runtime/worktree flags where T3 supports them cleanly.

### Phase 4 — Lifecycle

Implement:

- interrupt;
- stop;
- archive;
- unarchive.

### Phase 5 — Resume Artifacts

Implement one of:

- explicit `hatch threads resume <thread-id> --json`;
- archive-triggered `thread_resume.md` generation;
- fetch existing T3 summaries/metadata if available.

### Phase 6 — Hermes/Telegram Routing

- route Telegram into Hermes;
- Hermes chooses direct answer, `delegate_task`, or `hatch` operation;
- keep OMC/T3 unaware of Telegram unless later pressure proves otherwise.

### Phase 7 — Optional UI Polish

Only after the CLI substrate is useful:

- small metadata/status display;
- parent/child thread visibility;
- resume artifact links.

## 12. Removed / Superseded Concepts

These are not part of the canonical MVP:

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

## 13. Open Questions

Keep these tactical:

- Exact local repo path: `/Users/joaofelipe/Desktop/OMC` vs `/Users/joaofelipe/Desktop/OMC-T3`.
- Where should `hatch` live: inside the T3 repo, adjacent package, or external wrapper?
- Should `hatch` call a local HTTP API, an internal package, local DB, or existing T3 internals?
- What is the minimum resume artifact implementation?
- Is neutral parent/root metadata needed in v0, or can it wait?

## 14. Immediate Next Action

Execute Phase 0.

Do not start Admin/Master/UI/Telegram work before the T3 foundation and `hatch` thread substrate are proven.
