---
type: notes
title: Local-Only Multi-Model SDLC Research — Ideation, Review, and Current Design
description: Historical proposal, GPT 5.6 review, empirical corrections, and current six-step cold-session workflow for local open-weight models
timestamp: 2026-07-14T00:55:45Z
updated: 2026-07-13
status: working-design
tags:
  - notes
  - llm
  - sdlc
  - hardware
  - handoff
  - local-only
  - multi-model-review
---

# Local-Only Multi-Model SDLC Research

> **Project canon:** This copy now lives with the implementation project at `~/code/sbl-sdlc`. The vault copy preserves the research-session provenance; future project-specific evolution should land here and be summarized back to the vault/Seshat indexes.

## Purpose and reading order

This document records three distinct layers of the project:

1. **Original ideation** — the proposal produced after review by Gemini and Opus.
2. **GPT 5.6 review and corrections** — structural and empirical corrections made after the original sessions collapsed.
3. **Final version as of now** — the current coherent workflow to be reviewed again by Gemini and Opus.

The goal is not to make one model autonomously complete an entire software project. The goal is to determine how a rigorous software-development deliberation process can be run entirely on already-owned local hardware and open-weight models.

For each of six SDLC steps, the operator begins from a cold session and builds the artifact through a cumulative amendment ladder: a first model produces a skeleton that clears a human frame-gate, then successive models — ordered from least to most capable and drawn from different training lineages — each take the prior artifact and amend it as they see fit, with a non-editing teardown pass on the framing-heavy steps. This layers several sets of eyes onto every step, cancelling the weaknesses of any single local model while avoiding dependence on accumulated conversational context. (Parts I–II below preserve the earlier blind-pass/adjudication design this ladder replaced.)

No cloud inference is required for the intended production workflow. External frontier models may review this research design, but they are not part of the local execution path being tested.

---

# Part I — Original Ideation

## Original objective

The original research sought replacements for older dense baselines such as `Qwen 2.5 14B / 32B` and `Llama 3.3 70B` for a six-phase local software-development lifecycle.

The proposal was informed by the Seshat bootstrap canon (`bootstrap.md`, `hardware.md`), the `prd-to-atomic-tasks.md` runbook, and `prd-decomposer v2.5.1`, including its `TASKS.md` and `PROGRESS.json` invariants.

Two hardware constraints shaped the design:

1. **Daemon contention:** Daemon has 3× RTX 5060 Ti 16 GB GPUs, but those cards are also used for ComfyUI. Every SDLC step therefore needs a viable single-card fallback on Kratos, which has one RX 7800 XT 16 GB.
2. **Two-card orchestration:** MoE models such as Gemma 4 26B A4B and Qwen 3.6 35B-A3B can load with long context on two Daemon cards, allowing the third card to remain available for ComfyUI.

This was the original topology. The final design below reverses the priority: Kratos is the normal background worker, and Daemon is an optional escalation or backup tier.

## Original six-step mapping

### Step 1 — Rubber duck from square one

Explore the problem, question assumptions, and use local tool calling or a local search service where appropriate.

- Two-card candidates: Gemma 4 26B A4B; Qwen 3.6 35B-A3B
- Single-card candidates: Gemma 4 12B; Qwen 3.5 9B

### Step 2 — Roadmap and overall vision

Turn discovery into a strategic project direction and implementation roadmap.

- Two-card candidates: Gemma 4 31B Dense; Qwen 3.6 35B-A3B
- Single-card candidates: Phi-4 14B; Gemma 4 12B

### Step 3 — Prepare PRD / SRD

Produce the functional specification, system requirements, interfaces, and data model.

- Two-card candidate: Qwen 3.6 35B-A3B
- Single-card candidates: Gemma 4 12B; DeepSeek Coder V2 Lite 16B

### Step 4 — Decompose the PRD into atomic tasks

Apply the `prd-decomposer v2.5.1` rules and generate valid `TASKS.md` and `PROGRESS.json` artifacts.

Original invariants:

- Each build task touches no more than two files.
- Each task targets approximately 50–150 lines of code.
- Signatures, parameter types, return values, and error structures are specified explicitly.
- No placeholder `TODO` implementations are accepted.
- Batches contain 2–4 tasks and terminate in an automated `TEST` gate.

Candidates:

- Two-card: Qwen 3.6 35B-A3B; Gemma 4 26B A4B
- Single-card: Qwen 3.5 9B; DeepSeek Coder V2 Lite 16B

### Step 5 — Review or judge atomic-task outputs

