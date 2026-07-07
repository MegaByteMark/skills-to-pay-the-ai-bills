---
name: audit-blueprint-implementation
description: Audits the physical codebase against the established system blueprint and the Functional Design Specification (FDS) to expose missing features, structural implementation gaps, broken seams, and untracked assets.
dependencies:
  - design-vocab
  - agent-markup
  - interview-me
---
1. PHASE 1 (Contract Gate): Check `docs/requirements/functional-requirements.md` AND `docs/architecture/system-blueprint.md`.
   - Missing → trigger `interview-me` for ONE decision:
     - GENERATE: hand off to `gather-requirements`/`analyze-a-codebase`, then proceed.
     - ABORT: stop (cannot audit without baseline).
   - Ephemeral-in-context tier deliberately NOT offered — self-derived baseline from same code would produce meaningless "no drift" results.
   - ELSE: Parse FDS §2 requirements registry and blueprint module dependencies/seam topologies.
2. PHASE 2 (Physical Audit): Scan repository structures, import maps, class relationships. Cross-reference code paths against FDS requirement IDs and blueprint topology rules.
3. PHASE 3 (Gap Synthesis): Detect: code failing to implement FDS requirement, drift from blueprint, untracked assets, encapsulation breaks.

Directives:
- Strict `design-vocab`: describe gaps as Interface leakage, missing Seams, unfulfilled FDS requirements.
- Table-First: high-density Markdown tables per schema below.
- Tag every finding `[Confidence: Level]`. Confirmed = directly observed; Probable = strong indicator missing one link; Possible = requires verification.
- No Narrative Fluff.

Output Schema:

### 1. FDS Requirement vs. Code Implementation Gaps
| Feature ID | Requirement | Expected Module | Implementation Status / Deficit | Risk (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- | :--- |

### 2. Blueprint vs. Structural Drift Matrix
| Discovered Module | Blueprint Target | Gap Type / Violation | Risk (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 3. Interface vs. Implementation Drift & Leaks
- Leaky Interfaces: internal Implementation details leaking through public surface.
- Bypassed Seams: calling code interacts with concrete Impls instead of defined Interface slots.

### 4. Untracked Assets & Security Exposures
- Untracked Modules: physical folders/sub-modules not declared in blueprint/FDS scope.
- Security & Multi-Tenancy Gaps: data isolation rules or `[Auth: Scope]` missing/incorrectly applied.