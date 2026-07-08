---
name: competency-profile
description: Shared contract for the human's competency baseline — the per-user, out-of-tree record of demonstrated skill per area. Defines canonical storage, schema, and read/merge protocol. Consumed by teach-a-skill, teach-me, and vibe-code-antidote.
dependencies:
  - agent-markup
---
OWNS: per-area human competency. DOES NOT OWN: per-project codebase comprehension (`vibe-code-antidote`); course progress (`teach-me`).

STORAGE:
- OS PATH RESOLUTION — resolve `${XDG_STATE_HOME:-$HOME/.local/state}` to platform path:
  - **Linux:** `~/.local/state/`
  - **macOS:** `~/Library/Application Support/`
  - **Windows:** `%LOCALAPPDATA%`
- Canonical: `{resolved-base}/ai-skills/competency-profile.md`
- Discovery: read canonical path; if exists, use it. If absent = cold start — do NOT hunt elsewhere.
- Read == write. Always write back to canonical path. Cold start → create (make parent dir).
- One file, NOT per-project. Chat-only runtimes: hold in memory + paste-back snapshot — never workspace file, never volatile temp.

SCHEMA (ever-expanding, never drop rows):
```markdown
# Competency Baseline
**Updated:** [ISO date]

| Area | Competency | Confidence | Source | Date |
| :-- | :-- | :-- | :-- | :-- |
| TS async/await | Solo | Confirmed | antidote | 06-26 |
| SQL joins | Not-Ready | Probable | teach-me | 06-20 |
```
- Competency: Solo | Guided | Paired | Not-Ready
- Confidence: Confirmed (observed work) | Probable (mixed) | Possible (self-report only)
- Source: `antidote` (= vibe-code-antidote) | `teach-me` | `teach-a-skill`
- Date: MM-DD

READ/MERGE:
- Read at start. Confirmed rows are truth — do not re-interrogate.
- Observed work outranks self-report. Promote only on corroborating observed work; self-report capped at Possible.
- Update, don't clobber: refresh row on new evidence; never blank another skill's evidence.
- Conflict: more recent observed work wins.
- Regression allowed: downgrade on observed misunderstanding with new evidence.
- Every write records source + date.