Review the implementation outputs against the predefined interfaces and task contracts.

- Two-card candidate: Qwen 3.6 35B-A3B
- Single-card candidates: DeepSeek Coder V2 Lite 16B; Gemma 4 12B

### Step 6 — Recompose the final deliverable

Integrate accepted outputs into a coherent final project and supporting documentation.

- Two-card candidates: Qwen 3.6 35B-A3B; Gemma 4 31B Dense
- Single-card candidates: Qwen 3.5 9B; Gemma 4 12B

## Original hardware hypotheses

The initial model-to-hardware table was based partly on architecture and VRAM estimates. It treated Qwen 3.5 9B, Gemma 4 12B, Phi-4 14B, and DeepSeek Coder V2 Lite as single-card candidates, and Gemma 4 26B A4B, Qwen 3.6 35B-A3B, and Gemma 4 31B as two-card candidates.

The proposal also used qualitative labels such as “A-tier” and “S-tier.” Those labels represented informed candidate ranking, not completed task-quality benchmarks.

---

# Part II — GPT 5.6 Review and Corrections

## 1. Correct interpretation of the workflow

The original wording could be read as a single autonomous pipeline. The intended method is better described as a **cold-session, multi-lineage deliberation protocol**:

- One model produces an independent draft for the current step.
- A model from a different family or training lineage receives the same approved input packet plus the first output and performs a critical review or independent alternative.
- An optional third model adjudicates disagreements, checks omissions, or produces the reconciled candidate.
- The human operator approves the artifact that becomes the immutable input to the next step.

This distinction matters. The value comes from independent perspectives and explicit artifacts, not from preserving one long conversation.

## 2. Missing artifact contracts

Each step originally described an activity and candidate models but did not consistently define:

- Required inputs
- Required outputs
- Review rubric
- Completion gate
- Restart packet for a cold session
- Human approval point

Without those contracts, different models may solve subtly different problems and their outputs cannot be compared fairly.

## 3. Implementation ambiguity

The original sequence moved from atomic decomposition to review of atomic-task outputs without explicitly stating where implementation occurs.

The current correction keeps six top-level steps but defines **Step 5 as execution plus independent review**. Atomic tasks are implemented in batches, deterministic checks are run, and another model reviews the resulting changes. Step 6 then performs project-level integration and release validation.

## 4. Deterministic evidence must outrank model judgment

A model can assess architecture, interfaces, requirements, and likely defects, but it cannot declare an implementation correct merely by reading it. Pass/fail decisions should incorporate:

- Builds
- Unit and integration tests
- Type checking
- Linting and formatting
- Schema validation
- Security checks where applicable
- Clean-checkout reproduction
- Requirement-to-test traceability

Model review supplements this evidence; it does not replace it.

## 5. Author and reviewer independence

Where practical, the model that authored an artifact should not be its only reviewer. Family diversity is preferable to merely changing model size or quantization.

Examples of meaningfully different eyes in the current pool include Qwen-family, Gemma-family, DeepSeek-family, Phi-family, GPT-OSS, Kimi, Granite, Falcon, GLM, and LFM models. Two quantizations of the same base model do not count as independent lineages.

## 6. Atomic-task limits need an exception mechanism

The two-file and 50–150-line limits are useful defaults because they constrain review scope. They should not be absolute correctness conditions. Schema migrations, generated artifacts, interface changes, fixtures, and tightly coupled refactors may require exceptions.

Any exception should be declared in the task with a reason, expanded test scope, and explicit human approval.

## 7. Context fit is not task qualification

The tests performed so far establish whether a model loads with a requested context and KV-cache quantization. They do not establish:

- Effective recall near the end of that context
- Tool-call correctness
- Search discipline or citation quality
- PRD completeness
- Decomposition correctness
- Code quality
- Review sensitivity
- Prompt-processing or generation speed

The original “tier” labels are therefore retained only as historical hypotheses. Current selection must be based on controlled auditions using identical task packets and scoring rubrics.

## 8. Empirical hardware corrections

Observed `llama-server` load results supersede the initial VRAM estimates:

