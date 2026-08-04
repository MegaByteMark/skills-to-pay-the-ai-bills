---
name: create-hotfix
description: 'Leaf skill that creates and coordinates ONE gitflow hotfix: creates a hotfix branch from main inside an isolated worktree, applies the target fix, validates build + tests, tags a patch release, and merges back to main and develop with release notes via generate-release-notes. Spawned by the devops persona with clean context (Work Item reference or patch source, main reference, platform); also invocable directly.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.1
user-invocable: true
dependencies:
  - generate-release-notes
  - resolve-repository-platform
  - detect-test-harness
  - interview-me
  - agent-markup
  - design-vocab
  - agent-handoff
argument-hint: "<Work Item reference>  # e.g. 'create-hotfix 42' | 'create-hotfix <commit SHA>'"
---

```mermaid
flowchart TD
    START(["Invoke create-hotfix"]) --> INPUT["Consume [Handoff: Clean]:<br>Work Item ref, main ref,<br>platform"]
    INPUT --> BRANCH["Create hotfix branch<br>from main (worktree)"]
    BRANCH --> FIX{Fix source<br>available?}
    FIX -->|No| ASK["interview-me:<br>locate fix source"]
    ASK --> FIX
    FIX -->|Yes| APPLY["Apply fix<br>in worktree"]
    APPLY --> VALIDATE{Build + tests<br>pass?}
    VALIDATE -->|No| HOLD["HOLD: fix within worktree,<br>re-invoke"]
    VALIDATE -->|Yes| MERGE["Confirm + merge:<br>→ main, → develop;<br>tag patch"]
    MERGE --> NOTES["generate-release-notes:<br>hotfix deltas"]
    NOTES --> DONE(["Done"])
```

### PHASE 1 — Input & Isolation

**Accepts:** `[Handoff: Clean]` from `devops` PHASE 3
Accepted: Work Item reference (or patch source), main reference/SHA, platform resolution.

1. Consume handoff: Work Item reference (or patch source), main reference/SHA, platform resolution.
2. Operate inside the provided isolated worktree — never the developer's working tree. No worktree supplied → create a transient worktree (`git worktree add <path> <main-reference>`, path under OS temp), removed on exit. Branch, fix, tag, and merge operations run only in the worktree.

### PHASE 2 — Hotfix Branch

1. Create `hotfix/<slug>` from the main reference inside the worktree.
2. Apply the fix: cherry-pick the referenced commit, or apply the provided patch, or reproduce it from the Work Item. Fix source absent → `interview-me` ONE question; never fabricate a fix.

### PHASE 3 — Validation

1. Run the canonical build + tests (`detect-test-harness` for the test invocation).
2. Failure → fix within the worktree and re-validate until green or HOLD for the developer. Never merge a failing hotfix.

### PHASE 4 — Tag

Create annotated patch tag on the hotfix tip.

### PHASE 5 — Merge

1. Via the resolved platform, merge hotfix → main (production) and → develop (integration).
2. Present the exact planned mutation set and require explicit confirmation before any write. No authenticated CLI → emit portable Markdown; never silently mutate.

### PHASE 6 — Release Notes

1. Run `generate-release-notes`: baseline = previous release, scope = hotfix deltas.
2. Attach notes to the release.

Directives:
- Worktree-only mutation: never touch the developer's working tree.
- Anti-fabrication: no fix source / no validation run = stated absence, never invented.
- `[Handoff: Clean]`: Work Item ref + main reference + platform only; no parent reasoning. See `agent-handoff`.
- `agent-markup` tokens and `design-vocab` terms only.
