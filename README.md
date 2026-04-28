# OMC — One Man Company

Status: canonical entrypoint. Supersedes the scattered OMC planning notes.

Repository: https://github.com/jfsantoropereira/OMC.git
Local path: `/Users/joaofelipe/Desktop/OMC`


## Status

Accepted architecture:

- OMC is a thin T3 Code thread substrate controlled by Hermes through `hatch`.
- Hermes is the external master/operator interface.
- OMC does not implement an in-app Master Agent.
- OMC does not implement Admin Agent roles for MVP.
- OMC should stay close to normal T3 Code usage: projects, threads, messages, configuration, lifecycle, archive/resume.
- CLI/API first. UI changes only after the thread substrate is proven useful.

## Goal

Build OMC as programmable T3 Code usage for Hermes.

The useful primitive is not an agent hierarchy. The useful primitive is reliable control over persistent T3-backed work threads:

- create threads;
- message existing threads;
- configure model/runtime/worktree;
- list/show project and thread state;
- interrupt/stop/archive/unarchive threads;
- produce or fetch resume artifacts.

## Core Thesis

Do not build an agent org chart before the thread-control substrate exists.

T3 Code already has much of the substrate: projects, threads, turns, sessions, archive state, interruption, and worktree/bootstrap foundations. OMC should expose those primitives cleanly to Hermes instead of inventing Master/Admin roles inside the app.

## Control Model

### Hermes

Hermes is the master/operator layer.

Responsibilities:

- decide whether to answer directly, use native `delegate_task`, or operate a persistent OMC/T3 thread through `hatch`;
- hold global memory/context through tools and notes;
- route Telegram or other remote interfaces into the operator loop;
- call `hatch` as the local control surface for OMC/T3.

Hermes should not be reimplemented inside T3 as a special Master Agent thread.

### OMC/T3 Threads

OMC threads are normal T3 Code threads used as persistent agent/session units.

Responsibilities:

- retain visible history;
- run inside project/worktree context;
- be interruptible, stoppable, archivable, resumable;
- stay inspectable through T3 UI and `hatch`.

MVP threads do not need special Master/Admin/Regular roles.

### hatch CLI

`hatch` is the programmatic T3 operator surface used by Hermes.

It should expose T3-like operations:

- `hatch projects list/show`
- `hatch threads list/show/tree`
- `hatch threads create`
- `hatch threads message`
- `hatch threads configure`
- `hatch threads interrupt`
- `hatch threads stop`
- `hatch threads archive/unarchive`
- `hatch threads resume`

All operational commands should support compact `--json` output.

## Delegation Substrate Split

### Use Hermes `delegate_task` for

- one-shot research;
- codebase inspection;
- independent review/synthesis;
- clean-context ephemeral work;
- tasks where a final summary is enough.

### Use OMC/`hatch` threads for

- persistent T3-visible sessions;
- long-running coding/project/worktree tasks;
- tasks needing resume, archive, interrupt, or follow-up;
- workflows where visible lifecycle state matters.

These are complementary substrates, not replacements for each other.

## MVP Scope

### In scope

1. Fork/clone and verify `pingdotgg/t3code` locally.
2. Verify install/dev/build/Electron packaging.
3. Map existing T3 project/thread commands, events, and APIs.
4. Implement `hatch` read-only inspection.
5. Implement thread create/message/configure.
6. Implement lifecycle controls: interrupt, stop, archive, unarchive.
7. Implement or fetch resume artifacts.
8. Route Telegram through Hermes later, not directly into OMC/T3.

### Out of scope

- in-app Master Agent;
- Admin Agent role;
- role-based permissions;
- Master/Admin UI;
- hidden `OMC Control` project;
- direct Telegram-to-OMC control;
- WhatsApp integration;
- custom vector database;
- complex remote/multi-tenant auth;
- dashboard/registry UI before the CLI works.


## Repository Setup

```bash
cd /Users/joaofelipe/Desktop/OMC
git remote -v
git status --short --branch
```

Remote:

```text
origin  https://github.com/jfsantoropereira/OMC.git
```

This repo is currently documentation/bootstrap only. No T3 Code source or runnable OMC services are committed yet.

## Current Build Order

1. Choose local repo path.
2. Fork/clone `pingdotgg/t3code`.
3. Install dependencies.
4. Verify dev/build/Electron packaging.
5. Document exact commands and repo layout.
6. Map T3 thread/project internals.
7. Build `hatch` read-only commands.
8. Add create/message/configure.
9. Add lifecycle commands.
10. Add resume artifacts.
11. Test side-by-side against Hermes `delegate_task`.

## Canonical Docs

- `README.md` — short project entrypoint and execution orientation.
- `SystemOutline.md` — detailed architecture, command surface, build plan, and superseded concepts.

Historical planning notes are under `docs/planning/` and are not current source of truth.

## Next Action

Execute Phase 0 from `/Users/joaofelipe/Desktop/OMC`:

- decide whether `pingdotgg/t3code` should be forked into this repo, vendored as a subtree/submodule, or cloned as an adjacent upstream reference;
- install dependencies for the chosen T3 Code working copy;
- verify dev server/build/Electron packaging;
- document exact commands and blockers before starting `hatch` implementation.