| Machine | Model | Highest effective context observed | KV quantization | Cards | Correction |
| :--- | :--- | ---: | :---: | ---: | :--- |
| Kratos | Qwen3.5-9B-UD-Q4_K_XL | 262,144 | Q4_0 / Q4_0 | 1 | Confirmed long-context single-card candidate |
| Kratos | Gemma-4-12B-it-UD-Q4_K_XL | 131,072 | Q8_0 / Q8_0 | 1 | Model capped requests above 131,072 |
| Kratos | Phi-4-Q4_K_M | 16,384 | Q4_0 / Q4_0 | 1 | Requests above training context were capped; 131K Q8 failed |
| Kratos | DeepSeek-Coder-V2-Lite-Instruct-Q4_K_M | 65,536 | Q4_0 / Q4_0 | 1 | Q8 KV failed at 65,536; larger Q4 attempts failed |
| Kratos | GPT-OSS-20B-Q4_K_M | 131,072 | Q4_0 / Q4_0 and Q8_0 / Q8_0 | 1 | Confirmed alternative lineage |
| Kratos | Kimi-VL-A3B-Instruct-Q4_K_M | 131,072 | Q8_0 / Q8_0 | 1 | Confirmed alternative lineage |
| Kratos | Granite-3.3-8B-Instruct-Q4_K_M | 131,072 | Q4_0 / Q4_0 | 1 | Q8 KV failed from compute-buffer OOM; Q4 KV loaded successfully |
| Kratos | Falcon-H1-7B-Instruct-Q4_K_M | 262,144 | Q8_0 / Q8_0 | 1 | Confirmed at full model training context |
| Daemon | GLM-4.7-Flash-UD-Q4_K_XL | 202,752 | Q8_0 / Q8_0 | 2 | Capped at model training context |
| Daemon | LFM2-24B-A2B-Q4_K_M | 128,000 | Q8_0 / Q8_0 | 2 | Capped at model training context |
| Daemon | Qwen3-Coder-30B-A3B-Instruct-UD-Q4_K_XL | 262,144 | Q8_0 / Q8_0 | 2 | Classified as a two-card result per the operator's test summary |
| Daemon | Qwen3.6-27B-UD-Q4_K_XL | 262,144 | Q8_0 / Q8_0 | 2 | Confirmed |
| Daemon | Qwen3.6-35B-A3B-UD-Q4_K_XL | 262,144 | Q8_0 / Q8_0 | 2 | Confirmed |
| Daemon | Gemma-4-26B-A4B-it-UD-Q4_K_XL | 262,144 | Q8_0 / Q8_0 | 2 | Confirmed |
| Daemon | Gemma-4-31B-it-UD-Q4_K_XL | 262,144 | Q8_0 / Q8_0 | 3 | Two-card attempt failed; not a dependable two-card primary at this setting |

Operator note: all recorded Daemon models except Gemma 4 31B were able to load on two cards. Gemma 4 31B required the third card.

## 9. Corrected model assessment

- **Qwen 3.6 35B-A3B:** strongest currently confirmed two-card orchestration candidate by capacity; task quality remains to be auditioned.
- **Gemma 4 26B A4B:** primary cross-family reviewer and competing orchestrator; confirmed at 262K on two cards.
- **Qwen 3.5 9B:** strongest confirmed long-context single-card fallback by capacity.
- **Gemma 4 12B:** useful single-card alternative lineage, with an effective 131K ceiling in the recorded test.
- **DeepSeek Coder V2 Lite:** useful code-specialist lineage, but currently limited to 65K Q4 KV on Kratos.
- **Phi-4:** short-context reasoning specialist, not a general long-context fallback.
- **GPT-OSS 20B:** valuable additional single-card lineage at 131K and should be included in auditions.
- **Kimi-VL A3B:** additional single-card lineage, especially relevant if visual inputs become part of discovery or review.
- **Granite 3.3 8B:** confirmed at 131K with Q4 KV on Kratos; a strong independent IBM-lineage candidate for structured requirements, decomposition, and review auditions.
- **Falcon-H1 7B:** confirmed at its full 262K context with Q8 KV on Kratos; the strongest newly added hardware-qualified candidate for long-context, architecturally diverse review.
- **Gemma 4 31B:** optional three-card editorial or synthesis model; conflicts with preserving a Daemon card for ComfyUI.

---

# Part III — Final Version as of 2026-07-13

## Operating principles

