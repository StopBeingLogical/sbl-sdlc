---
type: notes
title: Local-Only Multi-Model SDLC Research — Ideation, Review, and Current Design
description: Historical proposal, GPT 5.6 review, empirical corrections, and current six-step cold-session workflow for local open-weight models
timestamp: 2026-07-13T13:52:00Z
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

For each of six SDLC steps, the operator begins from a cold session, gives one model the approved input packet, saves its output, and then repeats the step with a model from a different family or training lineage. A third model may review or adjudicate. This provides up to three independent sets of eyes per step while avoiding dependence on a single model's blind spots or accumulated conversational context.

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
3. Each step receives at least two independent model passes when practical, from different model families or training lineages.
4. A third pass is used for high-risk steps, unresolved disagreements, or final reconciliation.
5. Models produce proposals and critiques; the human operator approves state transitions.
6. Deterministic tests and validators have authority over prose claims of correctness.
7. No hidden conversational state is required to resume the project.
8. Local inference is the target operating mode; local search, repositories, documentation mirrors, and test tools may be provided as explicit tools.
9. Kratos is the primary always-available background worker for the pipeline.
10. Daemon is a secondary escalation and backup machine because its GPUs normally serve ComfyUI image and video generation.
11. No normal step may require Daemon in order for the workflow to continue; Daemon models are optional higher-capacity eyes.

## Final operating topology

### Primary tier — Kratos

Kratos runs the routine author, reviewer, and adjudicator sessions sequentially on its single RX 7800 XT 16 GB. A three-member quorum does not imply three models loaded concurrently. Each cold pass loads one model, writes its artifacts, unloads, and yields to the next lineage.

The Kratos pool therefore determines whether the local-only workflow is operational. Long-running discovery, review, decomposition, implementation, and validation jobs should default to this machine.

### Secondary tier — Daemon

Daemon is used when:

- ComfyUI is idle or sufficient cards are available
- A step exceeds the useful context or capability of the Kratos pool
- A higher-capacity tie-breaker is warranted
- Kratos is unavailable
- A controlled comparison against the larger MoE models is being conducted

Daemon capacity should be treated as opportunistic. Two-card models should not silently become required dependencies, and Gemma 4 31B should remain a special three-card experiment.

## Standard three-pass protocol for every step

### Pass A — Independent author

The author receives only the approved input packet and the step rubric. It produces a complete candidate artifact and a list of assumptions, uncertainties, and risks.

### Pass B — Cross-lineage critic

The critic receives the same approved input packet, the rubric, and Pass A's output. It must identify omissions, unsupported assumptions, contradictions, scope drift, and concrete corrections. It should not silently rewrite the artifact without explaining disagreements.

### Pass C — Adjudicator or second independent author

Use Pass C when the step is consequential or when A and B disagree. Depending on the experiment, the third model may:

- Produce a second independent solution before seeing A or B
- Adjudicate the differences after seeing both
- Validate the reconciled artifact against the rubric

The chosen mode must be recorded so that independence is not overstated.

### Human gate

The operator accepts, rejects, or edits the reconciled artifact. Only the approved version enters the next step's input packet.

## Cold-session handoff packet

Every step directory should contain enough information to resume without chat history:

```text
step-N/
├── INPUT.md                 # Approved facts, prior artifacts, constraints, and task prompt
├── RUBRIC.md                # Scored requirements and completion gate
├── pass-a-author.md         # First model output
├── pass-b-critic.md         # Cross-lineage review
├── pass-c-adjudicator.md    # Optional third pass
├── DECISIONS.md             # Human decisions and rationale
├── ASSUMPTIONS.md           # Open and resolved assumptions
├── EVIDENCE.md              # Commands, tests, sources, and observed results
└── APPROVED.md              # Immutable input artifact for the next step
```

Record the model file, quantization, context setting, KV quantization, prompt-template version, llama.cpp version, date, and role for every pass.

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

**Primary Kratos eyes:** Qwen 3.5 9B, Gemma 4 12B, and GPT-OSS 20B are the initial quorum candidates. Granite 3.3 8B, Falcon-H1 7B, or Kimi-VL may replace or supplement a member after context and task-quality auditions.

**Daemon escalation:** Gemma 4 26B A4B, Qwen 3.6 35B-A3B, GLM 4.7 Flash, or LFM2 24B A2B when additional capacity is justified and available.

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

**Primary Kratos eyes:** GPT-OSS 20B and Gemma 4 12B as blind authors, with Qwen 3.5 9B, Granite 3.3 8B, or Falcon-H1 7B as adjudicator candidates after auditioning.

**Daemon escalation:** Qwen 3.6 35B-A3B or Gemma 4 26B A4B. Gemma 4 31B is an optional three-card synthesis experiment only when ComfyUI is idle.

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

**Primary Kratos eyes:** GPT-OSS 20B, Gemma 4 12B, and Qwen 3.5 9B, with Granite 3.3 8B as a promising requirements and structured-output candidate pending audition. DeepSeek Coder V2 Lite can provide a code-oriented pass within its confirmed 65K limit.

**Daemon escalation:** Qwen 3.6 35B-A3B or Gemma 4 26B A4B for unusually large or difficult specifications.

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

**Primary Kratos eyes:** Run Qwen 3.5 9B, Granite 3.3 8B, GPT-OSS 20B, Gemma 4 12B, and DeepSeek Coder V2 Lite through the same decomposition audition. Select two different-family blind authors and a third-family adjudicator from the winners.

**Daemon escalation:** Compare the approved Kratos result against Qwen 3.6 35B-A3B or Gemma 4 26B A4B when Daemon is free or the decomposition gate fails locally.

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

