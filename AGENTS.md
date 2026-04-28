# AGENTS.md - AI Agent Operating Protocol for OMC

> Read [SystemOutline.md](SystemOutline.md) for the full system overview.

## Quick Start
- This repository is currently **documentation-first**. No runnable services are scaffolded yet.
- Start every task by reading:
  1. `README.md`
  2. `AGENTS.md`
  3. `SystemOutline.md`
  4. historical notes under `docs/planning/` only if needed for context
- Basic repo commands:
  - `git status --short --branch`
  - `find . -maxdepth 3 -type f | sort`
  - `rg -n "pattern" .`
- Canonical local workflow right now:
  1. inspect repo state
  2. inspect relevant docs/code
  3. make the smallest coherent change
  4. verify links / doc consistency / file placement
  5. report what changed and what remains
- There are currently no canonical app start/stop commands. When implementation code lands, update this section immediately.

## Agent Execution Protocol
- Source of truth order:
  1. runtime code
  2. configuration
  3. persistent docs
  4. planning docs
- Mandatory preflight before changes:
  - run `git status`
  - search for existing files related to the target area
  - read the nearest call chain or document chain before editing
  - identify whether the change is structural, behavioral, or purely documentary
- Required work sequence:
  1. **Contextualize** — understand the relevant files and current behavior
  2. **Plan** — define the intended change and affected files
  3. **Implement** — make the smallest clean change that solves the task
  4. **Verify** — validate formatting, links, and consistency with adjacent docs/code
  5. **Report** — summarize changed files, behavior, and any follow-up risk
- Long-running process policy:
  - detach long-running jobs
  - redirect output to a log file
  - return PID, log path, and stop instructions
  - never block the session on a persistent foreground process unless explicitly asked
- Definition of done by change type:
  - **Docs-only change**: links valid, file placement correct, no stale references remain
  - **API change**: contracts updated, endpoint docs updated, affected READMEs updated, verification captured
  - **Frontend change**: UI behavior documented, affected screenshots/flows described if needed, relevant README/SystemOutline updated
  - **Database change**: migration path documented, schema references updated, service ownership clarified
  - **Deploy/ops change**: operator steps updated in canonical docs and any runbook references reconciled
- Never declare work complete if the relevant documentation layer was left stale.

## Working Conventions
- Follow existing naming and layout patterns before introducing new ones.
- Prefer shared utilities and shared documentation over duplicate explanations.
- Keep secrets out of the repo; use environment variables or external secret stores.
- Separate **implemented reality** from **planned design**. Do not present roadmap items as shipped behavior.
- Planning docs may describe future architecture; implementation docs must clearly state current state.
- Keep root-level clutter low. New non-root docs should usually live under `docs/` unless they are required top-level control docs.
- Use Markdown links with relative paths.
- Keep tables concise and durable; avoid content that will become stale after trivial code changes.
- When behavior changes structurally, update both the nearest README and `SystemOutline.md`.
- See component READMEs for detailed structure once components exist.

## Documentation Maintenance Rules
### README Lifecycle
- Every component directory must have a `README.md`.
- A component directory means a service, library, module root, or script collection with meaningful behavior.
- Agents must create missing component READMEs when introducing a new component.
- Agents must update component READMEs when changing:
  - entry points
  - key files
  - API behavior
  - configuration
  - runtime usage
  - known constraints
- Component README template sections:
  - Purpose
  - Architecture / How It Works
  - Key Files
  - API Endpoints (if applicable)
  - Configuration
  - Running / Usage
  - Known Constraints

### Plan Doc Lifecycle
- Plan docs are temporary scratchpads for work that is not fully shipped.
- Plan docs may live under `docs/planning/` while the feature is in design or active execution.
- Once a planned feature ships, its durable knowledge must be absorbed into:
  - the relevant component README(s)
  - `SystemOutline.md`
  - root `README.md` if onboarding-facing
- After that absorption, shipped plan docs should be deleted.
- A component README must always coexist with any plan doc affecting that component.

### Self-Maintenance
- Agents may autonomously maintain:
  - component READMEs
  - `SystemOutline.md`
  - root `README.md`
  - `docs/README.md`
  - task handoff and gotcha sections in this file
- Agents must not change the operating rules in `AGENTS.md` without user approval.
- Agents may append/remove from **Task Handoff Board** and **Critical Gotchas** when needed.
- When repo structure materially changes, agents should update the directory tree in `SystemOutline.md`.

## Parallel Agent Coordination Protocol
- Each parallel agent should work in its **own worktree** whenever code changes are involved.
- Declare file scope at task start whenever multiple agents are active.
- Avoid overlapping write scopes unless the user explicitly accepts merge coordination risk.
- If parallel work stays conflict-free, self-merge and verify before reporting.
- If conflicts exist, stop and present the conflict set clearly to the user or coordinator.
- Coordinator responsibilities:
  - assign ownership boundaries
  - track status of each subtask
  - handle merge order
  - resolve or escalate conflicts
- For docs-only parallel tasks, partition by file or directory, not by overlapping topic.

## Commit Guidelines
- Follow any existing project convention if one is later introduced.
- Until then, use:
  - short imperative subject line
  - optional body for rationale
  - optional co-author tag when appropriate
- Keep commits scoped to one coherent change.
- Documentation bootstrap commits should separate repo setup from later implementation changes.

## Task Handoff Board

| Status | Item | Relevant READMEs |
|---|---|---|
| In Progress | Phase 0 foundation: verify repo state, then clone/fork and validate T3 Code substrate decisions | `README.md`, `SystemOutline.md` |
| Remaining | Scaffold the first implementation components and create per-component READMEs as soon as code exists | `SystemOutline.md` |
| Remaining | Keep historical planning docs superseded unless useful facts are merged into canonical docs | `docs/planning/*.md` |

## Critical Gotchas
- This repo currently has **no implementation components**; do not assume services exist.
- `docs/planning/` contains roadmap/design material; it is not proof that behavior is shipped.
- Canonical architecture is OMC as a T3 Code-derived UI/runtime with agent-native spawn tools and external CLI-agent adapters; Hermes is only one possible external backend, not vendored or required.
- `docs/planning/` is historical/superseded unless canonical docs explicitly revive a detail.
- When real code lands, update `SystemOutline.md` before or alongside structural changes.
- Keep top-level docs limited to onboarding/control docs; place detailed design docs under `docs/`.
