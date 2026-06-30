---
name: audit-test-coverage
description: Discovers physical test files, evaluates actual coverage patterns against the target test surface blueprint and the FDS contract, and flags brittle tests or untested deep interfaces.
dependencies:
  - design-vocab         # Uses Interface, Implementation, Depth, and Seam definitions
  - agent-markup         # Pulls token rules for tagging risk or policy categories
  - interview-me         # Resolves the tiered missing-contract gate with a single question
  - detect-test-harness  # Shared protocol for resolving the test runner/framework + test layout
---

## Operational Workflow
1. PHASE 1 (Blueprint & FDS Ingestion & Tiered Gate): Locate and attempt to read both `docs/architecture/system-blueprint.md` and `docs/requirements/functional-requirements.md`.
     - IF either file does not exist or is empty: do NOT silently abort. Trigger `interview-me` for ONE decision presenting the available tiers:
         - GENERATE (recommended): hand off to `gather-requirements` and/or `analyze-a-codebase` to generate and persist the missing contract(s), then proceed.
         - EPHEMERAL: reconstruct a minimalist FDS baseline in-context (from README, issues, change proposals, or the user) WITHOUT saving it, and proceed. Mark all resulting output `[Inferred: Unverified]` and down-weight `[Confidence: Level]`. Allowed because requirement-verification intent can be sourced independently of the code; note that any blueprint-derived target-test-surface targets are reduced fidelity under this tier.
         - ABORT: stop and inform the user the audit cannot run without a baseline.
     - ELSE: Extract "SECTION 6: Test Surface Architecture Blueprint" from the blueprint and the requirements registry from Section 2 of the FDS to lock in verification targets.
2. PHASE 2 (Test Suite Ingestion): Resolve the test harness via `detect-test-harness` (framework, run command, physical test layout, naming conventions) before scanning — do NOT assume a framework. Then scan the physical repository to discover all test suites, files, and mock setups against that resolved layout. Map out which modules, interfaces, and specific requirement paths are actually exercised by existing tests. Carry the harness Resolution Record into the audit header so remediation inherits it.
3. PHASE 3 (Verification Gap Synthesis): Compare reality against the target blueprints AND the Minimum Verification Surface Baseline below. Produce a structured audit reporting precisely where the implementation fails to verify deep modules, incorrectly binds to shallow implementations, leaves core functional requirements completely unverified, or omits a mandatory floor surface. CRITICAL: a conditional surface whose trigger is ABSENT is NOT a gap — record it as deliberately excluded so remediation never gold-plates it.

## Minimum Verification Surface Baseline
*(The shared floor consumed by `remediate-test-coverage`. This is the single source of truth for "what minimum surfaces every application must verify". Mandatory tiers are `[Policy: Enforced]`; conditional tiers are `[Policy: Advisory]` and become expected ONLY when their stated architectural trigger is present. Coverage maximalism is a defect, not a goal — never flag a trigger-absent surface as missing.)*

Mandatory Floor (expected in every application, no trigger required):
  - M1 — Deep-Interface Isolation Verification: every high-Depth Module's Interface verified in isolation, with real Adapters at its Seams replaced by fake/mock Adapters. (physical home: typically a `tests/unit` path)
  - M2 — Critical-Seam Integration Verification: the highest-`[Risk: Level]` Seams (datastore, external Adapter) exercised against a real or realistic Adapter rather than a fake. (physical home: typically `tests/integration`)
  - M3 — Primary-Flow Verification: each primary user flow from blueprint §2.3.1 walked through its full Seam chain — at minimum the authentication and value-exchange ("money") paths. (physical home: typically `tests/e2e`)
  - M4 — Requirement Verification Floor: at least one verification bound to every FDS requirement classified Critical or High.
  - M5 — Regression Lock: a pinning verification for each previously-fixed defect discoverable from history or the issue tracker.

Conditional Surfaces (expected ONLY when the trigger fires; otherwise record as "Excluded — trigger absent"):
  - C1 — Contract Verification — Trigger: ≥2 independently-deployed Modules share an Interface across a network Seam.
  - C2 — Property/Invariant Verification — Trigger: a Module with a broad input domain or pure algorithmic Depth that example-based cases under-sample.
  - C3 — Concurrency/Ordering Verification — Trigger: a flow flagged asynchronous / event-driven / saga in blueprint §2.3.4.
  - C4 — Performance/Load Verification — Trigger: an Interface declaring a performance trait or SLA in its contract.
  - C5 — Malformed-Input/Fuzz Verification — Trigger: a public-facing parse/deserialization Seam. Deep security fuzzing defers to `audit-security-and-governance`.
  - C6 — Snapshot/Visual/Accessibility Verification — Trigger: presentation-tier Modules emitting user-facing output.

## Operational Directives
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe gaps in terms of Interface verification vs. Implementation coupling. Avoid terms like unit test or component test unless referring to a physical folder path.
- Table-First Reporting: Deliver findings in high-density Markdown tables structured exactly like the schema below.
- Confidence Calibration: Tag every finding with `[Confidence: Level]`. Confirmed = directly observed in the test suite; Probable = strong indicator missing one corroborating link; Possible = heuristic needing human verification (phrase as "requires verification", never asserted).
- Minimum, Not Maximum: Flag absent Mandatory Floor surfaces and triggered Conditional surfaces only. A trigger-absent Conditional surface is reported as deliberately excluded, never as an open gap — this is what stops downstream remediation from over-engineering.
- No Narrative Fluff: Keep output technical, direct, and actionable.

## Test Audit Output Schema

### 1. Interface & Seam Verification Alignment Matrix
| Target Deep Interface / Seam | Discovered Test Suite | Match Status | Brittle Test Risk (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 2. Functional Requirement Verification Matrix
| Feature ID | Functional Requirement / Technical Capability | Test Verification Status | Validation Integrity (`[Policy]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 3. Minimum Verification Surface Compliance Matrix
*(One row per baseline tier. For Conditional tiers, state whether the trigger fired; if not, Status = `Excluded — trigger absent` so remediation knows NOT to build it.)*
| Baseline Tier | Trigger Status (Conditional only) | Target Interface / Seam / Flow | Coverage Status (Met / Partial / Absent) | Enforcement (`[Policy]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- | :--- |

### 4. Implementation Coupling & Structural Deficits
* **Untested Deep Interfaces:** List any core modules with high behavioral complexity that lack interface-level verification.
* **Shallow / Brittle Test Coverage:** Flag any tests directly coupled to volatile internal code bodies (Implementations) rather than stable boundaries (Interfaces), resulting in high maintenance overhead.
* **Adapter / Mock Deficiencies:** Identify missing or incorrectly implemented fake adapters at defined Seams.
