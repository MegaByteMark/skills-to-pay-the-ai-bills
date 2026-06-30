---
name: gather-requirements
description: Conducts an exhaustive, granular engineering discovery by executing the interview-me skill across two streams — a product stream that produces a Product Requirements Document (PRD) of vision, personas, Epics and prioritised user stories that seed the backlog, and a functional stream that produces a complete Functional Design Specification (FDS) of behavioral truth. The FDS traces every requirement back to its originating PRD Epic/story. Runs in two origins — interview (greenfield elicitation) or reverse-engineer (brownfield), which reconstructs a draft PRD/FDS from the existing codebase and the open/closed issue backlog, tags every reconstructed item with calibrated [Confidence: Level] and source provenance, then runs a confirmation-only interview over the gaps and low-confidence rows so an inherited system with no formal PRD/FDS can be converged with product over time. Creates these artifacts from scratch or amends existing ones in place, preserving stable IDs and a revision history.
dependencies:
  - interview-me  # Drives the single-question interactive conversation loop and context extraction mechanics
  - agent-markup  # Governs [Auth: Scope], [Policy], [Data: Classification], [Priority: MoSCoW], [Confidence: Level], and [Inferred: Unverified] token enumerations
  - resolve-repository-platform  # Read-side ingestion of the open/closed issue backlog as a requirement-evidence source in reverse-engineer origin
argument-hint: "[prd | fds | full] [create | amend] [interview | reverse-engineer]  # stream(s) to run; create/amend mode (auto-detected from existing files when omitted); and origin — interview (default; greenfield elicitation) or reverse-engineer (brownfield: draft from code + issue backlog, then confirm)"
---

This skill runs requirements discovery as a two-stream pipeline that mirrors how modern agile teams work: the **product stream** establishes *why* and *what* (the PRD that feeds Epics and backlog stories), and the **functional stream** establishes *how it must behave* (the FDS that downstream auditing agents consume). The functional stream is seeded from, and traces back to, the product stream.

The skill runs in one of two **origins**, selected in Phase 0:
- **`interview` (default — greenfield):** elicit truth from a human, blank-slate, one question at a time. Use when the people hold the knowledge and can answer.
- **`reverse-engineer` (brownfield):** for an inherited system with no formal PRD/FDS, no single owner, and a fragmented product team, where a full elicitation interview would take weeks. The agent first **reconstructs a draft** PRD/FDS from hard evidence — the existing codebase (via the `analyze-a-codebase` blueprint when present) and the open/closed issue backlog — tags every reconstructed item with calibrated `[Confidence: Level]` and a source reference, then runs a **confirmation-only** interview scoped to the gaps and the low-confidence rows. The draft is evidence-first and anti-hallucination: a needed fact with no evidence is written `Unknown — requires verification`, never invented.

Operational Workflow:
1. PHASE 0 (Stream, Mode & Origin Selection): Determine which stream(s) to run from the invocation argument — `prd`, `fds`, or `full` (default). `full` runs the PRD stream first, then carries its Epics/stories forward as the seed for the FDS stream. Running `fds` standalone is permitted when a PRD already exists at the output path; if none exists, announce that the FDS will proceed without product traceability. Then resolve the **mode per stream**: if the target document already exists at its output path, default to `amend` (run the Amendment Protocol) rather than overwriting; otherwise `create`. A user may force either mode explicitly. Never blind-overwrite an existing document. Finally resolve the **origin** — `interview` (default) or `reverse-engineer`. When the user reports an inherited/unfamiliar codebase, or no clear human can supply the requirements, recommend `reverse-engineer` rather than forcing a blank-slate interview.
2. PHASE 1 (Context Discovery / Evidence Reconstruction):
     - **`interview` origin:** Scan any provided project briefs, issue templates, existing PRDs/FDS, or codebases to establish initial domain context before interviewing.
     - **`reverse-engineer` origin:** Run the Reverse-Engineer Reconstruction Protocol (below). Ingest the evidence sources, reconstruct a *provisional* Epic Register, User Story Backlog, and FDS requirement set, and tag every reconstructed row with `[Confidence: Level]` plus its source provenance. This draft — not a blank page — is the input to the interview phases, which become confirmation passes.
