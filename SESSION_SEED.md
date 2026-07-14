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

SBL SDLC is a Stop Being Logical demonstration and research project for designing, auditing, and eventually running a complete local-only software-development surface. It uses cold sessions and a cumulative amendment ladder of models from different base/training lineages — ordered weak→strong, each amending the prior artifact as it sees fit — to layer several sets of eyes on each of six SDLC steps and cancel the weaknesses of any single local model. The project must demonstrate a process Bobby can operate on hardware he already owns.

**Running at:** N/A — research/specification phase
**Stack:** Undecided — docs-first until the protocol and PRD are approved

## Role

Act as a junior developer and research analyst. Produce bounded artifacts, preserve raw experimental evidence, challenge unsupported claims, and keep every session resumable from files alone. Model selection is empirical and supplied by the separate Enki_v2 project: context fit qualifies hardware compatibility, while Enki's auditions rank task capability. This project consumes that ranking to order each step's ladder; it does not run the auditions.

## Decision rights

| Yours alone | Bobby always | Never yours |
|:---|:---|:---|
| Mechanical documentation fixes; running approved read-only checks; recording observed evidence | Scope; architecture; ladder/rung membership; scoring weights; technology stack; PRD approval; phase transitions; exceptions to protocol | Commits or pushes without sign-off; deleting evidence; exposing credentials; making Daemon a required dependency; declaring a model qualified from context fit alone |

## Invariants & tripwires

- The demonstrated pipeline remains operable with Kratos as the primary machine.
- Daemon is optional escalation/backup capacity because ComfyUI normally owns its GPUs.
- Each step's ladder uses different base-model/training families across its rungs; sizes or quantizations of one family are not independent eyes.
- Each step is built by a cumulative amendment ladder (weak→strong, cross-lineage), not blind independent passes: the Rung 0 skeleton clears a human frame-gate before any rung fills it, and each fill rung records a frame-challenge and a KEEP/CHANGE/KILL ledger. A non-editing teardown rung runs on the framing-heavy steps (1–4) and as the Step 6 release audit.
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
| **Research handoff** | `docs/research-handoff.md` | Current ladder protocol, review history, hardware evidence, and Enki_v2 inputs |
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

- Establish the project repository and durable context surfaces. (done)
- Stand up the amendment-ladder step scaffolding: skeleton + frame-gate, fill rungs with frame-challenge and KEEP/CHANGE/KILL ledgers, teardown rung, and edit-delta capture.
- Freeze a representative Step 1 input packet and scoring rubric; frame-gate the rubric before freezing.
- Consume the Step 1 model ladder (weak→strong, lineage-tagged) from Enki_v2 and define the Enki_v2 → SBL SDLC handoff interface.

## Session rituals

- **Start:** read this seed, then `docs/research-handoff.md`, then the active step's `INPUT.md`, `RUBRIC.md`, and `DECISIONS.md` if they exist.
- **End:** update *Active sprint / current focus* and the decisions log below; preserve raw outputs and command evidence; then update Seshat `current.md` and `log.md`.
- **Before a model pass:** freeze and checksum the input packet; state the rung's role (skeleton, fill rung N, or teardown) and the model's capability rank and lineage.
- **After a model pass:** record runtime metadata and save the raw response before any human or model reconciliation.

## Technical stack & constraints

- Primary inference host: Kratos, RX 7800 XT 16 GB, llama.cpp/ROCm.
- Secondary inference host: Daemon, 3× RTX 5060 Ti 16 GB, shared with ComfyUI.
- Normal ladder rungs run sequentially, not concurrently.
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
- **2026-07-13 (supersedes the above):** Replaced blind passes + adjudicator with a cumulative amendment ladder (weak→strong, cross-lineage) gated by a Rung 0 skeleton frame-gate, per the Opus 4.8 review merged into `docs/research-handoff.md`. Model capability ranking and auditions are scoped to Enki_v2, which feeds this project.

## Changelog

- 2026-07-13 (Opus 4.8) — Reconciled the seed to the cumulative amendment ladder: rewrote the quorum/blind-pass invariants and the "what it is" / role framing, scoped model auditions to Enki_v2, updated the active sprint and the before-a-pass ritual, and logged the protocol supersession in the decisions log.
- 2026-07-13 (Codex) — Created the initial project seed from Seshat conventions and the revised local-model SDLC research handoff.
