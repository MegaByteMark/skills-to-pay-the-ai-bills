---
name: document-a-codebase
description: Ingests the Functional Design Specification (FDS), system blueprint, and physical codebase to generate consistent, highly structured user, technical, installation documentation, or inline code commentary based on a specified token archetype.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.1.0
dependencies:
  - commentary
  - design-vocab
  - agent-markup
  - interview-me
---
1. PHASE 1 (Contract Gate): Check `docs/requirements/functional-requirements.md` AND `docs/architecture/system-blueprint.md`.
   - Missing → trigger `interview-me` for ONE decision:
     - GENERATE: hand off to `gather-requirements`/`analyze-a-codebase`, then proceed.
     - ABORT: stop. Ephemeral tier deliberately NOT offered — shipping docs built on non-existent spec is worse than none.
   - ELSE: Ingest FDS + blueprint.
   - `[Doc: Commentary]` exception: if FDS/blueprint missing, proceed without them — commentary can be added from code alone.
2. PHASE 2 (Target Context): Identify token archetype (`[Doc: QuickStart]`, `[Doc: Technical]`, `[Doc: Troubleshooting]`, `[Doc: Installation]`, `[Doc: Commentary]`).
   - `[Doc: Installation]`: scan for deployment assets (Dockerfiles, Terraform, cloud configs). If multiple valid paths or ambiguous hosting → `interview-me` ONE clarification. Honour `move-next` advancement.
3. PHASE 3 (Generate):
   - `[Doc: QuickStart|Technical|Troubleshooting|Installation]`: Write pristine, table-heavy documentation to the domain path.
   - `[Doc: Commentary]`: Load `commentary` skill. Scan codebase. Identify code that warrants commentary per the `commentary` guidelines. Add comments. Never remove existing commentary unless it violates `commentary` prohibited rules.

Directives:
- Archetype Paths:
  - `[Doc: QuickStart]` → `docs/guides/quick-start.md`
  - `[Doc: Technical]` → `docs/guides/technical-reference.md`
  - `[Doc: Troubleshooting]` → `docs/guides/troubleshooting-runbook.md`
  - `[Doc: Installation]` → `docs/guides/installation-guide.md`
  - `[Doc: Commentary]` → inline — no file path. Operates on source code directly.
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