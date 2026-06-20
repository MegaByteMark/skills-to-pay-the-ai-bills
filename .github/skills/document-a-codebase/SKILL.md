---
name: document-a-codebase
description: Ingests the Functional Design Specification (FDS), system blueprint, and physical codebase to generate consistent, highly structured user, technical, or installation documentation based on a specified token archetype.
dependencies:
  - design-vocab  # Enforces Interface, Implementation, Module, and Seam terminology
  - agent-markup  # Pulls token rules for tagging operational or risk levels
  - interview-me  # Enforces single-question interaction for hosting ambiguities
---

Operational Workflow:
1. PHASE 1 (Contract Ingestion & Dual Guardrail Gate): Check for the existence of both foundational contracts.
     - IF `docs/requirements/functional-requirements.md` is missing or empty: Abort execution immediately. Inform the user that the FDS is missing and explicitly prompt them to run the `gather-requirements` skill first.
     - IF `docs/architecture/system-blueprint.md` is missing or empty: Abort execution immediately. Inform the user that the system blueprint is missing and explicitly prompt them to run the `analyze-a-codebase` skill first.
     - ELSE: Ingest the complete behavioral context from the FDS and the structural mappings from the system blueprint.
2. PHASE 2 (Target Context & Environment Validation): 
     - Identify the requested token archetype from the user's prompt (`[Doc: QuickStart]`, `[Doc: Technical]`, `[Doc: Troubleshooting]`, or `[Doc: Installation]`).
     - IF `[Doc: Installation]` is selected, scan the repository for deployment assets (e.g., Dockerfiles, Terraform, cloud configs). If multiple valid deployment pathways exist or the target hosting environment is ambiguous, trigger the `interview-me` skill to ask exactly ONE highly specific question to clarify the target infrastructure before proceeding.
3. PHASE 3 (Deterministic Content Generation): Generate pristine, technical, table-heavy documentation written directly to the appropriate domain path under the `docs/` tree.

[Operational Directives]
- Target Archetype Paths:
    - `[Doc: QuickStart]`: Writes onboarding text to `docs/guides/quick-start.md`
    - `[Doc: Technical]`: Writes core reference maps to `docs/guides/technical-reference.md`
    - `[Doc: Troubleshooting]`: Writes runbooks to `docs/guides/troubleshooting-runbook.md`
    - `[Doc: Installation]`: Writes environment setup details to `docs/guides/installation-guide.md`
- Vocabulary Compliance: Ensure all technical guides strictly adhere to the `design-vocab` taxonomy. Use Module, Interface, Implementation, Seam, and Adapter. Never use prohibited synonyms like service or component.
- Table-First Setup: Step-by-step setup guides and troubleshooting blocks must use high-density, scannable matrices rather than long paragraphs of conversational prose.

================================================================================
[Standard Documentation Matrix Archetypes]
================================================================================

### ARCHETYPE: [Doc: Installation]
* **Prerequisites Registry:** Version matrices of required system runtimes, database engines, and tooling dependencies.
* **Environment Configuration Map:** Structured tables outlining required environment variables, secret keys, and default network bindings.
* **Deployment Sequence Matrix:**
  | Step Number | Target Environment | CLI Execution Command | Expected Output / Success Indicator | Failure Protocol |

### ARCHETYPE: [Doc: Technical]
* **System Component Initialization:** Step-by-step code bootup patterns mapping how Modules hook into their abstract Seams.
* **Data Flow Tracing:** Markdown tables tracking how input data flows past Interfaces down to the concrete storage Layer implementations based on FDS validation rules.

### ARCHETYPE: [Doc: Troubleshooting]
* **Symptom & Resolution Matrix:**
  | Discovered Error / Exception State | Target Module / Seam | Root Cause Vector | Resolution Protocol | Urgent SLA Breach Action |