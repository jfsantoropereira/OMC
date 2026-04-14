# OMC

One Man Company: a planned agent orchestration system built on top of T3 Code concepts, centered on a global Master Agent, project-scoped Admin Agents, and controlled worker spawning.

## Core Services

| Service | Entry Point | Dev Port | Docker Port |
|---|---|---:|---:|
| _None scaffolded yet_ | — | — | — |

## Quick Start

```bash
git clone https://github.com/jfsantoropereira/OMC.git
cd OMC
git status --short --branch
```

Then read, in order:
1. [AGENTS.md](AGENTS.md)
2. [SystemOutline.md](SystemOutline.md)
3. [docs/README.md](docs/README.md)
4. the relevant planning docs under [`docs/planning/`](docs/planning/)

## Documentation

### Top-level Docs

| Document | Path | Purpose |
|---|---|---|
| Agent operating protocol | [AGENTS.md](AGENTS.md) | Rules and workflow for agents working in this repo |
| System reference | [SystemOutline.md](SystemOutline.md) | Canonical system-wide technical documentation |
| Docs index | [docs/README.md](docs/README.md) | Index of planning and supporting docs |
| OMC overview | [docs/planning/OMC_Overview.md](docs/planning/OMC_Overview.md) | Product concept and control model |
| Hatch CLI plan | [docs/planning/Hatch_CLI.md](docs/planning/Hatch_CLI.md) | CLI design and authorization plan |
| MVP build plan | [docs/planning/MVP_Build_Plan.md](docs/planning/MVP_Build_Plan.md) | Sequenced implementation plan |
| UI changes plan | [docs/planning/UI_Changes_Plan.md](docs/planning/UI_Changes_Plan.md) | Master/Admin sidebar and UI planning |
| T3Code gap analysis | [docs/planning/T3Code_Gap_Analysis.md](docs/planning/T3Code_Gap_Analysis.md) | Upstream research and required extension points |

### Component READMEs

| Component | README |
|---|---|
| _None yet_ | No implementation components have been scaffolded yet. |

## Repository Status

This repository is currently in a **documentation-first bootstrap state**:
- git repository initialized
- planning docs materialized into the repo
- no application code committed yet
- no deployment or runtime configuration committed yet

Implementation work should begin by scaffolding the first real components, then creating per-component READMEs as those components appear.
