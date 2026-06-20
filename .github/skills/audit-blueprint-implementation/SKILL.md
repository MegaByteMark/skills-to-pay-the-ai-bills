---
name: audit-blueprint-implementation
description: Audits the physical codebase against the established system blueprint to expose structural implementation gaps, broken seams, and untracked assets.
dependencies:
  - design-vocab  # Uses Module, Interface, Implementation, Depth, and Seam definitions
  - agent-markup  # Pulls token rules for tagging risk or policy categories
---

Operational Workflow:
1. PHASE 1 (Blueprint Ingestion & Guardrail Gate): Locate and attempt to read `docs/architecture/system-blueprint.md`.
     - IF the file does not exist or is empty: Abort execution immediately. Inform the user that the foundational system blueprint is missing, and explicitly prompt them to execute the `analyze-a-codebase` skill first to establish the baseline contract.
     - ELSE: Parse and extract the structural schema rules, module dependencies, and seam topologies.
2. PHASE 2 (Physical Codebase Audit): Scan the physical repository structures, import maps, and class relationships. Compare the concrete implementation against the rules and topology documented in the blueprint.
3. PHASE 3 (Gap & Drift Synthesis): Detect and classify discrepancies where the actual code has drifted from the blueprint design, identifying untracked assets or broken encapsulation boundaries.

[Operational Directives]
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe gaps in terms of Interface leakage, missing Seams, or shallow Implementations.
- Table-First Reporting: Deliver drift findings in high-density Markdown tables structured exactly like the schema below.
- No Narrative Fluff: Keep output technical, direct, and actionable.

================================================================================
[Implementation Gap Audit Output Schema]
================================================================================

### 1. Blueprint vs. Implementation Discrepancies Matrix
| Discovered Module | Stated Blueprint Target | Gap Type / Violation | Architectural Urgency (`[Risk: Level]`) |
| :--- | :--- | :--- | :--- |

### 2. Interface vs. Implementation Drift
* **Leaky Interfaces:** List instances where internal implementation details leak through a module's public surface area, breaking encapsulation.
* **Bypassed Seams:** Identify areas where calling code interacts directly with concrete implementations instead of decoupling via a defined interface slot.

### 3. Untracked Assets & Security Exposures
* **Untracked Modules:** List any physical folders, repositories, or significant sub-modules found in the codebase that were not declared or accounted for in the system blueprint.
* **Security & Multi-Tenancy Gaps:** Highlight any code paths where data isolation rules or authorization scope tokens (`[Auth: Scope]`) are missing or incorrectly applied in practice.