1. Every step begins in a cold model session.
2. Every session receives a versioned, self-contained input packet.
3. Each step is built by a cumulative amendment ladder rather than by independent voting: successive passes each take the prior artifact and alter it as they see fit, ordered from least to most capable, drawn from different training lineages so their weaknesses do not overlap.
4. Framing is gated before filling: an opening pass produces only a skeleton, which clears a human frame-gate before any rung writes the body, so a weak early frame cannot anchor the entire step.
5. Models build and amend; deterministic checks and the human operator approve state transitions.
6. Deterministic tests and validators have authority over prose claims of correctness.
7. No hidden conversational state is required to resume the project.
8. Local inference is the target operating mode; local search, repositories, documentation mirrors, and test tools may be provided as explicit tools.
9. Kratos is the primary always-available background worker for the pipeline.
10. Daemon is a secondary escalation and backup machine because its GPUs normally serve ComfyUI image and video generation.
11. No normal step may require Daemon in order for the workflow to continue; Daemon models are optional higher-capacity eyes.

## Final operating topology

### Primary tier — Kratos

Kratos runs each step's ladder rungs sequentially on its single RX 7800 XT 16 GB. A multi-rung ladder does not imply multiple models loaded concurrently. Each cold pass loads one model, writes its artifacts, unloads, and yields to the next rung.

The Kratos pool therefore determines whether the local-only workflow is operational. Long-running discovery, review, decomposition, implementation, and validation jobs should default to this machine.

### Secondary tier — Daemon

Daemon is used when:

- ComfyUI is idle or sufficient cards are available
- A step exceeds the useful context or capability of the Kratos pool
- A higher-capacity tie-breaker is warranted
- Kratos is unavailable
- A controlled comparison against the larger MoE models is being conducted

Daemon capacity should be treated as opportunistic. Two-card models should not silently become required dependencies, and Gemma 4 31B should remain a special three-card experiment.

## Standard amendment-ladder protocol for every step

Each step is built by a cumulative ladder of model passes. A pass does not merely critique — it takes the artifact it is handed and amends it as it sees fit. The ladder is ordered from least to most capable so cheap breadth is added first and the strongest model holds the final quality bar, and every rung is drawn from a different training lineage so overlapping blind spots do not survive. Capability ranking and lineage assignment are inputs supplied by Enki_v2 (see "Model inputs from Enki_v2"), not decided here.

The ordering creates one hazard — the *albatross*: because models anchor to text already on the page, a weak early frame tends to be decorated by later rungs rather than rebuilt. The protocol contains that hazard by separating **framing** from **filling** and by forcing reconsideration at every rung.

### Rung 0 — Skeleton and frame-gate

The opening pass produces only a skeleton: structure, section headers, the decomposition or interface/requirement list, an assumption register, and open questions — no prose body. Framing is the highest-leverage and least-reversible decision in the step, so it is not left to the weakest model and it is not laddered. The skeleton is set by a capable rung and then clears a **human frame-gate** — a short operator review whose only job is to accept, reject, or restructure the frame before any rung fills it. A bad frame dies here for the cost of one cheap pass instead of being carried up the whole ladder.

### Fill rungs — cumulative amendment, weak→strong, mixed lineage

On the frozen skeleton the ladder runs. Each rung, in order:

1. Reads the rubric and the problem statement **first**, before seeing the prior artifact.
2. Writes a three-sentence **frame-challenge**: if it were building from scratch, would it structure the artifact this way, and what would it change? A frame-challenge that calls for a *structural* change is escalated to the operator; it is not silently applied and not silently dropped.
3. Amends the artifact as it sees fit.
4. Emits a **KEEP / CHANGE / KILL ledger**: what it preserved and why it is defensible, what it changed and why, what it removed and why. Silent preservation is the anchoring failure mode; the KEEP column converts it into an explicit, auditable decision. A KEEP defended only by "it was already there" is a flagged smell.

Reading the rubric before the prior artifact, and writing the frame-challenge before editing, are what stop cumulative amendment from collapsing into sycophantic decoration of whatever the last rung produced.

### Teardown rung — for framing-heavy steps (Steps 1–4)

For the steps where the frame carries the most risk — discovery, vision, PRD, and decomposition — insert one pass whose job is to attack, not amend. It enumerates every structural weakness, unsupported assumption, and place the frame is wrong, and it produces findings, not an edited artifact. Red-teaming is psychologically opposite to improving, so a teardown model surfaces structural rot that a self-revising rung rationalizes past. Its findings feed the next fill rung or the operator.

### Ladder length is variable, not fixed at three

Stop when a rung's contribution goes cosmetic. Models carry a "must change something to justify the pass" reflex that adds verbosity and complexity rather than value, so more rungs are not monotonically better. Measure each rung by the size and substance of its diff and its CHANGE/KILL ledger; a rung that produces only cosmetic churn is the stop signal.

### Human gate

