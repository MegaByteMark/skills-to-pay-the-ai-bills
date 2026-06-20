name: audit-blueprint-implementation
description: Audits the physical codebase against the established system blueprint and the Functional Design Specification (FDS) to expose missing features, structural implementation gaps, broken seams, and untracked assets.
dependencies:
  - design-vocab  # Uses Module, Interface, Implementation, Depth, and Seam definitions
  - agent-markup  # Pulls token rules for tagging risk or policy categories
---

Operational Workflow:
1. PHASE 1 (Contract Ingestion & Guardrail Gate): Check for the existence of both contracts.
     - IF `docs/requirements/functional-requirements.md` OR `docs/architecture/system-blueprint.md` are missing: Abort execution immediately. Inform the user which foundational document is missing, and explicitly prompt them to run `gather-requirements` or `analyze-a-codebase` to establish the baseline contracts first.
     - ELSE: Parse and extract the functional requirements registry from Section 2 of the FDS, and the structural module dependencies/seam topologies from the blueprint.
2. PHASE 2 (Physical Codebase Audit): Scan the physical repository structures, import maps, and class relationships. Cross-reference code paths against the FDS requirement IDs and blueprint topology rules.
3. PHASE 3 (Gap & Drift Synthesis): Detect and classify discrepancies where the actual code fails to implement an FDS requirement, drifts from the blueprint design, contains untracked assets, or breaks encapsulation boundaries.

[Operational Directives]
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe gaps in terms of Interface leakage, missing Seams, or unfulfilled functional requirements.
- Table-First Reporting: Deliver drift findings in high-density Markdown tables structured exactly like the schema below.
- No Narrative Fluff: Keep output technical, direct, and actionable.

================================================================================
[Implementation Gap Audit Output Schema]
================================================================================

### 1. FDS Requirement vs. Code Implementation Gaps
| Feature ID | Functional Requirement / Technical Capability | Expected Target Module | Implementation Status / Deficit | Architectural Urgency (`[Risk: Level]`) |
| :--- | :--- | :--- | :--- | :--- |

### 2. Blueprint vs. Structural Drift Matrix
| Discovered Module | Stated Blueprint Target | Gap Type / Violation | Architectural Urgency (`[Risk: Level]`) |
| :--- | :--- | :--- | :--- |

### 3. Interface vs. Implementation Drift & Leaks
* **Leaky Interfaces:** List instances where internal implementation details leak through a module's public surface area, breaking encapsulation.
* **Bypassed Seams:** Identify areas where calling code interacts directly with concrete implementations instead of decoupling via a defined interface slot.

### 4. Untracked Assets & Security Exposures
* **Untracked Modules:** List any physical folders or significant sub-modules found in the codebase that were not declared in the system blueprint or accounted for in the FDS scope.
* **Security & Multi-Tenancy Gaps:** Highlight any code paths where data isolation rules or authorization scope tokens (`[Auth: Scope]`) are missing or incorrectly applied in practice.
