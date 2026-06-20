---
name: analyze-a-codebase
description: Ingest a repository, identify architecture drift, evaluate technical dependencies against rolling EOL horizons, and produce a rigid, text-based system blueprint delivered via interactive section-by-section checkpoints.
dependencies:
  - design-vocab        # Architectural taxonomy contract (Modules, Seams, etc.)
  - agent-markup        # Machine-readable token contract ([Risk: Level], [Policy], etc.)
  - gather-requirements # Enforces upfront FDS requirement extraction via interview-me
---

Operational Workflow:
1. PHASE 1 (Upfront Contract Verification & Guardrail Gate): Check for the existence of `docs/requirements/functional-requirements.md`.
     - IF the file does not exist or is empty: Hand off execution immediately to the `gather-requirements` skill to drive the technical interview and generate the FDS contract.
     - ELSE: Proceed directly to Phase 2, utilizing the FDS as the absolute behavioral baseline.
2. PHASE 2 (Ingestion & Delta Analysis): Map the physical repository structures, boundaries, and dependencies. Actively extract library, framework, and language versions from manifest files (e.g., package.json, .csproj, go.mod, Gemfile, Prisma schemas) and independently evaluate their official vendor support timelines and EOL (End of Life) statuses relative to the current calendar year.
3. PHASE 3 (Deterministic Output via Incremental Checkpoints): Output the final blueprint strictly matching the "Rationalized Schema Structure" below. You must deliver this blueprint ONE major section at a time. After rendering a section, PAUSE execution, explain your key findings/clarifications, and ask the user to verify. Do not proceed to generate the next section until the current section is approved or corrected.

[Operational Directives]
- Output Location Contract: Upon final approval of all checkpoints, the entire consolidated blueprint must be written to or overwritten at exactly `docs/architecture/system-blueprint.md` relative to the repository root.
- Vocabulary Compliance: You must strictly adhere to the taxonomy defined in the `design-vocab` skill. All structural elements must be described using Module, Interface, Implementation, Depth, Seam, and Adapter. The terms component, service, unit, API, signature, and boundary are explicitly prohibited.
- Markup Compliance: You must strictly restrict values inside square bracket tokens to the enumerations allowed by the `agent-markup` skill.
- No Narrative Fluff: Keep text minimal, punchy, and highly structured.
- Table-First Design: Use Markdown tables for profiles, registries, dictionaries, and matrices.
- Visuals: Use valid Mermaid.js code blocks exclusively for all diagrams.

================================================================================
[Rationalized Schema Structure & Checkpoint Sequence]
================================================================================

### SECTION 1 CHECKPOINT: System Overview & Governance Profile
#### 1.1 Core Intent & Persona Registry
| Persona / Role | System Owner / Contact | `[Auth: Scope]` | SLA / Support Tier |

---

### SECTION 2 CHECKPOINT: Structural Architecture & Code Mapping
#### 2.1 Technology Stack & Platform Targets
#### 2.2 Physical Repository Mapping (Mermaid Directory Graph)
#### 2.3 Module Dependency, View Transitions & Seam Topology (Mermaid System Topology)
*(Note: This topology must explicitly map execution lifecycles, user-interface view-to-view navigation paths, and downstream module invocation chains.)*

---

### SECTION 3 CHECKPOINT: Lifecycle & Ecosystem Matrix
#### 3.1 Automated Tech Stack Lifecycle & EOL Registry
*(Note: Evaluated independently by the agent against industry horizons for the current calendar year)*
| Module / Library | Discovered Version | Target Platform | Industry Support Status | Upgrade Risk (`[Risk: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

#### 3.2 Solution Ecosystem & Companion Dependencies Map
| System / Companion App | Seam / Relationship | Integration Vector | Shared Assets / State |

---

### SECTION 4 CHECKPOINT: DevOps & Operational Governance
#### 4.1 Development Workflow & Delivery Pipeline Matrix
| Phase | Tooling / Platform | Workflow Rule (`[Policy]`) | Verification Gates |

#### 4.2 Knowledge & Incident Infrastructure
| Resource Type | Location / Target | Update Policy | SLA Breach Protocol |

---

### SECTION 5 CHECKPOINT: Data Layer & Security Schemas
#### 5.1 Data Dictionary & Schema Definitions
#### 5.2 Multi-Tenancy & Data Isolation Model

---

### SECTION 6 CHECKPOINT: Test Surface Architecture Blueprint
#### 6.1 Target Test Surface Mapping
*(Note: Map the optimal surfaces for verification based on module depth. Identify the highest-leverage interfaces and seams where testing provides maximum capability per unit of test code, explicitly specifying where mock/fake adapters should be injected.)*

---

### SECTION 7 CHECKPOINT: Strategic Architectural Recommendation
#### 7.1 Discovered Architectural Pattern & Target Evolution
*(Note: Identify and name the dominant architectural style present in the codebase. Evaluate its alignment with the principles of depth, leverage, locality, and testability. If structural friction is discovered, provide an expert recommendation for an architectural pattern that would explicitly benefit the project's maintenance and lifecycle goals.)*
