---
name: generate-release-notes
description: Analyzes git commits and code deltas between a user-specified historical point and the current HEAD to generate a high-density, shorthand-categorized release note snapshot.
dependencies:
  - design-vocab  # Translates technical changes into clean Interface/Module summaries
  - interview-me  # Enforces single-question alignment if git points are ambiguous
---

Operational Workflow:
1. PHASE 1 (Target Point Alignment): Query the user for the starting comparison baseline (Git SHA, date, or dynamic pointer like "the last release tag"). If the input is ambiguous or multiple tags match the pattern, trigger `interview-me` for exactly ONE clarifying question.
2. PHASE 2 (Git Log & Delta Ingestion): Execute a diff engine from the target baseline to the current `HEAD`. Extract all intermediate commit messages and file deltas to map exactly which modules, interfaces, or documentation assets were altered.
3. PHASE 3 (Snapshot Synthesis): Parse, filter, and bucket the commit records into strict prefix categories based on the nature of the code changes. Format the final output into a hyper-condensed bulleted layout matching the schema below.

[Operational Directives]
- Defacto Escape Hatch Rule: Evaluate the parsed commit collection. If 100% of the diff contains exclusively routine patches, minor lint corrections, or test additions with zero structural modifications, bypass detailed summaries entirely and default the entire output to:
  `"Bug fixes and performance improvements."`
- Shorthand Categorization Enforcements: Every bullet point generated must be prefixed using the following strict architectural tokens:
    - `[feat]`: New functional capabilities, business logic, or user-facing enhancements.
    - `[fix]`: Bug resolutions, regression fixes, and broken interface patches.
    - `[refactor]`: Structural code changes that do not alter functionality (e.g., decoupling a seam).
    - `[perf]`: Code optimizations focused on speed, efficiency, VRAM utilization, or memory management.
    - `[docs]`: Modifications to technical specs, FDS documents, user guides, or the system blueprint.
    - `[test]`: Additions, modifications, or refactoring of the physical test suites.
    - `[chore]`: Routine maintenance tasks, dependency version bumps, build tooling, or minor CI/CD adjustments.
- No Technical Inversions: Do not leak raw commit SHAs, author handles, or raw file paths into the final output. Summarize changes at the capability or module boundary layer.

================================================================================
[Release Note Output Schema]
================================================================================

### Release Notes Snapshot (`[Scope: Release]`)

#### Application Changes
* `[feat]` [Short description of newly exposed capability or business logic]
* `[fix]` [Short description of the resolved defect or broken boundary fix]
* `[refactor]` [Short description of internal structural cleanup]
* `[perf]` [Short description of resource or speed optimizations]

#### Stability & Documentation
* `[docs]` [Short description of updates made to FDS, blueprints, or guides]
* `[test]` [Short description of modifications to the test surface suite]
* `[chore]` [Short description of dependency bumps or general tooling maintenance]
