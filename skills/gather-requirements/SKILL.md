---
name: gather-requirements
description: Conducts an exhaustive, granular engineering discovery by executing the interview-me skill across two streams — a product stream that produces a Product Requirements Document (PRD) of vision, personas, Epics and prioritised user stories that seed the backlog, and a functional stream that produces a complete Functional Design Specification (FDS) of behavioral truth. The FDS traces every requirement back to its originating PRD Epic/story. Runs in two origins — interview (greenfield elicitation) or reverse-engineer (brownfield), which reconstructs a draft PRD/FDS from the existing codebase and the open/closed issue backlog, tags every reconstructed item with calibrated [Confidence: Level] and source provenance, then runs a confirmation-only interview over the gaps and low-confidence rows so an inherited system with no formal PRD/FDS can be converged with product over time. Creates these artifacts from scratch or amends existing ones in place, preserving stable IDs and a revision history.
dependencies:
  - interview-me
  - agent-markup
  - resolve-repository-platform
argument-hint: "[prd | fds | full] [create | amend] [interview | reverse-engineer]  # stream(s), mode (auto-detected from existing files), origin (default interview)"
---
Two-stream pipeline: product stream (PRD — why/what) + functional stream (FDS — how it must behave). FDS seeded from and traces back to PRD.

Origins:
- `interview` (default): elicit from human, blank-slate, one question at a time.
- `reverse-engineer`: for inherited systems with no PRD/FDS. Agent reconstructs draft from codebase (blueprint when present) + open/closed issue backlog, tags every item `[Confidence: Level]` + source, then confirmation-only interview scoped to gaps + low-confidence rows.

1. PHASE 0 (Stream, Mode, Origin): Resolve stream(s) — `prd`, `fds`, or `full` (default). `full` = PRD first, then carries Epics/stories into FDS. `fds` standalone allowed if PRD exists; else announce no product traceability. Mode per stream: target document exists → `amend` (Amendment Protocol); else `create`. Never blind-overwrite. Resolve origin — `interview` (default) or `reverse-engineer`. When inherited/unfamiliar codebase with no clear human → recommend `reverse-engineer`.
2. PHASE 1 (Context Discovery / Evidence Reconstruction):
   - `interview`: scan project briefs, issue templates, existing PRDs/FDS, codebases for domain context.
   - `reverse-engineer`: run Reconstruction Protocol — ingest evidence sources, reconstruct provisional Epic Register, Story Backlog, FDS requirement set. Tag every row `[Confidence: Level]` + provenance. Draft = input to confirmation interviews.
3. PHASE 2 (PRD Elicitation / Confirmation): Run `interview-me` on PRD Interview Branches. Extract problem, personas, goals, Epics, user stories with acceptance criteria + `[Priority: MoSCoW]`. Skip when stream = `fds`.
   - `reverse-engineer`: do NOT re-elicit from scratch. Present reconstructed draft; confirmation pass scoped to `[Confidence: Possible]` rows, `Unknown — requires verification` fields, conflicting items. Frame as "I inferred X from <source>; confirm, correct, or flag". On confirmation, upgrade `[Confidence: Level]`, replace provenance with confirming authority.
4. PHASE 3 (PRD Document): Compile into PRD schema at `docs/requirements/product-requirements.md`. Assign stable `EPIC-###` / `STORY-###`.
5. PHASE 4 (FDS Elicitation / Confirmation): Run `interview-me` bypassing narrative fluff — extract fine-grained functional rules, cross-field validations, data transformations, `[Auth: Scope]`, exception states. Walk PRD stories one Epic at a time so every behavior maps to a story. Skip when stream = `prd`.
   - `reverse-engineer`: confirmation pass scoped to `[Confidence: Possible]` rows, `Unknown`, untraced behaviors. Validation/authorization/errors from code = `Confirmed|Probable`; issue-text-only = `Possible` — must confirm.
6. PHASE 5 (FDS Document): Compile into FDS schema at `docs/requirements/functional-requirements.md`. Populate `Source (PRD)` on every row.

Directives:
- Ask ONE question at a time. Honour `move-next` advancement contract.
- Stream Sequencing (`full`): do NOT begin FDS until PRD written + IDs assigned (traceability anchors).
- Output Location: PRD → `docs/requirements/product-requirements.md`. FDS → `docs/requirements/functional-requirements.md`.
- Traceability: every FDS row references at least one `STORY-###` (or `EPIC-###`). No product origin → `[Inferred: Unverified]`.
- Detail Threshold: reject vague responses ("save data securely", "nice dashboard"). PRD: force measurable metrics, explicit personas, testable acceptance criteria. FDS: force validation limits, precise `[Auth: Scope]`, error states.
- Reverse-Engineer Calibration: every reconstructed row carries `[Confidence: Level]` + provenance (e.g. `src/OrderService.cs:88`, `issue #231 (closed)`, `blueprint §2.2`). Confirmed = directly read from code/config; Probable = strong code indicator missing one link; Possible = inferred from issue text/naming/heuristics — requires verification, never asserted. Unevidenced = `Unknown — requires verification`. No product origin = `[Inferred: Unverified]`.
- Export-clean Markdown.
- Diagrams: Valid Mermaid.js markdown ONLY. One diagram topic per diagram — never conflate (e.g. no mixing state diagram with sequence diagram, flowchart with ERD, etc.). Place each diagram in the section it supports.

