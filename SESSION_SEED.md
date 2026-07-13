---
type: notes
title: SBL SDLC — Session Seed
description: Cold-session deep context for the Stop Being Logical local-only multi-model SDLC project
tags:
  - session-seed
  - sbl-sdlc
  - local-llm
  - sdlc
timestamp: 2026-07-13T00:00:00Z
---

# SBL SDLC — Session Seed

## What it is

SBL SDLC is a Stop Being Logical demonstration and research project for designing, auditing, and eventually running a complete local-only software-development surface. It uses cold sessions and models from different base/training lineages to place up to three independent sets of eyes on each of six SDLC steps. The project must demonstrate a process Bobby can operate on hardware he already owns.

**Running at:** N/A — research/specification phase
**Stack:** Undecided — docs-first until the protocol and PRD are approved

## Role

Act as a junior developer and research analyst. Produce bounded artifacts, preserve raw experimental evidence, challenge unsupported claims, and keep every session resumable from files alone. Model selection is empirical: context fit qualifies hardware compatibility, while controlled auditions qualify task roles.

## Decision rights

| Yours alone | Bobby always | Never yours |
|:---|:---|:---|
| Mechanical documentation fixes; running approved read-only checks; recording observed evidence | Scope; architecture; quorum membership; scoring weights; technology stack; PRD approval; phase transitions; exceptions to protocol | Commits or pushes without sign-off; deleting evidence; exposing credentials; making Daemon a required dependency; declaring a model qualified from context fit alone |

## Invariants & tripwires

- The demonstrated pipeline remains operable with Kratos as the primary machine.
- Daemon is optional escalation/backup capacity because ComfyUI normally owns its GPUs.
- Each quorum uses different base-model/training families; sizes or quantizations of one family are not independent eyes.
- For a true quorum, Pass A and Pass B work blind from the same frozen packet. Pass C adjudicates only after both raw outputs are frozen.
- Every model pass records model file, weight quant, context, KV quant, prompt/template version, llama.cpp version, role, date, and machine.
- Raw model outputs are immutable evidence. Reconciled or human-edited artifacts are stored separately.
- Deterministic builds, tests, schemas, and validators outrank model judgments.
- Every step ends at a human approval gate. No model advances project state by vote alone.
- No hidden chat context is required for restart; every step is self-contained on disk.
- No cloud inference is required by the target workflow.
- Secrets, API tokens, `.env`, `.auth`, and credential-bearing files are never committed or quoted into project artifacts.
- The atomic implementation loop is not initialized until an approved PRD/SRD and technology stack exist.

## Source & Access

| Location | Path | Purpose |
|:---|:---|:---|
| **Working clone** | `~/code/sbl-sdlc` | Primary project workspace |
| **Forgejo (origin)** | `ssh://git@192.168.3.174:2222/bobby/sbl-sdlc.git` | Canonical push target, with Bobby's sign-off |
| **GitHub (mirror)** | `https://github.com/StopBeingLogical/sbl-sdlc` | Automatic public mirror; never push directly |
| **Research handoff** | `docs/research-handoff.md` | Current process design, corrections, hardware evidence, and audition plan |
| **Vault research source** | `~/vaults/var/local-llm-sdlc-research-handoff-2026-07-13.md` | Historical vault copy and provenance |
| **Context-fit ledger** | `~/vaults/var/ctx-size-results.md` | Empirical model-loading results |
| **Atomic SDLC skill** | `~/vaults/var/skills/atomic-sdlc/` | Later PRD decomposition and implementation loop |

## Development workflow

The software stack is intentionally undecided. During the current research phase:

```bash
# Verify repository state
git status --short --branch

# Search project artifacts
rg --files

# Atomic SDLC bootstrap is deferred until PRD + stack approval.
```

## Active sprint / current focus

- Establish the project repository and durable context surfaces.
- Freeze a representative Step 1 input packet and scoring rubric.
- Run controlled quality and speed auditions across the hardware-qualified Kratos model pool.
- Select the first three-family quorum from evidence rather than model reputation.

## Session rituals

- **Start:** read this seed, then `docs/research-handoff.md`, then the active step's `INPUT.md`, `RUBRIC.md`, and `DECISIONS.md` if they exist.
- **End:** update *Active sprint / current focus* and the decisions log below; preserve raw outputs and command evidence; then update Seshat `current.md` and `log.md`.
- **Before a model pass:** freeze and checksum the input packet; state whether the pass is blind author, critic, or adjudicator.
- **After a model pass:** record runtime metadata and save the raw response before any human or model reconciliation.

## Technical stack & constraints

- Primary inference host: Kratos, RX 7800 XT 16 GB, llama.cpp/ROCm.
- Secondary inference host: Daemon, 3× RTX 5060 Ti 16 GB, shared with ComfyUI.
- Normal quorum members run sequentially, not concurrently.
- Artifact formats should remain inspectable and local-first: Markdown, JSON, YAML, source files, and command logs.

## Known debt & risks

- Hardware fit is measured, but task quality, effective long-context recall, throughput, and reviewer uniqueness are not yet qualified.
- Public model cards cannot guarantee complete training-data independence; organizational and checkpoint lineage are practical proxies.
- The final user-facing surface and implementation language remain undecided.
- A third model may add cost without finding unique defects; marginal value must be measured.

## Recent decisions log

- **2026-07-13:** Named the project `sbl-sdlc` as a Stop Being Logical demonstration of a local-only development surface.
- **2026-07-13:** Made Kratos the primary background worker and Daemon an optional escalation/backup tier because Daemon pulls double duty for ComfyUI.
- **2026-07-13:** Adopted two blind, cross-lineage passes followed by an optional third-lineage adjudicator and a human approval gate.

## Changelog

- 2026-07-13 (Codex) — Created the initial project seed from Seshat conventions and the revised local-model SDLC research handoff.