3. PHASE 2 (PRD Elicitation / Confirmation Interview): Trigger and execute the `interview-me` skill directed at product-discovery branches (see "PRD Interview Branches"). Extract the problem, target personas, measurable goals, Epics, and granular user stories with acceptance criteria and `[Priority: MoSCoW]` ranking. Skip when the stream selector is `fds`.
     - **`reverse-engineer` origin:** Do NOT re-elicit from scratch. Present the reconstructed PRD draft and run the interview as a **confirmation pass** scoped to (a) every row tagged `[Confidence: Possible]`, (b) every `Unknown — requires verification` field, and (c) reconstructed items that conflict with each other. Confirmed/Probable rows are carried forward unless the user corrects them. Frame each question as "I inferred X from <source>; confirm, correct, or flag for product" rather than "what should X be?". On confirmation, upgrade the row's `[Confidence: Level]` and replace the provenance note with the confirming authority.
4. PHASE 3 (PRD Document Generation): Compile the verified product requirements into the PRD, matching the "PRD Markdown Schema" below, written to the permanent requirements directory. Assign stable `EPIC-###` and `STORY-###` identifiers.
5. PHASE 4 (FDS Elicitation / Confirmation Interview): Trigger and execute the `interview-me` skill again, directed to bypass high-level narrative fluff and explicitly extract fine-grained functional rules, cross-field validations, data transformations, security boundaries, and exception states. When a PRD exists, walk its stories one Epic at a time so every behavior maps to a story. Skip when the stream selector is `prd`.
     - **`reverse-engineer` origin:** Run as a **confirmation pass** over the reconstructed FDS draft, scoped to `[Confidence: Possible]` rows, `Unknown — requires verification` fields, and behaviors with no traceable `STORY-###`. Validation limits, authorization scopes, and error states read from code are `[Confidence: Confirmed|Probable]` evidence; behaviors inferred only from issue text or naming are `[Confidence: Possible]` and must be confirmed with product before being asserted.
6. PHASE 5 (FDS Document Generation): Compile the verified requirements into the FDS, matching the "FDS Markdown Schema" below. Populate the Source (PRD) traceability column on every requirement row.

## Operational Directives
- Execution Protocol: You must use the `interview-me` skill mechanics for every interview phase. Ask exactly ONE highly specific question at a time to prevent human cognitive fatigue. Honor the `interview-me` advancement contract: never move to the next question until the user issues the literal `move-next` command — answering or asking follow-ups does not authorize advancing.
- Stream Sequencing: In `full` mode, do NOT begin the FDS stream until the PRD is written and its Epic/story IDs are assigned, because those IDs are the FDS traceability anchors.
- Output Location Contract: The PRD must be written to exactly `docs/requirements/product-requirements.md` and the FDS to exactly `docs/requirements/functional-requirements.md`, both relative to the repository root.
- Traceability Contract: Every FDS requirement row must reference at least one originating `STORY-###` (or `EPIC-###` where a requirement is cross-cutting) in its Source (PRD) column. If a behavior has no product origin, flag it inline as `[Inferred: Unverified]` so the gap is visible rather than silently invented.
- Detail Threshold: Reject open-ended or vague human responses (e.g., "The form should save data securely" or "users want a nice dashboard"). For the PRD, force measurable success metrics, explicit personas, and testable acceptance criteria. For the FDS, force validation limits, precise authorization scopes, and error states.
- Origin & Evidence Calibration (reverse-engineer): Every reconstructed row MUST carry an inline `[Confidence: Level]` and a source provenance reference (e.g. `src/OrderService.cs:88`, `issue #231 (closed)`, `blueprint §2.2`). Calibrate strictly against observed evidence: `Confirmed` = behaviour directly read from code/config; `Probable` = strong code indicator missing one corroborating link; `Possible` = inferred from issue text, naming, or heuristics only — phrase these as "requires verification" and NEVER assert them as fact. Any needed field with no evidence is written `Unknown — requires verification`. A reconstructed FDS row with no traceable product origin still takes `[Inferred: Unverified]` per the Traceability Contract.
- Structural Layout: Each generated document must rigorously match its schema below and remain export-clean Markdown per the `agent-markup` Output Portability Convention.

