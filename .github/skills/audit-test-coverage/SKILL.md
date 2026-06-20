---
name: audit-test-coverage
description: Discovers physical test files, evaluates actual coverage patterns against the target test surface blueprint, and flags brittle tests or untested deep interfaces.
dependencies:
  - design-vocab  # Uses Interface, Implementation, Depth, and Seam definitions
  - agent-markup  # Pulls token rules for tagging risk or policy categories
---

Operational Workflow:
1. PHASE 1 (Blueprint Ingestion & Guardrail Gate): Locate and attempt to read `docs/architecture/system-blueprint.md`. 
     - IF the file does not exist or is empty: Abort execution immediately. Inform the user that the foundational architecture blueprint is missing, and explicitly prompt them to execute the `analyze-a-codebase` skill first to establish the baseline.
     - ELSE: Extract the "SECTION 6: Test Surface Architecture Blueprint" to lock in the target high-leverage interfaces and seams.
2. PHASE 2 (Test Suite Ingestion): Scan the physical repository to discover all test suites, files, and mock setups. Map out which modules and interfaces are actually exercised by existing tests.
3. PHASE 3 (Verification Gap Synthesis): Compare reality against the target blueprint. Produce a structured audit reporting precisely where the implementation fails to verify deep modules or incorrectly binds to shallow implementations.

[Operational Directives]
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe gaps in terms of Interface verification vs. Implementation coupling. Avoid terms like unit test or component test unless referring to a physical folder path.
- Table-First Reporting: Deliver findings in high-density Markdown tables structured exactly like the schema below.
- No Narrative Fluff: Keep output technical, direct, and actionable.

================================================================================
[Test Audit Output Schema]
================================================================================

### 1. Verification Alignment Matrix
| Target Deep Interface / Seam | Discovered Test Suite | Match Status | Brittle Test Risk (`[Risk: Level]`) |
| :--- | :--- | :--- | :--- |
| *(Example: IOrderProcessing)* | *(Example: OrderProcessingTests.cs)* | `Verified` / `Partial` / `Untested` | `Low` / `Medium` / `High` |

### 2. Implementation Coupling & Structural Deficits
* **Untested Deep Interfaces:** List any core modules with high behavioral complexity that lack interface-level verification.
* **Shallow / Brittle Test Coverage:** Flag any tests directly coupled to volatile internal code bodies (Implementations) rather than stable boundaries (Interfaces), resulting in high maintenance overhead.
* **Adapter / Mock Deficiencies:** Identify missing or incorrectly implemented fake adapters at defined Seams.