The operator accepts, rejects, or edits the final artifact. Only the approved version enters the next step's input packet, and the human edit delta is recorded (see cross-cutting controls) so correction effort remains measurable.

## Cold-session handoff packet

Every step directory should contain enough information to resume without chat history:

```text
step-N/
├── INPUT.md                 # Approved facts, prior artifacts, constraints, and task prompt
├── INPUT.sha256             # Frozen-packet checksum (committed, not just ritual)
├── RUBRIC.md                # Scored requirements and completion gate (frame-gated before freezing)
├── SKELETON.md              # Rung 0 frame, as accepted at the human frame-gate
├── rung-1.md … rung-N.md    # Each fill rung's amended artifact (immutable per-rung snapshots)
├── LEDGERS.md               # Frame-challenge + KEEP/CHANGE/KILL ledger for each rung
├── TEARDOWN.md              # Structural attack findings (framing-heavy steps)
├── DECISIONS.md             # Human decisions, frame-gate ruling, and rationale
├── ASSUMPTIONS.md           # Open and resolved assumptions
├── EVIDENCE.md              # Commands, tests, sources, and observed results
├── EDIT-DELTA.md            # Diff of operator edits to the final rung (correction-effort metric)
└── APPROVED.md              # Immutable input artifact for the next step
```

Record the model file, quantization, context setting, KV quantization, prompt-template version, llama.cpp version, date, and rung role for every pass.

## Cross-cutting controls

These apply to every step regardless of ladder shape:

- **Frozen, checksummed packet.** Each step's `INPUT.md` is frozen and its checksum recorded as a committed artifact (`INPUT.sha256`), not merely as a ritual step, because the ladder's validity depends on every rung receiving the identical packet.
- **The rubric is itself gated.** `RUBRIC.md` encodes one lineage's notion of "good" and is the one artifact whose blind spot would bias every score consistently. It passes its own frame-gate — an independent read before it is frozen — or its human authorship is recorded as a named bias source.
- **Honest gate taxonomy.** Steps 1–3 end at human-judgment gates; only Steps 4–6 add deterministic gates (schema validation, tests, coverage). The design does not claim machine rigor for the judgment steps. Where a "deterministic" gate depends on a validator that does not yet exist — requirement-ID→task-ID coverage for Step 4, finding-severity classification for Step 5 — that validator is named as required work, and until it exists the gate is labeled as human judgment.
- **Backflow / amendment.** `APPROVED.md` is immutable forward, but a late step can falsify an early one (Step 5 implementation exposing a wrong Step 3 requirement). A reopen-prior-step procedure with its own human gate and evidence trail is defined so the pipeline can iterate instead of breaking at the first genuine contradiction.
- **Immutable per-rung evidence.** Every rung's raw output is preserved before any later rung or human edit, so a regression — a later rung silently deleting a correct earlier addition — can be caught by diffing the chain.

## Final six-step process

### Step 1 — Discovery and rubber-duck analysis

**Purpose:** Establish what is being built, for whom, why it matters, and what is still unknown.

**Required output:**

- Problem statement
- Stakeholders and users
- Goals and measurable outcomes
- Constraints and non-goals
- Existing-system observations
- Alternatives considered
- Assumption register
- Open-question register
- Initial risk register
- Evidence and source references where tools were used

**Completion gate:** No unresolved question remains that could materially alter scope or architecture without being explicitly marked for later resolution.

**Ladder:** Rung 0 skeleton → fill rungs (weak→strong, mixed lineage) → teardown rung; discovery framing is high-leverage, so the teardown pass is mandatory here. Rung capability ranking and lineage assignment come from Enki_v2. Daemon rungs are optional escalation only when Kratos capacity is exceeded and ComfyUI is idle.

### Step 2 — Vision, architecture options, and roadmap

**Purpose:** Turn discovery into an intentional direction without prematurely fixing every implementation detail.

**Required output:**

- Product or system vision
- Candidate architectures and trade-offs
- Selected direction and decision rationale
- Milestones and dependencies
- Risk-adjusted sequence
- Success metrics
- Architecture decision records
- Rollback, migration, or abandonment criteria where applicable

**Completion gate:** The selected direction is traceable to Step 1 goals and constraints, and rejected alternatives have documented reasons.

**Ladder:** Rung 0 skeleton → fill rungs (weak→strong, mixed lineage) → teardown rung on the selected architecture. Rung ranking and lineage come from Enki_v2; Daemon rungs are optional escalation only.

### Step 3 — PRD / SRD and acceptance contract

**Purpose:** Define the product and system behavior precisely enough to decompose and verify.

