Status: superseded by canonical OMC docs.

Canonical docs:

- [../../README.md](../../README.md)
- [../../SystemOutline.md](../../SystemOutline.md)

This file is historical context only. Do not treat it as current architecture or build plan. Current architecture: OMC is a T3 Code-derived UI/runtime with agent-native spawn tools; external CLI agents such as Hermes, OpenClaw, Claude Code, and Codex connect through adapters; Hermes is not vendored or required; no in-app Master Agent or Admin Agent role for MVP.

---

# Hatch CLI — OMC Control Plan

## Purpose
`hatch` is the control-plane CLI that lets the **Master Agent**, **Admin Agents**, and **Regular Agents** interact with the OMC orchestration layer.

It is not just a thread utility. It is the policy-aware command surface for:
- agent discovery
- agent-to-agent messaging
- spawn control
- admin assignment
- lifecycle control
- auditability

## CLI Design Principles
- **System-wide binary on PATH** — available to all agents and processes, like the `notes` CLI.
- **Token-efficient output** — terse, structured output. No verbose banners, decorations, or redundant labels. Every byte of output costs tokens in the invoking agent's context.
- **`-help` on every command** — self-documenting so agents can discover usage without external docs.
- **Compact formats** — prefer single-line-per-entity, tab-separated or minimal whitespace. Pagination and filtering flags where output could grow large.
- **Exit codes** — `0` success, `1` error, `2` permission denied. Stderr for errors, stdout for data.

## Core Roles

### Master Agent
- global control plane
- can act across all projects
- can spawn agents anywhere
- can assign or revoke Admin status in any project
- can inspect, interrupt, archive, unarchive, and message any agent

### Admin Agent
- normal project agent with elevated privileges inside one project
- there may be multiple Admin Agents per project
- Admin status is a designation, not a permanent subtype
- can spawn subagents inside its own project scope only
- can manage non-admin agents in that project
- can archive non-admin agents in that project
- cannot archive fellow Admin Agents
- cannot grant itself broader global power

### Regular Agent
- default role for normal agent threads
- may converse, inspect, report, self-archive, and request help
- may not spawn subagents
- may not assign/revoke Admin status
- may not perform cross-project control actions

### Worker Agent
- short-lived agent spawned for bounded execution
- usually created by the Master Agent or an Admin Agent
- typically regular by default unless explicitly elevated later by Master

## Key Policy Rules

### Spawn authority
Only these roles may spawn:
- Master Agent — any project
- Admin Agent — only its own project

Regular Agents cannot spawn.

### Admin authority
- a project may have zero, one, or many Admin Agents
- Admin designation belongs to a thread while that thread is active and authorized
- if an Admin Agent is archived, deleted, or loses project membership, it must be removed from the project admin set
- unarchiving does not automatically restore Admin status unless explicitly designed later

## Recommended Data Model

### Thread-level fields
- `agentKind: master | regular | worker`
- `parentThreadId`
- `rootThreadId`
- `ownerThreadId`
- `spawnPurpose`
- `spawnedByThreadId`
- `archivedReason`

Admin should **not** be stored as a permanent thread kind.

### Project-level fields
- `adminThreadIds: ThreadId[]`

## Recommended Command Groups
- `hatch whoami`
- `hatch context`
- `hatch agents list`
- `hatch agents tree`
- `hatch agents show <threadId>`
- `hatch message send`
- `hatch request spawn`
- `hatch spawn create --model <provider/version>` (e.g. `--model claude/latest`, `--model codex/gpt-5.4-high`)
- `hatch spawn continue`
- `hatch admin list`
- `hatch admin grant`
- `hatch admin revoke`
- `hatch message send <threadId> [--interrupt] "message"` — queue message; `--interrupt` bypasses queue and forces immediate delivery
- `hatch interrupt <threadId>`
- `hatch archive <threadId>`
- `hatch unarchive <threadId>`
- `hatch stop <threadId>`
- `hatch self status`
- `hatch self archive`
- `hatch self handoff`
- `hatch self request-admin-help`
- `hatch self request-spawn`

## Message Queue Behavior
- every thread has an **inbox queue** for incoming messages from other agents
- if target thread is idle: message delivered immediately (injected as a user-turn, same as T3 native messaging)
- if target thread is busy (active provider session): message queued
- **`--interrupt` flag**: bypasses queue, interrupts current generation, delivers immediately
- **auto-drain**: queued messages auto-drain when flowing **downstream** in the hierarchy (Master→Admin, Admin→Worker)
- **upstream notifications** (subagent completion → parent): also auto-drain, since this is the wakeup mechanism

## Notification Signal Format
When a subagent completes a turn, its parent receives a **signal-only** notification:
```
[subagent:<threadId> completed]
```
Parent uses `hatch agents show <threadId>` to pull details if needed. This keeps notification token cost minimal.

## Final MVP Policy Recommendation
- only Master and Admin can spawn
- only Master can grant/revoke Admin
- Admin can archive only non-admin agents in its own project
- Regular agents can request, message, inspect, and self-manage, but not expand hierarchy directly
- archiving an Admin automatically removes it from the project admin set
- unarchive does not restore Admin automatically
- notes CLI write access: Master and Admin only (convention-enforced via system prompt, not CLI-level restriction)
