---
name: document-a-codebase
description: Ingests the Functional Design Specification (FDS), system blueprint, and physical codebase to generate consistent, highly structured user, technical, or installation documentation based on a specified token archetype.
dependencies:
  - design-vocab  # Enforces Interface, Implementation, Module, and Seam terminology
  - agent-markup  # Pulls token rules for tagging operational or risk levels
  - interview-me  # Enforces single-question interaction for hosting ambiguities and gate resolution
---

Operational Workflow:
1. PHASE 1 (Contract Ingestion & Tiered Gate): Check for the existence of both foundational contracts. Documentation must never be generated from an inferred or simulated baseline — shipping a guide built on a non-existent spec is worse than shipping none — so the ephemeral tier is deliberately NOT offered here.
     - IF `docs/requirements/functional-requirements.md` is missing or empty, OR `docs/architecture/system-blueprint.md` is missing or empty: do NOT silently abort. Trigger `interview-me` for ONE decision naming exactly which contract is absent and presenting the available tiers:
         - GENERATE (recommended): hand off to `gather-requirements` (for the FDS) and/or `analyze-a-codebase` (for the blueprint) to generate and persist the missing contract(s), then proceed.
         - ABORT: stop and inform the user that documentation cannot be generated without verified contracts.
     - ELSE: Ingest the complete behavioral context from the FDS and the structural mappings from the system blueprint.
2. PHASE 2 (Target Context & Environment Validation): 
     - Identify the requested token archetype from the user's prompt (`[Doc: QuickStart]`, `[Doc: Technical]`, `[Doc: Troubleshooting]`, or `[Doc: Installation]`).
     - IF `[Doc: Installation]` is selected, scan the repository for deployment assets (e.g., Dockerfiles, Terraform, cloud configs). If multiple valid deployment pathways exist or the target hosting environment is ambiguous, trigger the `interview-me` skill to ask exactly ONE highly specific question to clarify the target infrastructure before proceeding. Honor the `interview-me` advancement contract: keep discussing the clarification and do not proceed to Phase 3 until the user issues the literal `move-next` command.
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