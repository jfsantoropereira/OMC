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
- `hatch spawn create`
- `hatch spawn continue`
- `hatch admin list`
- `hatch admin grant`
- `hatch admin revoke`
- `hatch interrupt <threadId>`
- `hatch archive <threadId>`
- `hatch unarchive <threadId>`
- `hatch stop <threadId>`
- `hatch self status`
- `hatch self archive`
- `hatch self handoff`
- `hatch self request-admin-help`
- `hatch self request-spawn`

## Final MVP Policy Recommendation
- only Master and Admin can spawn
- only Master can grant/revoke Admin
- Admin can archive only non-admin agents in its own project
- Regular agents can request, message, inspect, and self-manage, but not expand hierarchy directly
- archiving an Admin automatically removes it from the project admin set
- unarchive does not restore Admin automatically
