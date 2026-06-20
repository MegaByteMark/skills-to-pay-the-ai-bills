---
name: gather-requirements
description: Conducts an exhaustive, granular engineering interview by executing the interview-me skill to produce a complete Functional Design Specification (FDS) that serves as the system's behavioral truth.
dependencies:
  - interview-me  # Drives the single-question interactive conversation loop and context extraction mechanics
---

Operational Workflow:
1. PHASE 1 (Context Discovery): Scan any provided project briefs, issue templates, or existing codebases to establish initial domain context.
2. PHASE 2 (FDS Elicitation Interview): Trigger and execute the `interview-me` skill. Direct its core execution loop to bypass high-level narrative fluff and explicitly extract fine-grained functional rules, cross-field validations, data transformations, security boundaries, and exception states.
3. PHASE 3 (FDS Document Generation): Compile the verified requirements into a comprehensive, multi-section Functional Design Specification written directly to the permanent requirements directory.

[Operational Directives]
- Execution Protocol: You must use the `interview-me` skill mechanics for Phase 2. Ask exactly ONE highly specific question at a time to prevent human cognitive fatigue. 
- Output Location Contract: The final generated artifact must be written to exactly `docs/requirements/functional-requirements.md` relative to the repository root.
- Detail Threshold: Reject open-ended or vague human responses (e.g., "The form should save data securely"). Force details on validation limits, precise authorization scopes, and error states.
- Structural Layout: The output document must rigorously match the structure defined in the "FDS Markdown Schema" below.

================================================================================
[FDS Markdown Schema]
================================================================================

# Functional Design Specification (FDS)

## 1. Document Governance & System Scope
* **Core Business Intent:** High-density summary of why this system exists and its primary commercial leverage.
* **Actor & Persona Matrix:** | Persona / Role | System Access Level | `[Auth: Scope]` | Operational Boundary / Responsibility |

## 2. Granular Functional Requirements (The Audit Contract)
*(Note: This structured registry is the direct interface consumed by downstream automated auditing agents.)*
| Feature ID | Functional Requirement / Technical Capability | Target Module / View | Policy Stance (`[Policy]`) |
| :--- | :--- | :--- | :--- |
| *REQ-001* | *Must process concurrent data streams up to 50MB without local disk serialization.* | *IngestionAdapter* | *Enforced* |

## 3. Data Invariants & Domain Validation Rules
* **State Transition Logic:** List valid state pathways and conditions required for mutation.
* **Validation Schema Matrix:**
  | Context / Entity | Field Name | Type / Constraints | Failure Mode / Exception Rule |

## 4. Operational Boundaries & Security Profiles
* **Data Isolation Model:** Explicit rules regarding multi-tenancy boundaries and storage access rules.
* **SLA & Performance Baselines:** High-priority execution limits (e.g., UI responsiveness, batch windows).
