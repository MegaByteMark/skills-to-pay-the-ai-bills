---
name: seed-backlog
description: Write-side orchestrator that seeds the workflow tracking system from the PRD and FDS produced by gather-requirements. Resolves the platform once, sequences the create-epic and create-user-story leaves across the whole Epic Register and User Story Backlog, wires every story to its parent epic, and is safely re-runnable — on a second pass it reconciles the tracker against amended requirements (create new, update changed, close deprecated) using embedded stable-ID markers, never duplicating. Mirrors gather-requirements on the publish side.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - resolve-repository-platform
  - create-epic
  - create-user-story
  - gather-requirements
  - agent-markup
  - interview-me
argument-hint: "[epics | stories | full]  # tiers to seed; full (default) does epics then stories"
user-invocable: true
---
Publish-side mirror of `gather-requirements`. Owns set-level concerns: resolve platform once, decide create/amend/close per item, order parents before children, report result. Composes leaves; does NOT re-render epics/stories.

1. PHASE 1 (Central Gate): Resolve everything ONCE.
   - Run `resolve-repository-platform`; carry resolved platform + Work-Item Authoring row + Parent Link mechanism into every leaf.
   - Load PRD at `docs/requirements/product-requirements.md` (required) and FDS at `docs/requirements/functional-requirements.md` (optional).
   - PRD missing → `interview-me` ONE decision: "Generate PRD with `gather-requirements`?" YES → hand off; NO → halt.
   - FDS missing → proceed PRD-only, announce every epic/story Technical Contract `[Inferred: Unverified]`.
2. PHASE 2 (Reconciliation Planning): Read PRD Epic Register + User Story Backlog. Query tracker for work items carrying `skills:work-item` markers. Classify:
   - Create — `Status: Active`, no matching tracker marker.
   - Update — `Status: Active`, tracker work item exists (content may differ).
   - Close — `Status: Deprecated` / `[Priority: Wont]`, tracker work item exists.
   - Skip — deprecated, no tracker work item.
   Match by marker, NEVER title.
   Run readiness check over Active items: flag too-thin items (missing scope, untestable acceptance criteria, absent technical contract). Collect as Gaps list — do NOT silently push thin items.
3. PHASE 3 (Approval Gate): Present complete plan — counts + per-item action + target platform + Gaps list. Require explicit confirmation before ANY write. Gaps present → offer fork: (a) targeted `interview-me` now, (b) loop back through `gather-requirements` `amend`, or (c) proceed with `[Inferred: Unverified]`. Keep ad-hoc interviews at THIS gate.
4. PHASE 4 (Sequenced Execution):
   1. Epics first — `create-epic` per epic (skip if tier selector excludes).
   2. Stories next — `create-user-story` per story, passing resolved parent epic (skip if tier selector excludes).
   3. Closures — close deprecated items with deprecation note citing `EPIC-###`/`STORY-###`; never delete.
   Leaf reports blocker → record, continue rest of plan.
5. PHASE 5 (Seed Report): Emit Backlog Seed Report summarising every action + work-item reference.

Directives:
- Composition: sequence leaves, reconcile set; never render epic/story bodies.
- Idempotent Re-Runs: converge tracker to match requirements — create new, update changed, close deprecated — no duplicates. Stable-ID marker = sole identity anchor.
- Deprecate, Never Delete: dropped items CLOSED with traceable note. Mirror PRD retain-don't-delete discipline.
- Parents Before Children: always create epics before their stories.
- Single Resolution, Single Consent: resolve platform once; ONE approval for whole mutation set. Leaves must not re-prompt.
- Write-Side Safety: no authenticated CLI → stop at plan, emit full rendered backlog as portable Markdown.
- Honest Provenance: carry `[Inferred: Unverified]` from leaves when FDS absent; carry `[Priority: MoSCoW]`.
- Gaps Closed at Gate: unresolved gap → item flagged `[Inferred: Unverified]` or excluded.

Report Schema:

# Backlog Seed Report
**Platform:** [resolved] (`[Confidence: Level]`) | **Tiers:** [epics | stories | full] | **FDS:** [present | absent — `[Inferred: Unverified]`] | **Mode:** [push | Markdown-only]

## Summary
| Action | Epics | Stories | Total |
| :--- | :--- | :--- | :--- |
| Created | | | |
| Updated | | | |
| Closed (deprecated) | | | |
| Skipped | | | |
| Blocked | | | |

## Detail
| Stable ID | Kind | Action | Parent | Work Item Reference | Note |
| :--- | :--- | :--- | :--- | :--- | :--- |

## Blockers
*(Anything not actioned + why.)*