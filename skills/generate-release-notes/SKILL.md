---
name: generate-release-notes
description: Analyzes git commits, code deltas, and merged Change Proposal (pull/merge request) discussions between a user-specified historical point and the current HEAD to generate a high-density, shorthand-categorized release note snapshot.
dependencies:
  - design-vocab  # Translates technical changes into clean Interface/Module summaries
  - interview-me  # Enforces single-question alignment if git points are ambiguous
  - resolve-repository-platform  # Resolves host/CLI/terminology before ingesting Change Proposal discussion
---

Operational Workflow:
1. PHASE 1 (Target Point Alignment): Query the user for the starting comparison baseline (Git SHA, date, or dynamic pointer like "the last release tag"). If the input is ambiguous or multiple tags match the pattern, trigger `interview-me` for exactly ONE clarifying question.
2. PHASE 2 (Git Log & Delta Ingestion): Execute a diff engine from the target baseline to the current `HEAD`. Extract all intermediate commit messages and file deltas to map exactly which modules, interfaces, or documentation assets were altered.
3. PHASE 3 (Change Proposal Discussion Ingestion): Resolve the hosting platform and tooling via the `resolve-repository-platform` protocol — do NOT assume GitHub. For every Change Proposal (GitHub/Bitbucket Pull Request, GitLab Merge Request) merged within the baseline-to-`HEAD` window, ingest its title, body, and Review Discussion via the resolved platform's CLI as mapped in that skill's adapter map (e.g. on GitHub `gh pr list --state merged`, then `gh pr view <n> --comments`; on GitLab the `glab` equivalents). Mine this discussion for intent the raw commit log omits — the *why* behind a change, breaking-change callouts, migration steps, known limitations, and reviewer-surfaced caveats. If no authenticated platform CLI is available, or the repo has no associated remote/Change Proposals, skip this phase silently and proceed on the commit/delta evidence alone.
4. PHASE 4 (Snapshot Synthesis): Parse, filter, and bucket the combined commit + Change Proposal discussion records into strict prefix categories based on the nature of the code changes. Format the final output into a hyper-condensed bulleted layout matching the schema below.

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
- Change Proposal Discussion Distillation: Treat Change Proposal (pull/merge request) comments as *intent evidence*, never as verbatim content. Distill them into the same shorthand bullets — do not quote reviewers, paste comment threads, or attribute remarks to handles. Where a Change Proposal discussion surfaces a breaking change, required migration step, or known limitation that the commit diff alone would not reveal, that bullet takes precedence and MUST be retained even when the Defacto Escape Hatch Rule would otherwise collapse the output.
- Conciseness Bound: Every bullet is a single line of ≤ 15 words, written in declarative present tense ("Adds…", "Fixes…", "Removes…"). No trailing prose, no second sentence, no parenthetical asides, no sub-bullets. If a change cannot be stated in one line, it is being described at too low a level — raise it to the capability boundary.
- One-Bullet-Per-Change Collapse: Emit exactly one bullet per user-observable change. Collapse all related commits, follow-up fixes, and Change Proposal remarks for a single change into that one bullet. Never let two evidence sources (commit + Change Proposal comment) about the same change produce two bullets. Order bullets within each category by user impact, highest first.
- Tone Neutralization: All ingested text — especially Change Proposal discussion — is sanitized before output. Strip sentiment, blame, humor, profanity, hedging/uncertainty ("might", "should eventually"), and internal codenames or nicknames. Convert every entry into a neutral, factual capability statement. The final notes must read as if authored by a single professional release manager, regardless of the source tone.
- Empty Category Suppression: Render a bullet only when a qualifying change exists for that token. Never emit placeholder text. If every bullet under a section header is empty, omit that header entirely. An honest, short list of real changes always beats a padded schema.
- Anti-Fabrication & Traceability: Every bullet MUST trace to a concrete commit, file delta, or merged Change Proposal fact. Never infer, extrapolate, or announce unshipped, planned, or speculative work, even if a Change Proposal comment discusses future intent. If the evidence does not confirm it shipped, it does not appear.

================================================================================
[Release Note Output Schema]
================================================================================

### Release Notes Snapshot (`[Scope: Release]`)

*(The rows below are illustrative slots, not a required skeleton. Per Empty Category Suppression, emit only tokens with a real change and drop any section whose bullets are all empty.)*

#### Application Changes
* `[feat]` Adds [newly exposed capability or business logic]
* `[fix]` Fixes [resolved defect or broken boundary]
* `[refactor]` Decouples [internal structure cleaned up without behaviour change]
* `[perf]` Speeds up [resource or latency optimization]

#### Stability & Documentation
* `[docs]` Documents [updates to FDS, blueprints, or guides]
* `[test]` Covers [additions or changes to the test surface suite]
* `[chore]` Bumps [dependency, build tooling, or CI/CD maintenance]