**Required output:**

- Numbered functional requirements
- Numbered nonfunctional requirements
- Interfaces, schemas, and data ownership
- Error behavior and edge cases
- Security and privacy requirements
- Performance and resource budgets
- Observability requirements
- Compatibility and migration constraints
- Acceptance scenarios and testable outcomes
- Traceability from requirements to Step 1 goals

**Completion gate:** Every requirement is uniquely identified, testable or explicitly justified as qualitative, and free of unresolved interface ambiguity.

**Ladder:** Rung 0 skeleton (requirement/interface list) → fill rungs (weak→strong, mixed lineage) → teardown rung against the acceptance contract. Rung ranking and lineage come from Enki_v2; Daemon rungs are optional escalation only.

### Step 4 — Atomic implementation plan

**Purpose:** Convert the approved specification into bounded, executable tasks and verification batches.

**Required output:** Valid `TASKS.md` and `PROGRESS.json`, including:

- Requirement IDs satisfied by each task
- Preconditions and dependencies
- Expected files and interfaces
- Exact signatures, types, return values, and error behavior
- Estimated scope
- Exact test command and expected observable result
- Definition of done
- Batches of 2–4 tasks followed by a `TEST` gate
- Explicitly approved exceptions to normal task-size limits

**Default sizing rules:** No more than two files and approximately 50–150 changed lines per task. Exceptions require rationale, wider test coverage, and human approval.

**Completion gate:** Both artifacts pass schema validation; every implementation requirement is covered; dependencies form a valid order; and every batch ends in an executable gate.

**Ladder:** Rung 0 skeleton (task/batch structure) → fill rungs (weak→strong, mixed lineage) → teardown rung, with schema validation as the deterministic gate over all of it. Rung ranking and lineage come from Enki_v2; Daemon rungs are optional escalation only when the decomposition gate fails locally.

### Step 5 — Execute atomic batches and review evidence

**Purpose:** Implement approved tasks, verify each batch, and independently review the resulting changes.

**Required output:**

- Code and supporting artifacts
- Updated `PROGRESS.json`
- Per-task implementation notes
- Test, build, lint, type-check, and security-check evidence as applicable
- Diffs mapped to task IDs and requirement IDs
- Independent review findings
- Resolution record for every blocking finding

**Execution protocol:** An implementation model may work one atomic task at a time. After each 2–4-task batch, run the defined deterministic gate. A different-lineage reviewer examines the diff, contracts, tests, and requirement coverage. Failed gates return the batch for correction; they do not advance by model vote.

**Completion gate:** All task and batch checks pass, all blocking review findings are resolved, and the implementation remains traceable to the approved requirements.

**Ladder:** Rung 0 implementation plan per task → fill rungs (weak→strong, mixed lineage) amending code and notes, with the deterministic batch gate — build, tests, lint, type-check — carrying correctness over any model judgment. Rung ranking and lineage come from Enki_v2; Daemon rungs are optional escalation for failed, broad, or high-risk tasks only.

### Step 6 — Integrate, validate, and package the final deliverable

**Purpose:** Prove that the accepted parts form a reproducible whole and produce the final handoff or release package.

**Required output:**

- Clean integrated tree
- Clean-checkout build and setup evidence
- Full unit, integration, and end-to-end test results
- Requirement-to-test coverage matrix
- Release notes and user/developer documentation
- Known limitations and unresolved risks
- Migration, rollback, or recovery instructions where applicable
- Final decision record

**Completion gate:** A clean checkout can reproduce the build and pass the complete acceptance suite; documentation matches observed behavior; and every requirement is implemented, deferred with approval, or rejected with rationale.

**Ladder:** Rung 0 integrated skeleton → fill rungs (weak→strong, mixed lineage) → a final release-audit rung (teardown against the acceptance suite and docs), with the clean-checkout build and full test run as the deterministic gate. Rung ranking and lineage come from Enki_v2; Daemon rungs are optional escalation only.

## Rung roles and lineage diversity

Each step's ladder fills the same roles regardless of which models occupy them:

| Role | Capability rank | Lineage rule |
| :--- | :--- | :--- |
| Rung 0 — skeleton / frame | High — framing is not laddered from the weakest | Any qualified lineage |
| Fill rungs | Ascending, least → most capable | Each rung a different training lineage from the last |
| Teardown (Steps 1–4; Step 6 release audit) | High; attacks rather than amends | Different lineage from the final fill rung where possible |
| Human gate | — | Operator |

