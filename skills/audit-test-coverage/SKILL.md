---
name: audit-test-coverage
description: Discovers physical test files, evaluates actual coverage patterns against the target test surface blueprint and the FDS contract, and flags brittle tests or untested deep interfaces.
dependencies:
  - design-vocab  # Uses Interface, Implementation, Depth, and Seam definitions
  - agent-markup  # Pulls token rules for tagging risk or policy categories
  - interview-me  # Resolves the tiered missing-contract gate with a single question
---

Operational Workflow:
1. PHASE 1 (Blueprint & FDS Ingestion & Tiered Gate): Locate and attempt to read both `docs/architecture/system-blueprint.md` and `docs/requirements/functional-requirements.md`.
     - IF either file does not exist or is empty: do NOT silently abort. Trigger `interview-me` for ONE decision presenting the available tiers:
         - GENERATE (recommended): hand off to `gather-requirements` and/or `analyze-a-codebase` to generate and persist the missing contract(s), then proceed.
         - EPHEMERAL: reconstruct a minimalist FDS baseline in-context (from README, issues, PRs, or the user) WITHOUT saving it, and proceed. Mark all resulting output `[Inferred: Unverified]` and down-weight `[Confidence: Level]`. Allowed because requirement-verification intent can be sourced independently of the code; note that any blueprint-derived target-test-surface targets are reduced fidelity under this tier.
         - ABORT: stop and inform the user the audit cannot run without a baseline.
     - ELSE: Extract "SECTION 6: Test Surface Architecture Blueprint" from the blueprint and the requirements registry from Section 2 of the FDS to lock in verification targets.
2. PHASE 2 (Test Suite Ingestion): Scan the physical repository to discover all test suites, files, and mock setups. Map out which modules, interfaces, and specific requirement paths are actually exercised by existing tests.
3. PHASE 3 (Verification Gap Synthesis): Compare reality against the target blueprints. Produce a structured audit reporting precisely where the implementation fails to verify deep modules, incorrectly binds to shallow implementations, or leaves core functional requirements completely unverified.

[Operational Directives]
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe gaps in terms of Interface verification vs. Implementation coupling. Avoid terms like unit test or component test unless referring to a physical folder path.
- Table-First Reporting: Deliver findings in high-density Markdown tables structured exactly like the schema below.
- Confidence Calibration: Tag every finding with `[Confidence: Level]`. Confirmed = directly observed in the test suite; Probable = strong indicator missing one corroborating link; Possible = heuristic needing human verification (phrase as "requires verification", never asserted).
- No Narrative Fluff: Keep output technical, direct, and actionable.

================================================================================
[Test Audit Output Schema]
================================================================================

### 1. Interface & Seam Verification Alignment Matrix
| Target Deep Interface / Seam | Discovered Test Suite | Match Status | Brittle Test Risk (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 2. Functional Requirement Verification Matrix
| Feature ID | Functional Requirement / Technical Capability | Test Verification Status | Validation Integrity (`[Policy]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 3. Implementation Coupling & Structural Deficits
* **Untested Deep Interfaces:** List any core modules with high behavioral complexity that lack interface-level verification.
* **Shallow / Brittle Test Coverage:** Flag any tests directly coupled to volatile internal code bodies (Implementations) rather than stable boundaries (Interfaces), resulting in high maintenance overhead.
* **Adapter / Mock Deficiencies:** Identify missing or incorrectly implemented fake adapters at defined Seams.