## Amendment Protocol
When a stream runs in `amend` mode, the existing document is the authoritative baseline — you are editing it, not regenerating it:
- Diff-Scoped Interview: Load the existing document and confirm with the user exactly which areas changed. Run `interview-me` ONLY over those areas; do not re-elicit settled requirements. Carry unchanged sections forward verbatim.
- Append-Only Identifiers: NEVER renumber or reuse IDs. New items take the next free `EPIC-###` / `STORY-###` / `REQ-###`. Removed or abandoned items are NOT deleted — set their `Status` to `Deprecated` (and, for PRD backlog items, `[Priority: Wont]`) so traceability and history survive.
- Cross-Stream Drift Reconciliation: When a PRD Epic/story is amended or deprecated, immediately flag every FDS requirement whose `Source (PRD)` references it for review, and resolve the divergence in the same session. A changed product intent must never leave a stale FDS row silently behind.
- Revision History: Append a new row to the document's Revision History table (date, summary of change, affected IDs) on every amendment. Bump the document version.
- Write-Back & Versioning: Write the amended document back to the same canonical output path. Git is the version store — do NOT create parallel `-vN` files or copies.

## Reverse-Engineer Reconstruction Protocol
Runs in Phase 1 when the origin is `reverse-engineer`. Produces the *provisional* draft that the Phase 2/4 confirmation interviews then converge. Evidence-first and anti-hallucination throughout.
- Evidence Sources (ingest before drafting):
  1. **Code structure** — prefer the existing `analyze-a-codebase` blueprint at `docs/architecture/system-blueprint.md` (Modules, Seams, data dictionary, persona registry). If it is absent, recommend running `analyze-a-codebase` first; proceed against a direct code scan only if the user declines, and tag the resulting rows no higher than `[Confidence: Probable]`.
  2. **Issue backlog** — resolve the host via the `resolve-repository-platform` skill, then read BOTH open and closed Work Items. Closed/merged items are evidence of *delivered* behaviour (lean `[Confidence: Probable]`); open items are evidence of *intended* behaviour and ranked into the backlog with `[Priority: MoSCoW]` and `[Confidence: Possible]`. Never treat an issue's wishful title as confirmed behaviour.
- Reconstruction Steps:
  1. Derive candidate **Epics** from Module/capability clusters in the blueprint and recurring backlog themes.
  2. Derive candidate **user stories** from closed feature issues and observable user flows; write each `As a / I want / So that` with acceptance criteria reconstructed from the evidence, tagged with confidence + provenance.
  3. Derive candidate **FDS requirements** from validations, authorization checks, state transitions, and error handling read in code; map each to its candidate `STORY-###`, or mark `[Inferred: Unverified]` where no product origin exists.
  4. Record every conflict (code says one thing, an issue says another) as an explicit confirmation item rather than silently picking a winner.
- Output of the protocol: a complete draft PRD/FDS where the proportion of `Possible`/`Unknown` rows is the explicit, visible measure of how much product confirmation remains — this is the queue the confirmation interview burns down over subsequent sessions (use `amend` mode across sessions; never renumber).

PRD Interview Branches (product stream):
  - Problem & Business Intent: the pain being solved, the commercial leverage, and why now.
  - Target Personas & Jobs-to-be-Done: who uses it, their goals, and their authentication scope (`[Auth: Scope]`).
  - Goals & Success Metrics: measurable outcomes and the KPIs that prove success or failure.
  - Epic Decomposition: the major capability themes that group stories.
  - User Stories & Acceptance Criteria: `As a / I want / So that` statements with testable acceptance criteria and `[Priority: MoSCoW]` ranking.
  - Scope Boundaries: explicit out-of-scope items, assumptions, external dependencies, and release milestones.

## PRD Markdown Schema

# Product Requirements Document (PRD)

## 0. Document Control & Revision History
*(Append a row on every amendment; git stores the full diff. Document version uses semantic-ish bumps: major for scope changes, minor for additions/edits.)*
| Version | Date | Change Summary | Affected IDs |
| :--- | :--- | :--- | :--- |
| *1.0* | *2026-06-29* | *Initial PRD.* | *EPIC-001, STORY-001* |