Which concrete models occupy each rank, per step, is supplied by Enki_v2. This project consumes that ranking; it does not produce it. Two rungs from the same lineage do not count as independent eyes, because overlapping-lineage models share the blind spots the ladder exists to cancel.

## Model inputs from Enki_v2

Model capability ranking, per-step task-quality scoring, and hardware-fit qualification are **not** produced by this project. They are the output of the separate Enki_v2 evaluation and benchmarking project, which feeds this one. SBL SDLC consumes two things from Enki_v2:

- **Hardware-runnable pool** — which models load on Kratos (single card) and Daemon (two cards), at what context and KV quantization.
- **Per-step capability ranking** — the weak→strong ordering used to place models on each step's ladder, and the lineage of each, so rungs can be assigned and lineage diversity enforced.

The scoring dimensions Enki uses to rank a model for a step — requirement coverage, factual grounding, explicit assumptions, internal consistency, interface precision, testability, defect discovery, scope discipline, schema validity, and human-correction effort — are also the dimensions each step's `RUBRIC.md` scores an *artifact* against; the two share a vocabulary but serve different masters (Enki ranks models, the rubric ranks artifacts).

**Open interface:** the concrete handoff format between Enki_v2 and SBL SDLC — how a ranked, lineage-tagged, hardware-qualified model list per step is delivered and versioned — is not yet defined. Until it is, the placeholder is a static ranked list maintained in the vault and cited by version in each step's `INPUT.md`.

## Current hardware-runnable pool (snapshot)

This is the hardware-fit snapshot consumed from the load tests (an Enki_v2 input, recorded here for convenience). It says only what *loads*; capability ranking among these models is Enki's, not implied by their order here.

### Two-card Daemon pool

- Qwen 3.6 35B-A3B — confirmed 262K, Q8 KV, two cards
- Gemma 4 26B A4B — confirmed 262K, Q8 KV, two cards
- Qwen 3.6 27B — confirmed 262K, Q8 KV, two cards; useful additional lineage within Qwen training but not fully independent from Qwen 3.6 35B
- GLM 4.7 Flash — confirmed 202,752, Q8 KV, two cards; useful family-diversity candidate
- LFM2 24B A2B — confirmed 128K, Q8 KV, two cards; useful family-diversity candidate

### Single-card Kratos pool

- Qwen 3.5 9B — confirmed 262K with Q4 KV
- Gemma 4 12B — confirmed 131K with Q8 KV
- GPT-OSS 20B — confirmed 131K with Q4 or Q8 KV
- Kimi-VL A3B — confirmed 131K with Q8 KV
- DeepSeek Coder V2 Lite — confirmed 65K with Q4 KV
- Phi-4 — confirmed 16K effective context; specialized short-context option
- Granite 3.3 8B Instruct — confirmed 131K with Q4 KV; Q8 KV failed at 131K from insufficient compute-buffer headroom
- Falcon-H1 7B Instruct — confirmed full 262K training context with Q8 KV

### Conditional three-card pool

- Gemma 4 31B — confirmed 262K with Q8 KV on three cards

## Immediate next actions

1. Define the step-directory scaffolding for the amendment ladder: `INPUT.md` + `INPUT.sha256`, `RUBRIC.md`, `SKELETON.md`, per-rung snapshots, `LEDGERS.md`, `TEARDOWN.md`, `EDIT-DELTA.md`, and `APPROVED.md`.
2. Freeze one representative project concept and write the Step 1 `INPUT.md` and `RUBRIC.md`; give the rubric its own frame-gate before freezing.
3. Obtain the Step 1 model ladder from Enki_v2 — a hardware-runnable, weak→strong, lineage-tagged ranking — and define the Enki_v2 → SBL SDLC handoff interface (currently undefined).
4. Run Rung 0 to produce `SKELETON.md`; take it through the human frame-gate before any fill rung runs.
5. Run the fill ladder weak→strong with the frame-challenge and KEEP/CHANGE/KILL ledger at each rung, then the teardown rung; record raw per-rung outputs, runtime metadata, and the operator edit delta.
6. Approve `step-1/APPROVED.md`, then start fresh sessions for Step 2 and repeat through Step 6 without relying on prior chat memory or Daemon availability.
7. Build the missing deterministic validators before Steps 4–6 need them: requirement→task coverage (Step 4) and finding-severity classification (Step 5).
8. Run the ladder-vs-top-rung-solo baseline on a representative step to confirm the lower rungs earn their cost.

## Open research questions

