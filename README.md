# SBL SDLC

SBL SDLC is the Stop Being Logical demonstration and research project for running a complete software-development deliberation pipeline on local hardware and open-weight models.

The core experiment uses cold model sessions and independent training lineages. Each SDLC step receives two blind passes, an optional third-model adjudication pass, deterministic validation where possible, and a human approval gate before the next step begins.

## Operating topology

- **Kratos is primary:** routine author, critic, implementation, and adjudication jobs run sequentially on its RX 7800 XT 16 GB.
- **Daemon is secondary:** its RTX 5060 Ti cards normally serve ComfyUI and are used only for backup, escalation, or controlled heavyweight comparisons.
- **Local-only is the target:** the demonstrated pipeline must remain operable without cloud inference.

## Current phase

The project is in research and protocol-design. Model context fit has been measured; controlled task-quality auditions and the first frozen Step 1 packet come next. No implementation stack has been selected and no PRD has been approved yet.

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

- 2026-07-13 (Codex) — Initialized the docs-first project surface and linked the research handoff and session seed.
