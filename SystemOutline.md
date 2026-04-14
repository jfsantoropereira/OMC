# OMC System Documentation

> Comprehensive Technical Reference | Last Updated: 2026-04-14
> For AI agent guidance, see [AGENTS.md](AGENTS.md).

## Table of Contents
1. [Component README Index](#component-readme-index)
2. [Executive Summary](#1-executive-summary)
3. [Architecture Overview](#2-architecture-overview)
4. [Database Schema](#3-database-schema)
5. [Services Reference](#4-services-reference)
6. [API Reference](#5-api-reference)
7. [Configuration](#6-configuration)
8. [Operations](#7-operations)
9. [Migration & Deployment](#8-migration--deployment)
10. [Development Guide](#9-development-guide)
11. [Appendices](#appendices)

## Component README Index

| Component | README Path | Description |
|---|---|---|
| _None yet_ | — | No service, library, module, or script components have been scaffolded in this repository yet. |

## 1. Executive Summary

OMC (One Man Company) is a planned agent orchestration system intended to be built on top of the architectural foundation provided by T3 Code. The current repository state is **documentation-first**: it contains planning, operating, and system reference documents, but no implementation services, libraries, migrations, or deployment assets yet.

The core intended product model is:
- a global **Master Agent**
- project-scoped **Admin Agents**
- **Regular Agents** with restricted privileges
- spawned **Worker Agents** for bounded tasks
- long-term memory integration through notes/embedding workflows
- remote control beginning with Telegram

### Current state
- Git repository initialized
- top-level operating and system docs present
- planning material materialized under `docs/planning/`
- no application code committed yet
- no environment variables, schemas, endpoints, or deployment configuration committed yet

### Planned operating model (high level)
```text
User / Telegram / UI
        |
        v
   Master Agent
        |
        +----------------------+----------------------+
        |                      |                      |
        v                      v                      v
   Project A Admins       Project B Admins      Project C Admins
        |                      |                      |
        v                      v                      v
  Regular / Worker agents Regular / Worker agents Regular / Worker agents
```

The canonical current planning sources are:
- [docs/planning/OMC_Overview.md](docs/planning/OMC_Overview.md)
- [docs/planning/Hatch_CLI.md](docs/planning/Hatch_CLI.md)
- [docs/planning/MVP_Build_Plan.md](docs/planning/MVP_Build_Plan.md)
- [docs/planning/UI_Changes_Plan.md](docs/planning/UI_Changes_Plan.md)
- [docs/planning/T3Code_Gap_Analysis.md](docs/planning/T3Code_Gap_Analysis.md)

## 2. Architecture Overview

### High-level system diagram

#### Current repository state
```text
OMC repo
├── AGENTS.md
├── README.md
├── SystemOutline.md
└── docs/
    ├── README.md
    └── planning/
        ├── OMC_Overview.md
        ├── Hatch_CLI.md
        ├── MVP_Build_Plan.md
        ├── UI_Changes_Plan.md
        └── T3Code_Gap_Analysis.md
```

#### Planned runtime architecture
```text
┌──────────────────────────────────────────────┐
│ Client Surfaces                              │
│ - Web UI                                     │
│ - Telegram bridge                            │
│ - Hatch CLI                                  │
└───────────────────────┬──────────────────────┘
                        │
                        v
┌──────────────────────────────────────────────┐
│ OMC Control Plane                            │
│ - Master Agent                               │
│ - authorization / role rules                 │
│ - project admin designations                 │
│ - agent registry                             │
└───────────────────────┬──────────────────────┘
                        │
                        v
┌──────────────────────────────────────────────┐
│ T3-derived orchestration/runtime layer       │
│ - projects / threads / sessions              │
│ - provider orchestration                     │
│ - worktree / git flows                       │
│ - archival / handoff generation              │
└───────────────────────┬──────────────────────┘
                        │
                        v
┌──────────────────────────────────────────────┐
│ Memory + Persistence                         │
│ - orchestration event store                  │
│ - projection state                           │
│ - notes-backed project memory                │
│ - master memory index / embeddings           │
└──────────────────────────────────────────────┘
```

### Services overview table

| Service | Entry Point | Port | Purpose |
|---|---|---:|---|
| _None scaffolded yet_ | — | — | The repository is currently in documentation/planning state only. |

### Technology stack table

| Layer | Current State | Planned Direction |
|---|---|---|
| Repository | Git + Markdown docs | Git monorepo based on or derived from T3 Code |
| UI | None committed yet | React / T3 Code sidebar and chat surfaces |
| Backend | None committed yet | TypeScript / Node or Bun, reusing T3 orchestration patterns |
| Persistence | None committed yet | SQLite event store + projections + notes-backed memory |
| Integrations | None committed yet | Telegram first, WhatsApp later |
| Agent control | Planning only | Hatch CLI + Master/Admin/Regular policy model |

### Directory structure (annotated tree)

```text
.
├── AGENTS.md                     # Agent operating protocol for this repo
├── README.md                     # Root onboarding and doc index
├── SystemOutline.md              # Canonical system reference
└── docs/
    ├── README.md                 # Documentation directory index
    └── planning/
        ├── OMC_Overview.md       # Product and control model summary
        ├── Hatch_CLI.md          # CLI interaction and authorization plan
        ├── MVP_Build_Plan.md     # Implementation sequence and phases
        ├── UI_Changes_Plan.md    # Sidebar/UI behavior plan
        └── T3Code_Gap_Analysis.md# Upstream research and gap analysis
```

## 3. Database Schema

### Current state
No database schema, migrations, ORM models, or SQL files are committed in this repository yet.

### Planned connection architecture
Expected future architecture, based on current planning:
- orchestration event store for projects, threads, and lifecycle events
- projection tables for efficient UI queries
- project-level admin designation state
- memory artifact storage and embedding/index references
- notes-backed memory materialization layer for project and master memory

### Planned logical entities (non-implemented)

| Entity | Planned Purpose |
|---|---|
| Project | Container for project-scoped agents and admin designations |
| Thread / Agent | Unit of conversation, control, and execution |
| Admin designation | Project-level membership indicating elevated privileges |
| Spawn relation | Parent/child link between agents |
| Memory artifact | Handoff, resume, or embedded long-term memory document |
| Archive artifact | Generated `thread_resume.md` and related summaries |

### Tables, indexes, views
Current state: none committed.

When schema work begins, this section becomes the canonical place for:
- table definitions
- column descriptions
- indexes
- views
- relationships
- service ownership of schema surfaces

## 4. Services Reference

No implementation services are committed yet.

When services are added, each subsection in this section must include:
- location
- entry point
- purpose
- short workflow summary
- a cross-reference in the form:

> Future service sections should begin with a relative link to that component README; for example, a service section may open with a sentence pointing readers to `apps/web/README.md`.

### Planned service categories
These are roadmap categories only, not implemented services:
- control plane / orchestration extensions
- Hatch CLI surface
- UI extensions for Master/Admin visibility
- Telegram bridge
- memory ingestion / archive summarization pipeline

## 5. API Reference

### Current state
No API endpoints are committed in this repository yet.

### Planned API/interaction surfaces
The primary planned control interfaces are:
- Hatch CLI (see [docs/planning/Hatch_CLI.md](docs/planning/Hatch_CLI.md))
- Web UI extensions (see [docs/planning/UI_Changes_Plan.md](docs/planning/UI_Changes_Plan.md))
- remote message ingress through Telegram

### Response models
None implemented yet.

When code lands, this section should hold:
- full endpoint tables per service
- method and path
- request and response model summaries
- auth requirements
- cross-links to relevant component READMEs

## 6. Configuration

### Current state
No `.env`, `.env.example`, deployment config, or CI/CD pipeline files are committed yet.

### Environment variables
Current state: none documented because none are implemented.

### Planned configuration categories
These are expected future buckets only:
- runtime environment selection
- provider/runtime auth
- Telegram bot credentials
- persistence paths / database settings
- notes/memory configuration
- observability / logging

### Database users and permissions
None implemented yet.

## 7. Operations

### How to run each service
Current state: no runnable services exist in the repo.

### Troubleshooting guide

| Symptom | Likely Cause | Current Resolution |
|---|---|---|
| Nothing to run | Repo is documentation-only so far | Read planning docs and scaffold implementation before expecting runtime commands |
| Missing component README | Component has not been created yet, or docs were not updated | Create/update the component README alongside the component |
| Planning doc conflicts with implementation | Plan became stale | Treat code as source of truth and update or delete the plan doc |

### Common issues
- confusing roadmap docs with shipped behavior
- forgetting to update `SystemOutline.md` after structural changes
- leaving planning docs in place after features ship without absorbing them into durable docs

## 8. Migration & Deployment

### Current deployment state
No deployment artifacts or workflows are committed yet.

### Target deployment direction
Planned future state includes:
- local development environment derived from T3 Code foundations
- possibly desktop/web UI plus remote control surfaces
- Telegram as the first remote control ingress
- staged progression from docs → local implementation → remote operation

### Deployment references
There is currently no `DEPLOY.md` in the repository.
If deployment docs are added later, link them here and from `README.md`.

## 9. Development Guide

### Local setup
At present, setup is documentation-oriented:
1. clone the repository
2. read `README.md`
3. read `AGENTS.md`
4. read `SystemOutline.md`
5. inspect the planning docs under `docs/planning/`
6. scaffold the first code components before adding runtime expectations

### Code conventions
Current conventions available from planning:
- separate current implementation from future design
- prefer extending T3 orchestration patterns rather than replacing them
- model Admin as a project-level designation, not a permanent subtype
- preserve role policy boundaries in both UI and CLI design

### Documentation conventions
- root docs remain lean and onboarding-focused
- detailed design docs live under `docs/planning/` until implemented
- durable technical knowledge moves into component READMEs and this file once code ships

## Appendices

### Appendix A — Planning document reference card

| Document | Purpose |
|---|---|
| [docs/planning/OMC_Overview.md](docs/planning/OMC_Overview.md) | Product concept and role model |
| [docs/planning/Hatch_CLI.md](docs/planning/Hatch_CLI.md) | CLI behavior and authorization model |
| [docs/planning/MVP_Build_Plan.md](docs/planning/MVP_Build_Plan.md) | Build sequencing and MVP phases |
| [docs/planning/UI_Changes_Plan.md](docs/planning/UI_Changes_Plan.md) | UI behavior and sidebar changes |
| [docs/planning/T3Code_Gap_Analysis.md](docs/planning/T3Code_Gap_Analysis.md) | Upstream research and extension points |

### Appendix B — Upstream reference
The current planning assumes T3 Code as the upstream implementation substrate. That dependency is architectural and documentary at this stage only; no upstream source has been committed into this repository yet.