Amendment Protocol:
- Diff-Scoped: load existing document, confirm changed areas, interview ONLY those areas. Carry unchanged sections verbatim.
- Append-Only IDs: NEVER renumber/reuse. New = next free ID. Removed/abandoned → `Status: Deprecated` (PRD: also `[Priority: Wont]`).
- Cross-Stream Drift: when PRD Epic/story amended/deprecated, flag every FDS requirement tracing to it for review. Resolve in same session.
- Revision History: append row (date, summary, affected IDs). Bump version. Write to same canonical path. Git = version store.

Reverse-Engineer Reconstruction Protocol:
Evidence Sources (ingest before drafting):
1. Code structure — prefer existing blueprint at `docs/architecture/system-blueprint.md`. Absent → recommend `analyze-a-codebase`; proceed against direct code scan only if user declines, tag rows ≤ `Probable`.
2. Issue backlog — resolve platform via `resolve-repository-platform`, read BOTH open + closed Work Items. Closed/merged = evidence of delivered behavior (`Probable`); open = evidence of intended behavior (`Possible`). Never treat issue title as confirmed behavior.

Steps:
1. Derive candidate Epics from Module/capability clusters in blueprint + recurring backlog themes.
2. Derive candidate stories from closed feature issues + observable user flows; write `As a/I want/So that` with acceptance criteria reconstructed from evidence, tagged confidence + provenance.
3. Derive candidate FDS requirements from validations, auth checks, state transitions, error handling in code; map to candidate `STORY-###`, or `[Inferred: Unverified]` where no product origin.
4. Record every conflict (code vs. issue) as explicit confirmation item.
Output: complete draft PRD/FDS where proportion of `Possible`/`Unknown` = visible measure of remaining product confirmation. Burn down across sessions via `amend` mode.

PRD Interview Branches:
- Problem & Business Intent: pain solved, commercial leverage, why now.
- Target Personas & Jobs-to-be-Done: who, goals, `[Auth: Scope]`.
- Goals & Success Metrics: measurable outcomes, KPIs.
- Epic Decomposition: major capability themes grouping stories.
- User Stories & Acceptance Criteria: `As a/I want/So that`, testable criteria, `[Priority: MoSCoW]`.
- Scope Boundaries: out-of-scope, assumptions, dependencies, release milestones.

PRD Schema:

# Product Requirements Document (PRD)

## 0. Document Control & Revision History
| Version | Date | Change Summary | Affected IDs |
| :--- | :--- | :--- | :--- |

## 1. Product Vision & Problem Statement
* **Problem Statement:** pain being solved, why matters now.
* **Core Business Intent:** primary commercial leverage.
* **Target Personas:** | Persona / Role | Job-to-be-Done | `[Auth: Scope]` | Primary Value Delivered |

## 2. Goals & Success Metrics
| Goal | Success Metric / KPI | Baseline | Target |
| :--- | :--- | :--- | :--- |

## 3. Epic Register
| Epic ID | Epic Title | Outcome / Definition of Done | `[Priority: MoSCoW]` | Status |
| :--- | :--- | :--- | :--- | :--- |

## 4. User Story Backlog
| Story ID | Epic ID | User Story (`As a/I want/So that`) | Acceptance Criteria | `[Priority: MoSCoW]` | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |

## 5. Scope Boundaries & Assumptions
* **Out of Scope:** record `[Priority: Wont]` decisions.
* **Assumptions & Dependencies:** external services, teams, conditions.
* **Release Milestones:** Epic sequencing across releases.

FDS Schema:

# Functional Design Specification (FDS)

## 0. Document Control & Revision History
| Version | Date | Change Summary | Affected IDs |
| :--- | :--- | :--- | :--- |

## 1. Document Governance & System Scope
* **Core Business Intent:** why system exists, primary commercial leverage.
* **Actor & Persona Matrix:** | Persona / Role | System Access Level | `[Auth: Scope]` | Operational Boundary |

## 2. Granular Functional Requirements
| Feature ID | Source (PRD) | Requirement / Capability | Target Module / View | Policy (`[Policy]`) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |

## 3. Data Invariants & Domain Validation Rules
* **State Transition Logic:** valid state pathways + mutation conditions.
* **Validation Schema Matrix:** | Context / Entity | Field | Type / Constraints | Failure Mode / Exception Rule |

## 4. Operational Boundaries & Security Profiles
* **Data Isolation Model:** multi-tenancy boundaries + storage access rules.
* **Data Sensitivity Register:** | Entity / Data Element | Sensitivity (`[Data: Classification]`) | Lawful Basis / Handling Note |
* **SLA & Performance Baselines:** execution limits (UI responsiveness, batch windows).