- Does weak→strong cumulative amendment beat the top rung running the step alone, and at which steps does the margin justify the added passes?
- How often does the frame-gate actually catch a bad skeleton, versus adding operator overhead for frames that were fine?
- Does the teardown rung surface structural defects the fill rungs miss, or does it mostly restate the rubric?
- How much does lineage diversity help after controlling for capability rank?
- What context size is actually needed at each step, rather than merely available?
- Does Q4 KV materially degrade long-context review or requirement recall on the Kratos pool?
- Can the full process be resumed solely from the step directory after every session is discarded?

---

# Part IV — Opus 4.8 Review (2026-07-13, merged)

An Opus 4.8 review challenged the four areas Part III's predecessor asked an external model to stress — artifact contracts, independence controls, completion gates, and the Kratos-first topology — under the project's local-only, weakness-cancellation premise. Its corrections have been **merged into Part III**; this section records what changed and why, in the doc's layered-history style.

The largest finding retired the design's central assumption. The blind-independence quorum (two independent drafts plus adjudication) was an artifact of an earlier mistaken goal; the operator's actual intent is a **cumulative amendment ladder**, where each rung alters the artifact as it sees fit. Part III's protocol was rewritten to that model. The review's job then became protecting the ladder from its one hazard — the *albatross*, where a weak early frame anchors every later rung — which Part III now handles by gating the frame before filling, forcing a frame-challenge and KEEP/CHANGE/KILL ledger at each rung, adding a non-editing teardown pass for framing-heavy steps, and letting ladder length vary.

The protocol-agnostic corrections became Part III's **cross-cutting controls**: a checksummed frozen packet as a real artifact, an independence gate on the rubric itself, an honest gate taxonomy (Steps 1–3 are human-judgment; only 4–6 add deterministic gates) with the missing validators named, a backflow procedure for when a late step falsifies an approved early one, and immutable per-rung evidence. The control-arm idea was reframed to the local-only premise: not a cloud upper bound but the **top rung running the step solo**, diffed against the full ladder to prove the lower rungs earn their cost.

Two of the original review's points were withdrawn as incompatible with the chosen ladder: the recommendation for blind-parallel authorship on Steps 1–3, and the defect-disjointness metric across parallel passes — replaced, respectively, by the frame-gate and by per-rung marginal-value measurement (did this rung repair the prior rung's defects and add value).

---

## Changelog

- 2026-07-13 (Opus 4.8 merge) — Merged the Opus 4.8 review into Part III and rewrote the protocol from the blind-independence quorum to the operator's cumulative amendment ladder: skeleton + human frame-gate before filling, weak→strong cross-lineage fill rungs each with a frame-challenge and KEEP/CHANGE/KILL ledger, a non-editing teardown rung for Steps 1–4 (and a Step 6 release audit), and variable ladder length. Added cross-cutting controls (checksummed packet, rubric independence gate, honest gate taxonomy + named validators, backflow, immutable per-rung evidence). Scoped model auditions and capability ranking out to Enki_v2, which feeds this project; reframed the step ladder lines, the rung-role table, the model-inputs section, the hardware-pool snapshot, next actions, and open questions accordingly. Collapsed Part IV to a merged-review record.
- 2026-07-13 (Opus 4.8 review) — Initial Part IV review challenging artifact contracts, independence controls, completion gates, and the Kratos-first topology (now merged into Part III).
- 2026-07-13 (Codex) — Adopted into the `sbl-sdlc` repository as the project's deep-context research handoff.
- 2026-07-13 (Gemini 3.1 Pro / Opus review context) — Original local-model SDLC proposal and candidate mapping.
- 2026-07-13 (GPT 5.6 review) — Clarified the cold-session multi-lineage goal; corrected the missing implementation phase, artifact contracts, deterministic gates, context assumptions, and empirical hardware classifications.
- 2026-07-13 (current) — Recast the project as a six-step, up-to-three-pass local deliberation protocol with reproducible handoff packets and human-controlled state transitions.
- 2026-07-13 (topology correction) — Established Kratos as the primary background pipeline machine, Daemon as optional escalation/backup capacity, removed stale Llama/Mistral/Nemotron availability assumptions, and added Granite 3.3 8B and Falcon-H1 7B to the pending Kratos audition pool.
- 2026-07-13 (Kratos qualification) — Recorded Granite 3.3 8B at 131K with Q4 KV and Falcon-H1 7B at full 262K with Q8 KV on the RX 7800 XT; both are now hardware-qualified and pending task-quality auditions.
