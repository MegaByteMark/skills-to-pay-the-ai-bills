---
name: competency-profile
description: Shared contract for the human's competency baseline — the per-user, out-of-tree record of demonstrated skill per area. Defines the canonical storage location, schema, and read/merge protocol so any skill that teaches, assesses, or hands work to a human reads and writes the SAME baseline. Consumed by teach-a-skill, teach-me, and vibe-code-antidote so calibration is continuous across skills and sessions and never duplicated or committed to a project.
dependencies:
  - agent-markup  # [Competency: Level] (the baseline enumeration) and [Confidence: Level] (calibration certainty)
---

# Purpose
The human's skill in an area (e.g. "TypeScript async/await", "SQL joins") is a fact about the *person*, not about a project or a skill. It must be recorded once and shared, so a workday under `vibe-code-antidote` and an evening under `teach-me` read and update the same baseline. This skill owns that record. It is a contract, not an interactive experience — consuming skills invoke it; the user does not.

# Scope (own this, nothing else)
- **OWNS:** per-area human competency baseline — `[Competency: Level]`, calibration `[Confidence: Level]`, the evidence behind each, and provenance (which skill last observed it, when).
- **DOES NOT OWN:** per-project codebase comprehension (stays with `vibe-code-antidote`); course/syllabus progress (stays with `teach-me`, per language). These reference the baseline but persist separately. Never write project- or course-specific state into the shared baseline.

# Storage — per-user, global, NEVER in the workspace
The baseline is one record per user/machine, shared across every project and skill. It MUST NOT live in any project tree, be committed, or appear in `git status`.

**Canonical location — two candidate paths, strict priority order:**
1. `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/competency-profile.md`
2. `${TMPDIR:-/tmp}/ai-skills/competency-profile.md`

**Deterministic discovery (do exactly this — never improvise a wider search).** Check path 1, then path 2; use the FIRST that exists and STOP. If NEITHER exists it is a cold start — there is no profile, so do NOT go hunting in home, the workspace, or other directories. State the outcome (which path, or cold start) once on first touch.

**Read == write (this is what prevents the read/write split).** Always write back to the exact path you read from. On a cold start, create path 1 (make its parent dir); only if that location is unwritable, fall to path 2. If you began from path 2 but path 1 later becomes writable, migrate the file up to path 1 and note it once.

One file, NOT namespaced per project (the whole point is cross-project continuity). If the runtime is chat-only with no writable out-of-tree location, hold the baseline in agent memory and emit it as a paste-back snapshot on pause — never substitute a workspace file. Treat temp storage as best-effort; if absent, the consumer falls back to fresh assessment.

# Schema
```markdown
# Competency Baseline
**Owner:** [user/handle or machine] | **Updated:** [ISO date]

## Areas
| Area | [Competency: Level] | [Confidence: Level] | Evidence (observed work) | Last source | Updated |
| :-- | :-- | :-- | :-- | :-- | :-- |
| TypeScript async/await | Solo | Confirmed | wrote retry wrapper unaided | vibe-code-antidote | 2026-06-26 |
| SQL joins | Not-Ready | Probable | could not explain a JOIN | teach-me | 2026-06-20 |
```
Use stable, granular area names (language/domain + concept) so different skills resolve to the same row. When unsure whether an area already exists, match on concept rather than minting a near-duplicate row.

# Read / Merge Protocol
Consuming skills MUST follow this so concurrent and sequential updates stay coherent:
- **Read at start.** Before assessing or teaching, read the baseline and treat existing rows as the starting truth — do not re-interrogate areas already recorded with `[Confidence: Confirmed]`.
- **Observed work outranks self-report.** Only promote an area's `[Competency: Level]` on corroborating *observed* work; a verbal claim is provisional and recorded at most `[Confidence: Possible]`.
- **Update, don't clobber.** When updating a row, write the new level/confidence, refresh evidence + source + date; never blank another skill's evidence.
- **Conflict resolution.** On disagreement, the row backed by the more recent *observed work* wins. A higher `[Confidence: Level]` from older observed work is only overridden by newer observed work, not by self-report.
- **Regression is allowed and expected.** If a skill observes a misunderstanding of a previously higher-rated area, downgrade it (with new evidence). Skill atrophy is real; the baseline must be able to fall, not only rise.
- **Provenance is mandatory.** Every write records which skill observed it and when, so consumers can weigh staleness.

# Compliance
- Use the `[Competency: Level]` and `[Confidence: Level]` enumerations from `agent-markup` exactly; introduce no parallel rubric.
- This skill defines storage + schema + merge rules only. It performs no teaching, no syllabus building, and no project analysis — those belong to its consumers.
