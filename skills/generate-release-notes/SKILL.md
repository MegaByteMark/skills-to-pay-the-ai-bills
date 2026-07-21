---
name: generate-release-notes
description: Analyzes git commits, code deltas, and merged Change Proposal (pull/merge request) discussions between a user-specified historical point and the current HEAD to generate a high-density, shorthand-categorized release note snapshot.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - design-vocab
  - interview-me
  - resolve-repository-platform
---
1. PHASE 1 (Baseline): Query starting comparison baseline (SHA, date, or "last release tag"). Ambiguous → `interview-me` ONE question.
2. PHASE 2 (Git Diff): Diff from baseline to HEAD. Extract commit messages + file deltas mapping altered Modules/Interfaces/docs.
3. PHASE 3 (Change Proposal Discussion): Resolve platform via `resolve-repository-platform`. For every merged Change Proposal in window, ingest title, body, Review Discussion via resolved CLI. Mine for intent, breaking changes, migration steps, known limitations, reviewer caveats. No authenticated CLI / no remote → skip silently, proceed on commits alone.
4. PHASE 4 (Synthesis): Bucket into strict prefix categories. Hyper-condensed bulleted layout per schema.

Directives:
- Defacto Escape Hatch: If 100% of diff is routine patches, lint corrections, or test additions with zero structural modifications → entire output: `"Bug fixes and performance improvements."`
- Category Tokens (every bullet prefixed):
  - `[feat]`: New capabilities, business logic, user-facing enhancements.
  - `[fix]`: Bug resolutions, regression fixes, broken Interface patches.
  - `[refactor]`: Structural changes not altering functionality (e.g. decoupling a Seam).
  - `[perf]`: Speed, efficiency, VRAM, memory optimizations.
  - `[docs]`: Technical specs, FDS, guides, blueprint modifications.
  - `[test]`: Test suite additions/modifications/refactoring.
  - `[chore]`: Routine maintenance, dependency bumps, build tooling, minor CI/CD.
- No raw SHAs, author handles, raw file paths in output. Summarize at capability/Module boundary.
- Change Proposal discussion is intent evidence, not verbatim. Distill to shorthand bullets — never quote reviewers or attribute remarks. Breaking change/migration/limitation revealed only by discussion takes precedence over Escape Hatch.
- Conciseness: ≤15 words per bullet, declarative present tense ("Adds…", "Fixes…", "Removes…"). No second sentence, no sub-bullets, no parentheticals.
- One-Bullet-Per-Change: Collapse related commits + follow-ups + remarks for single change into one bullet.
- Tone Neutralization: Strip sentiment, blame, humor, profanity, hedging, codenames. Read as single professional release manager voice.
- Empty Category Suppression: Render only tokens with a real change. Drop empty headers entirely.
- Anti-Fabrication: Every bullet traces to concrete commit/delta/merged Change Proposal. Never announce unshipped work.

Schema:

### Release Notes Snapshot (`[Scope: Release]`)

#### Application Changes
* `[feat]` [capability]
* `[fix]` [resolved defect]
* `[refactor]` [structural cleanup]
* `[perf]` [optimization]

#### Stability & Documentation
* `[docs]` [documentation update]
* `[test]` [test coverage]
* `[chore]` [maintenance]