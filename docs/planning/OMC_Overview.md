# OMC Overview

## Goal
Build OMC as an agent orchestration system on top of T3 Code concepts, with:
- a **Master Agent** above all other agents
- **Admin Agents** inside projects, with zero-to-many admins per project
- **Worker Agents** spawned for bounded tasks
- **remote operation** starting with Telegram
- **durable memory** using notes-backed and embedding-backed workflows

## Working Assumptions
- Base implementation substrate is expected to derive from `pingdotgg/t3code`
- Initial remote channel is Telegram only
- WhatsApp comes later, after orchestration is stable
- Project memory can use `notes` as the first persistence interface
- Master memory should be global and higher priority than per-thread memory

## Architecture Thesis
T3 Code already provides strong primitives for:
- projects
- threads
- thread archive / unarchive
- thread interrupt / stop
- remote environment access over WebSocket
- event-sourced orchestration with SQLite projections
- provider sessions and resume cursors

OMC should therefore extend that orchestration model, not replace it wholesale.

## Proposed Control Model
1. **Master Agent**
   - global control plane
   - can create, message, interrupt, archive, and inspect downstream agents
   - can operate across projects
2. **Admin Agents**
   - normal project agents with elevated authority inside a project
   - there can be multiple Admin Agents per project
   - Admin is a designation, not a permanent subtype
   - only Master and Admin Agents may spawn subagents
3. **Regular Agents**
   - default non-admin project agents
   - can converse, inspect, and self-manage within policy
   - cannot spawn subagents directly
4. **Worker Agents**
   - short-lived execution threads spawned for bounded tasks
   - usually subordinate to another agent

## Important Design Clarification
Admin should not be modeled as a permanent thread kind.
It should be a project-level designation applied to normal agent threads.

That implies:
- thread hierarchy metadata
- a project-level `adminThreadIds` set or equivalent
- commands to grant/revoke Admin designation
- archive behavior where Admins may archive only non-admin agents in their own project
- archive behavior that clears Admin status automatically

## MVP Scope
### In scope
- Master agent UX + orchestration primitives
- Hatch / Sub-Agent CLI
- Admin designation and admin management
- Telegram ingress/egress
- archive hook that produces `thread_resume.md`
- notes-backed project memory adapter

### Out of scope for v0
- WhatsApp integration
- sophisticated multi-tenant auth
- fully custom vector database
- deep autonomous planning beyond thread + task orchestration
