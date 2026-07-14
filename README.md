# SBL SDLC

SBL SDLC is the Stop Being Logical demonstration and research project for running a complete software-development deliberation pipeline on local hardware and open-weight models.

The core experiment uses cold model sessions and a cumulative amendment ladder of independent training lineages. Each SDLC step is built weak→strong: a skeleton clears a human frame-gate, then successive models each amend the artifact as they see fit, with a teardown pass on framing-heavy steps, deterministic validation where possible, and a human approval gate before the next step begins.

## Operating topology

- **Kratos is primary:** every step's ladder rungs — skeleton, fill, teardown, implementation — run sequentially on its RX 7800 XT 16 GB.
- **Daemon is secondary:** its RTX 5060 Ti cards normally serve ComfyUI and are used only for backup, escalation, or controlled heavyweight comparisons.
- **Local-only is the target:** the demonstrated pipeline must remain operable without cloud inference.

## Current phase

The project is in research and protocol-design. Model context fit has been measured (an Enki_v2 input); standing up the ladder scaffolding and the first frozen Step 1 packet come next. No implementation stack has been selected and no PRD has been approved yet.

## Start here

1. Read [SESSION_SEED.md](SESSION_SEED.md) for project invariants and decision rights.
2. Read [docs/research-handoff.md](docs/research-handoff.md) for the original ideation, review corrections, current six-step design, hardware results, and audition plan.
3. Do not initialize the atomic implementation loop until a PRD/SRD and technology stack have been approved.

## Repository topology

- Forgejo origin: `ssh://git@192.168.3.174:2222/bobby/sbl-sdlc.git`
- GitHub mirror: `https://github.com/StopBeingLogical/sbl-sdlc`
- Working directory: `~/code/sbl-sdlc`

GitHub is a silent mirror. Never push to it directly.

## Changelog

- 2026-07-13 (Opus 4.8) — Updated the summary and current-phase to the cumulative amendment ladder and scoped model auditions to Enki_v2.
- 2026-07-13 (Codex) — Initialized the docs-first project surface and linked the research handoff and session seed.
