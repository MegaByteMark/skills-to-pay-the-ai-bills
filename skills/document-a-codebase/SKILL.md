---
name: document-a-codebase
description: Ingests the Functional Design Specification (FDS), system blueprint, and physical codebase to generate consistent, highly structured user, technical, or installation documentation based on a specified token archetype.
dependencies:
  - design-vocab
  - agent-markup
  - interview-me
---
1. PHASE 1 (Contract Gate): Check `docs/requirements/functional-requirements.md` AND `docs/architecture/system-blueprint.md`.
   - Missing → trigger `interview-me` for ONE decision:
     - GENERATE: hand off to `gather-requirements`/`analyze-a-codebase`, then proceed.
     - ABORT: stop. Ephemeral tier deliberately NOT offered — shipping docs built on non-existent spec is worse than none.
   - ELSE: Ingest FDS + blueprint.
2. PHASE 2 (Target Context): Identify token archetype (`[Doc: QuickStart]`, `[Doc: Technical]`, `[Doc: Troubleshooting]`, `[Doc: Installation]`).
   - `[Doc: Installation]`: scan for deployment assets (Dockerfiles, Terraform, cloud configs). If multiple valid paths or ambiguous hosting → `interview-me` ONE clarification. Honour `move-next` advancement.
3. PHASE 3 (Generate): Write pristine, table-heavy documentation to the domain path.

Directives:
- Archetype Paths:
  - `[Doc: QuickStart]` → `docs/guides/quick-start.md`
  - `[Doc: Technical]` → `docs/guides/technical-reference.md`
  - `[Doc: Troubleshooting]` → `docs/guides/troubleshooting-runbook.md`
  - `[Doc: Installation]` → `docs/guides/installation-guide.md`
- Strict `design-vocab`: Module, Interface, Implementation, Seam, Adapter. Prohibited: service, component.
- Table-First: scannable matrices, not conversational prose.
- Diagrams: Valid Mermaid.js markdown ONLY. One diagram topic per diagram — never conflate (e.g. no mixing sequence diagram with deployment diagram, class diagram with data flow, etc.).

Archetypes:

### [Doc: Installation]
- Prerequisites Registry: version matrices of runtimes, DB engines, tooling.
- Environment Configuration Map: env vars, secret keys, network bindings.
- Deployment Sequence Matrix:
  | Step | Target Environment | CLI Command | Expected Output / Success Indicator | Failure Protocol |

### [Doc: Technical]
- System Module Initialization: bootup patterns mapping Modules to Seams.
- Data Flow Tracing: tables tracking input data past Interfaces to storage Impls per FDS validation rules.

### [Doc: Troubleshooting]
- Symptom & Resolution Matrix:
  | Error / Exception State | Target Module / Seam | Root Cause Vector | Resolution Protocol | Urgent SLA Breach Action |