## 1. Product Vision & Problem Statement
* **Problem Statement:** The user/business pain being solved and why it matters now.
* **Core Business Intent:** High-density summary of the product's primary commercial leverage.
* **Target Personas:** | Persona / Role | Job-to-be-Done | System Access Level (`[Auth: Scope]`) | Primary Value Delivered |

## 2. Goals & Success Metrics
| Goal | Success Metric / KPI | Baseline | Target |
| :--- | :--- | :--- | :--- |

## 3. Epic Register
*(Epics are the capability themes that group user stories and structure the backlog. `Status`: Active | Deprecated — deprecated Epics are retained, never deleted.)*
| Epic ID | Epic Title | Outcome / Definition of Done | `[Priority: MoSCoW]` | Status |
| :--- | :--- | :--- | :--- | :--- |
| *EPIC-001* | *Streaming Ingestion* | *Operators can ingest large feeds without local serialization.* | *Must* | *Active* |

## 4. User Story Backlog
*(This registry is the direct seed for the FDS functional stream and the team's backlog. `Status`: Active | Deprecated. In `reverse-engineer` origin, append an inline `[Confidence: Level]` and a source reference to each reconstructed row, and write any unevidenced-but-needed cell as `Unknown — requires verification`.)*
| Story ID | Epic ID | User Story (`As a / I want / So that`) | Acceptance Criteria | `[Priority: MoSCoW]` | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| *STORY-001* | *EPIC-001* | *As an operator, I want to ingest 50MB feeds so that I can process partner data live.* | *Given a 50MB feed, when ingested, then no local disk serialization occurs and throughput stays within SLA.* | *Must* | *Active* |

## 5. Scope Boundaries & Assumptions
* **Out of Scope:** Items explicitly excluded this cycle (record `[Priority: Wont]` decisions here rather than dropping them silently).
* **Assumptions & Dependencies:** External services, teams, or conditions the plan relies on.
* **Release Milestones:** High-level sequencing of Epics across releases.

## FDS Markdown Schema

# Functional Design Specification (FDS)

## 0. Document Control & Revision History
*(Append a row on every amendment; git stores the full diff.)*
| Version | Date | Change Summary | Affected IDs |
| :--- | :--- | :--- | :--- |
| *1.0* | *2026-06-29* | *Initial FDS.* | *REQ-001* |

## 1. Document Governance & System Scope
* **Core Business Intent:** High-density summary of why this system exists and its primary commercial leverage.
* **Actor & Persona Matrix:** | Persona / Role | System Access Level | `[Auth: Scope]` | Operational Boundary / Responsibility |

## 2. Granular Functional Requirements (The Audit Contract)
*(Note: This structured registry is the direct interface consumed by downstream automated auditing agents. The Source (PRD) column traces each behavior to the product intent that justifies it. `Status`: Active | Deprecated — deprecated rows are retained for history, never deleted. In `reverse-engineer` origin, append an inline `[Confidence: Level]` and a source reference (code path, `issue #` and state, or `blueprint §`) to each reconstructed requirement; `Possible`-confidence rows are phrased as requiring verification until product confirms them.)*
| Feature ID | Source (PRD) | Functional Requirement / Technical Capability | Target Module / View | Policy Stance (`[Policy]`) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| *REQ-001* | *STORY-001* | *Must process concurrent data streams up to 50MB without local disk serialization.* | *IngestionAdapter* | *Enforced* | *Active* |

## 3. Data Invariants & Domain Validation Rules
* **State Transition Logic:** List valid state pathways and conditions required for mutation.
* **Validation Schema Matrix:**
  | Context / Entity | Field Name | Type / Constraints | Failure Mode / Exception Rule |

## 4. Operational Boundaries & Security Profiles
* **Data Isolation Model:** Explicit rules regarding multi-tenancy boundaries and storage access rules.
* **Data Sensitivity Register:** Classify each significant entity so downstream audits can detect drift. Treat `Special-Category` (GDPR Art. 9) as the strictest tier.
  | Entity / Data Element | Sensitivity (`[Data: Classification]`) | Lawful Basis / Handling Note |
  | :--- | :--- | :--- |
* **SLA & Performance Baselines:** High-priority execution limits (e.g., UI responsiveness, batch windows).
