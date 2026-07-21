---
name: audit-test-coverage
description: Discovers physical test files, evaluates actual coverage patterns against the target test surface blueprint and the FDS contract, and flags brittle tests or untested deep interfaces.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - design-vocab
  - agent-markup
  - interview-me
  - detect-test-harness
---
1. PHASE 1 (Blueprint & FDS Gate): Read `docs/architecture/system-blueprint.md` + `docs/requirements/functional-requirements.md`.
   - Missing → `interview-me` ONE decision:
     - GENERATE: hand off to `gather-requirements`/`analyze-a-codebase`, then proceed.
     - EPHEMERAL: reconstruct minimalist FDS in-context (from README, issues, change proposals, user) WITHOUT saving. Mark all output `[Inferred: Unverified]`, down-weight `[Confidence: Level]`. Allowed because requirement-verification can be sourced independently of code; blueprint-derived test-surface targets are reduced fidelity.
     - ABORT.
   - ELSE: Extract blueprint §6 "Test Surface Architecture Blueprint" + FDS §2 requirements registry.
2. PHASE 2 (Test Suite Ingestion): Resolve harness via `detect-test-harness` (framework, run command, layout, naming) before scanning — do NOT assume. Scan physical repo to discover all test suites/files/mock setups. Map which Modules/Interfaces/requirements are exercised. Carry harness Resolution Record into audit header.
3. PHASE 3 (Gap Synthesis): Compare reality against blueprint targets + Minimum Verification Surface Baseline. Flag where verification fails to cover deep Modules, binds to shallow Impls, leaves core requirements unverified, or omits mandatory floor surface. Conditional surface whose trigger is ABSENT = deliberately excluded, NOT a gap.

Minimum Verification Surface Baseline:
Mandatory Floor (always expected):
- M1 — Deep-Interface Isolation: every high-Depth Module's Interface verified in isolation, real Adapters at Seams replaced by fake/mock Adapters (typically `tests/unit`).
- M2 — Critical-Seam Integration: highest-`[Risk: Level]` Seams (datastore, external Adapter) exercised against real/realistic Adapter (typically `tests/integration`).
- M3 — Primary-Flow Verification: each primary user flow from blueprint §2.3.1 walked through full Seam chain — at minimum auth + value-exchange ("money") paths (typically `tests/e2e`).
- M4 — Requirement Verification Floor: at least one verification per FDS requirement classified Critical or High.
- M5 — Regression Lock: pinning verification for each previously-fixed defect discoverable from history/issue tracker.

Conditional Surfaces (ONLY when trigger fires; else "Excluded — trigger absent"):
- C1 — Contract Verification: ≥2 independently-deployed Modules share Interface across network Seam.
- C2 — Property/Invariant Verification: Module with broad input domain or pure algorithmic Depth that example-based cases under-sample.
- C3 — Concurrency/Ordering Verification: flow flagged async/event-driven/saga in blueprint §2.3.4.
- C4 — Performance/Load Verification: Interface declaring performance trait or SLA.
- C5 — Malformed-Input/Fuzz Verification: public-facing parse/deserialization Seam. Deep security fuzzing → `audit-security-and-governance`.
- C6 — Snapshot/Visual/Accessibility Verification: presentation-tier Modules emitting user-facing output.

Directives:
- Strict `design-vocab`: Interface verification vs. Implementation coupling. Avoid unit/component test unless naming physical folder path.
- Table-First. Tag every finding `[Confidence: Level]`.
- Minimum, Not Maximum: flag absent Mandatory Floor + triggered Conditional only. Trigger-absent Conditional = deliberately excluded, never open gap.
- No Narrative Fluff.

Schema:

### 1. Interface & Seam Verification Alignment Matrix
| Target Deep Interface / Seam | Discovered Test Suite | Match Status | Brittle Test Risk (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 2. Functional Requirement Verification Matrix
| Feature ID | Requirement | Test Verification Status | Validation Integrity (`[Policy]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 3. Minimum Verification Surface Compliance Matrix
| Baseline Tier | Trigger Status (Conditional) | Target Interface / Seam / Flow | Coverage (Met/Partial/Absent) | Enforcement (`[Policy]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- | :--- |

### 4. Implementation Coupling & Structural Deficits
- **Untested Deep Interfaces:** core Modules with high behavioral complexity lacking Interface-level verification.
- **Shallow / Brittle Test Coverage:** tests coupled to volatile Impls rather than stable Interfaces.
- **Adapter / Mock Deficiencies:** missing or incorrect fake Adapters at defined Seams.