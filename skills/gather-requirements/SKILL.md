---
name: gather-requirements
description: Conducts an exhaustive, granular engineering discovery by executing the interview-me skill across two streams — a product stream that produces a Product Requirements Document (PRD) of vision, personas, Epics and prioritised user stories that seed the backlog, and a functional stream that produces a complete Functional Design Specification (FDS) of behavioral truth. The FDS traces every requirement back to its originating PRD Epic/story.
dependencies:
  - interview-me  # Drives the single-question interactive conversation loop and context extraction mechanics
  - agent-markup  # Governs [Auth: Scope], [Policy], [Data: Classification], and [Priority: MoSCoW] token enumerations
argument-hint: "[prd | fds | full]  # which stream(s) to run; defaults to full (PRD then FDS)"
---

This skill runs requirements discovery as a two-stream pipeline that mirrors how modern agile teams work: the **product stream** establishes *why* and *what* (the PRD that feeds Epics and backlog stories), and the **functional stream** establishes *how it must behave* (the FDS that downstream auditing agents consume). The functional stream is seeded from, and traces back to, the product stream.

Operational Workflow:
1. PHASE 0 (Stream Selection): Determine which stream(s) to run from the invocation argument — `prd`, `fds`, or `full` (default). `full` runs the PRD stream first, then carries its Epics/stories forward as the seed for the FDS stream. Running `fds` standalone is permitted when a PRD already exists at the output path; if none exists, announce that the FDS will proceed without product traceability.
2. PHASE 1 (Context Discovery): Scan any provided project briefs, issue templates, existing PRDs/FDS, or codebases to establish initial domain context before interviewing.
3. PHASE 2 (PRD Elicitation Interview): Trigger and execute the `interview-me` skill directed at product-discovery branches (see "PRD Interview Branches"). Extract the problem, target personas, measurable goals, Epics, and granular user stories with acceptance criteria and `[Priority: MoSCoW]` ranking. Skip when the stream selector is `fds`.
4. PHASE 3 (PRD Document Generation): Compile the verified product requirements into the PRD, matching the "PRD Markdown Schema" below, written to the permanent requirements directory. Assign stable `EPIC-###` and `STORY-###` identifiers.
5. PHASE 4 (FDS Elicitation Interview): Trigger and execute the `interview-me` skill again, directed to bypass high-level narrative fluff and explicitly extract fine-grained functional rules, cross-field validations, data transformations, security boundaries, and exception states. When a PRD exists, walk its stories one Epic at a time so every behavior maps to a story. Skip when the stream selector is `prd`.
6. PHASE 5 (FDS Document Generation): Compile the verified requirements into the FDS, matching the "FDS Markdown Schema" below. Populate the Source (PRD) traceability column on every requirement row.

[Operational Directives]
- Execution Protocol: You must use the `interview-me` skill mechanics for every interview phase. Ask exactly ONE highly specific question at a time to prevent human cognitive fatigue. Honor the `interview-me` advancement contract: never move to the next question until the user issues the literal `move-next` command — answering or asking follow-ups does not authorize advancing.
- Stream Sequencing: In `full` mode, do NOT begin the FDS stream until the PRD is written and its Epic/story IDs are assigned, because those IDs are the FDS traceability anchors.
- Output Location Contract: The PRD must be written to exactly `docs/requirements/product-requirements.md` and the FDS to exactly `docs/requirements/functional-requirements.md`, both relative to the repository root.
- Traceability Contract: Every FDS requirement row must reference at least one originating `STORY-###` (or `EPIC-###` where a requirement is cross-cutting) in its Source (PRD) column. If a behavior has no product origin, flag it inline as `[Inferred: Unverified]` so the gap is visible rather than silently invented.
- Detail Threshold: Reject open-ended or vague human responses (e.g., "The form should save data securely" or "users want a nice dashboard"). For the PRD, force measurable success metrics, explicit personas, and testable acceptance criteria. For the FDS, force validation limits, precise authorization scopes, and error states.
- Structural Layout: Each generated document must rigorously match its schema below and remain export-clean Markdown per the `agent-markup` Output Portability Convention.

PRD Interview Branches (product stream):
  - Problem & Business Intent: the pain being solved, the commercial leverage, and why now.
  - Target Personas & Jobs-to-be-Done: who uses it, their goals, and their authentication scope (`[Auth: Scope]`).
  - Goals & Success Metrics: measurable outcomes and the KPIs that prove success or failure.
  - Epic Decomposition: the major capability themes that group stories.
  - User Stories & Acceptance Criteria: `As a / I want / So that` statements with testable acceptance criteria and `[Priority: MoSCoW]` ranking.
  - Scope Boundaries: explicit out-of-scope items, assumptions, external dependencies, and release milestones.

================================================================================
[PRD Markdown Schema]
================================================================================

# Product Requirements Document (PRD)

## 1. Product Vision & Problem Statement
* **Problem Statement:** The user/business pain being solved and why it matters now.
* **Core Business Intent:** High-density summary of the product's primary commercial leverage.
* **Target Personas:** | Persona / Role | Job-to-be-Done | System Access Level (`[Auth: Scope]`) | Primary Value Delivered |

## 2. Goals & Success Metrics
| Goal | Success Metric / KPI | Baseline | Target |
| :--- | :--- | :--- | :--- |

## 3. Epic Register
*(Epics are the capability themes that group user stories and structure the backlog.)*
| Epic ID | Epic Title | Outcome / Definition of Done | `[Priority: MoSCoW]` |
| :--- | :--- | :--- | :--- |
| *EPIC-001* | *Streaming Ingestion* | *Operators can ingest large feeds without local serialization.* | *Must* |

## 4. User Story Backlog
*(This registry is the direct seed for the FDS functional stream and the team's backlog.)*
| Story ID | Epic ID | User Story (`As a / I want / So that`) | Acceptance Criteria | `[Priority: MoSCoW]` |
| :--- | :--- | :--- | :--- | :--- |
| *STORY-001* | *EPIC-001* | *As an operator, I want to ingest 50MB feeds so that I can process partner data live.* | *Given a 50MB feed, when ingested, then no local disk serialization occurs and throughput stays within SLA.* | *Must* |

## 5. Scope Boundaries & Assumptions
* **Out of Scope:** Items explicitly excluded this cycle (record `[Priority: Wont]` decisions here rather than dropping them silently).
* **Assumptions & Dependencies:** External services, teams, or conditions the plan relies on.
* **Release Milestones:** High-level sequencing of Epics across releases.

================================================================================
[FDS Markdown Schema]
================================================================================

# Functional Design Specification (FDS)

## 1. Document Governance & System Scope
* **Core Business Intent:** High-density summary of why this system exists and its primary commercial leverage.
* **Actor & Persona Matrix:** | Persona / Role | System Access Level | `[Auth: Scope]` | Operational Boundary / Responsibility |

## 2. Granular Functional Requirements (The Audit Contract)
*(Note: This structured registry is the direct interface consumed by downstream automated auditing agents. The Source (PRD) column traces each behavior to the product intent that justifies it.)*
| Feature ID | Source (PRD) | Functional Requirement / Technical Capability | Target Module / View | Policy Stance (`[Policy]`) |
| :--- | :--- | :--- | :--- | :--- |
| *REQ-001* | *STORY-001* | *Must process concurrent data streams up to 50MB without local disk serialization.* | *IngestionAdapter* | *Enforced* |

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