**Primary Kratos eyes:** Use the best auditioned Kratos coding model as implementer and a different-family Kratos model as reviewer. DeepSeek Coder V2 Lite, Qwen 3.5 9B, GPT-OSS 20B, Granite 3.3 8B, and Gemma 4 12B should compete on identical atomic tasks.

**Daemon escalation:** Use Qwen3-Coder 30B A3B, Qwen 3.6 35B-A3B, or another qualified Daemon model only for failed, unusually broad, or high-risk tasks.

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

**Primary Kratos eyes:** Use the best Kratos synthesis model and a different-family release auditor. GPT-OSS 20B, Gemma 4 12B, Qwen 3.5 9B, and Granite 3.3 8B are the initial candidates.

**Daemon escalation:** Qwen 3.6 35B-A3B or Gemma 4 26B A4B may provide a higher-capacity final audit. Gemma 4 31B remains an optional three-card editorial pass, not a dependency.

## Model-lineage rotation strategy

The same pair does not need to author and review every step. A useful audition matrix is:

| Capability | Author candidate | Cross-lineage critic | Optional third eye |
| :--- | :--- | :--- | :--- |
| Discovery and ambiguity | GPT-OSS 20B | Gemma 4 12B | Qwen 3.5 9B, Granite 3.3 8B, Falcon-H1 7B, or Kimi-VL |
| Vision and roadmap | GPT-OSS 20B | Gemma 4 12B | Qwen 3.5 9B, Granite 3.3 8B, or Falcon-H1 7B |
| PRD / SRD | GPT-OSS 20B | Gemma 4 12B | Qwen 3.5 9B or Granite 3.3 8B |
| Atomic decomposition | Winner of Kratos decomposition audition | Different-family Kratos winner | Third-family Kratos adjudicator |
| Implementation | Winner of Kratos coding audition | Different-family Kratos code reviewer | Third-family Kratos adjudicator |
| Integration and release | Winner of Kratos synthesis audition | Different-family Kratos release auditor | Daemon escalation only if needed |

This table is an audition plan, not a declaration of winners.

## Audition methodology

### Hardware qualification

For each candidate, record:

- Whether it loads
- Effective context after model capping
- Weight and KV quantization
- GPU count and tensor split
- Prompt-processing speed
- Generation speed
- Peak VRAM and host RAM
- Stability across repeated starts and long prompts

### Task-quality qualification

Use the same frozen input packet for every candidate. Score outputs without relying on model self-assessment.

Recommended scoring categories:

- Requirement coverage
- Factual grounding
- Explicit assumptions
- Internal consistency
- Interface precision
- Testability
- Defect discovery
- Scope discipline
- Artifact/schema validity
- Human correction effort
- Latency and throughput

For Steps 4–6, deterministic artifact validation and executable tests should be reported separately from qualitative scores.

### Independence controls

- For a true independent third solution, do not show it previous model outputs.
- For a critic, show the candidate but require cited findings and proposed corrections.
- For an adjudicator, show all competing artifacts and require a decision table.
- Preserve raw outputs before human editing.
- Record prompt and sampling settings so results can be repeated.

## Current practical lineup

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

1. Freeze one representative project concept and create the Step 1 `INPUT.md` and `RUBRIC.md`.
2. Run controlled task-quality and speed auditions for Granite 3.3 8B and Falcon-H1 7B; their Kratos context fit is now confirmed.
3. Run Step 1 as two blind author passes on different Kratos families.
4. Use a third Kratos model to adjudicate after both blind outputs are frozen.
5. Record raw outputs, settings, latency, and human edits.
6. Approve `step-1/APPROVED.md`, then start fresh sessions for Step 2.
7. Repeat through Step 6 without relying on prior chat memory or Daemon availability.
8. Separately run controlled speed and prompt-length benchmarks; do not mix hardware fit scores with artifact-quality scores.
9. Use Daemon as an explicitly recorded escalation, not as the silent default.
10. Give this revised design to Gemini and Opus for another review, asking them specifically to challenge artifact contracts, independence controls, completion gates, and the Kratos-first topology.

## Open research questions

- Is critique-after-draft better than two blind independent drafts followed by adjudication?
- At which steps does a third model materially improve quality relative to operator time?
- How much does family diversity help after controlling for model capability?
- What context size is actually needed at each step, rather than merely available?
- Does Q4 KV materially degrade long-context review or requirement recall for these models?
- Which model minimizes human correction effort for each artifact type?
- Can the full process be resumed solely from the step directory after every session is discarded?

---

## Changelog

- 2026-07-13 (Codex) — Adopted into the `sbl-sdlc` repository as the project's deep-context research handoff.
- 2026-07-13 (Gemini 3.1 Pro / Opus review context) — Original local-model SDLC proposal and candidate mapping.
- 2026-07-13 (GPT 5.6 review) — Clarified the cold-session multi-lineage goal; corrected the missing implementation phase, artifact contracts, deterministic gates, context assumptions, and empirical hardware classifications.
- 2026-07-13 (current) — Recast the project as a six-step, up-to-three-pass local deliberation protocol with reproducible handoff packets and human-controlled state transitions.
- 2026-07-13 (topology correction) — Established Kratos as the primary background pipeline machine, Daemon as optional escalation/backup capacity, removed stale Llama/Mistral/Nemotron availability assumptions, and added Granite 3.3 8B and Falcon-H1 7B to the pending Kratos audition pool.
- 2026-07-13 (Kratos qualification) — Recorded Granite 3.3 8B at 131K with Q4 KV and Falcon-H1 7B at full 262K with Q8 KV on the RX 7800 XT; both are now hardware-qualified and pending task-quality auditions.
