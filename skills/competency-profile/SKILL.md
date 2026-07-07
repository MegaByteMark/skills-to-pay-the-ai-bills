---
name: competency-profile
description: Shared contract for the human's competency baseline — the per-user, out-of-tree record of demonstrated skill per area. Defines the canonical storage location, schema, and read/merge protocol so any skill that teaches, assesses, or hands work to a human reads and writes the SAME baseline. Consumed by teach-a-skill, teach-me, and vibe-code-antidote so calibration is continuous across skills and sessions and never duplicated or committed to a project.
dependencies:
  - agent-markup
---
OWNS: per-area human competency — `[Competency: Level]`, `[Confidence: Level]`, evidence, provenance (which skill, when).

DOES NOT OWN: per-project codebase comprehension (`vibe-code-antidote`); course/syllabus progress (`teach-me`).

Storage: one record per user/machine, across every project/skill. NEVER in workspace, never committed.
- Canonical: `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/competency-profile.md`
- Discovery: read canonical path; if exists, use it. If absent = cold start — do NOT hunt elsewhere. State outcome once.
- Migrate legacy `${TMPDIR:-/tmp}/ai-skills/competency-profile.md` to canonical path on first read.
- Read == write. Always write back to canonical path. Cold start → create (make parent dir).
- One file, NOT per-project. Chat-only runtimes: hold in memory + paste-back snapshot on pause — never workspace file, never volatile temp.

Schema:
```markdown
# Competency Baseline
**Owner:** [user/handle] | **Updated:** [ISO date]

## Areas
| Area | [Competency: Level] | [Confidence: Level] | Evidence (observed work) | Last source | Updated |
| :-- | :-- | :-- | :-- | :-- | :-- |
| TypeScript async/await | Solo | Confirmed | wrote retry wrapper unaided | vibe-code-antidote | 2026-06-26 |
| SQL joins | Not-Ready | Probable | could not explain a JOIN | teach-me | 2026-06-20 |
```
Use stable, granular area names (language/domain + concept) so different skills resolve to same row. Match on concept rather than minting near-duplicates.

Read/Merge Protocol:
- Read at start. Existing `[Confidence: Confirmed]` rows are truth — do not re-interrogate.
- Observed work outranks self-report. Promote level only on corroborating *observed* work; verbal claim = provisional, at most `[Confidence: Possible]`.
- Update, don't clobber: write new level/confidence, refresh evidence + source + date; never blank another skill's evidence.
- Conflict: row backed by more recent *observed work* wins.
- Regression allowed: downgrade on observed misunderstanding with new evidence.
- Provenance mandatory: every write records which skill + when.

Compliance:
- Use `[Competency: Level]` and `[Confidence: Level]` enumerations from `agent-markup` exactly.
- This skill = storage + schema + merge rules only. No teaching, syllabus, or project analysis.