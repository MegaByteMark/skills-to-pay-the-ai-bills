---
name: seed-backlog
description: Write-side orchestrator that seeds the workflow tracking system from the PRD and FDS produced by gather-requirements. Resolves the platform once, sequences the create-epic and create-user-story leaves across the whole Epic Register and User Story Backlog, wires every story to its parent epic, and is safely re-runnable — on a second pass it reconciles the tracker against amended requirements (create new, update changed, close deprecated) using embedded stable-ID markers, never duplicating. Mirrors gather-requirements on the publish side.
dependencies:
  - resolve-repository-platform  # Resolves platform/tooling ONCE and supplies the Work-Item Authoring map; leaves never re-prompt
  - create-epic                  # Leaf: renders/writes one epic from EPIC-###
  - create-user-story            # Leaf: renders/writes one story from STORY-###, linked to its epic
  - gather-requirements          # Produces the PRD/FDS this orchestrator reads; offered to generate them when absent
  - agent-markup                 # [Priority: MoSCoW], [Confidence: Level], [Inferred: Unverified]
  - interview-me                 # Single-question gate when the PRD is missing
argument-hint: "[epics | stories | full]  # which tiers to seed; full (default) does epics then their stories"
user-invocable: true
---

This is the publish-side mirror of `gather-requirements`: it turns the PRD/FDS backlog into tracked work. It owns the set-level concerns the single-item leaves cannot see — resolving the platform once, deciding create-vs-amend-vs-close for every item, ordering parents before children, and reporting the result. It composes the leaves; it does NOT re-render epics/stories itself.

Operational Workflow:
1. PHASE 1 (Central Gate Resolution): Resolve everything ONCE, up front, so the leaves never re-prompt.
     - Run `resolve-repository-platform` once; carry the resolved platform, Work-Item Authoring row, and Parent Link mechanism (or git-only fallback) into every leaf invocation.
     - Load the PRD at `docs/requirements/product-requirements.md` (required) and the FDS at `docs/requirements/functional-requirements.md` (optional enrichment).
     - IF the PRD is missing: trigger `interview-me` for ONE decision — "Generate the PRD with `gather-requirements` before seeding?" YES hands off to `gather-requirements`; NO halts (there is no backlog to seed).
     - IF the FDS is missing: proceed PRD-only and announce that every epic/story Technical Contract will be marked `[Inferred: Unverified]`.
2. PHASE 2 (Reconciliation Planning): Read the PRD Epic Register and User Story Backlog. Query the tracker for existing work items carrying `skills:work-item` stable-ID markers. For every PRD item classify the action:
     - **Create** — `Status: Active` item with no matching marker in the tracker.
     - **Update** — `Status: Active` item whose tracker work item exists (content may have changed since last run).
     - **Close** — item the PRD now marks `Status: Deprecated` / `[Priority: Wont]` whose tracker work item still exists.
     - **Skip** — deprecated item with no tracker work item (nothing to do).
   Build the full mutation set as a plan. Match strictly by stable-ID marker, never by title.
   While planning, run a **readiness check** over each Active item: flag any whose PRD/FDS source is too thin to render a complete epic/story (missing scope, untestable acceptance criteria, absent technical contract). Collect these as a Gaps list — do NOT silently push thin items.
3. PHASE 3 (Approval Gate): Present the complete plan — counts and per-item action (create/update/close), target platform, AND the Gaps list — and require explicit confirmation before ANY write. This is the single point of consent for the whole run. When gaps exist, offer the user a fork: (a) authorise targeted `interview-me` questions now to close the gaps in-session, or (b) go back around the requirements loop with `gather-requirements` (`amend`) to improve the PRD/FDS first, or (c) proceed and let the affected items render with `[Inferred: Unverified]` sections. Keep ad-hoc interviews at THIS gate, not scattered through the write loop.
4. PHASE 4 (Sequenced Execution): Execute the approved plan in dependency order so parents always exist before children:
     1. Epics first — invoke `create-epic` for each epic to create or amend (skip if the tier selector excludes epics).
     2. Stories next — invoke `create-user-story` for each story, passing the resolved parent epic work item (skip if the tier selector excludes stories).
     3. Closures — close deprecated work items with a deprecation note that cites the originating `EPIC-###` / `STORY-###`; never delete.
   If a leaf reports a blocker (e.g. a parent epic absent in `stories`-only mode), record it and continue the rest of the plan rather than aborting.
5. PHASE 5 (Seed Report): Emit the Backlog Seed Report summarising every action with its resulting work-item reference, so the run is auditable and the next re-run is predictable.

## Operational Directives
- Composition, Not Re-Derivation: This skill sequences the leaves and reconciles the set; it does NOT render epic/story bodies itself — that is the leaves' job. Preserve their output faithfully.
- Idempotent Re-Runs: A re-run after `gather-requirements` amends the PRD/FDS must converge the tracker to match the requirements — create new, update changed, close deprecated — and produce NO duplicates. The stable-ID marker is the sole identity anchor.
- Deprecate, Never Delete: Items dropped from scope are CLOSED with a traceable note, mirroring the PRD's retain-don't-delete discipline. Removed product intent must never leave an orphaned open work item silently behind.
- Parents Before Children: Always create/resolve epics before their stories so every Parent Link can be established. Never orphan a story.
- Single Resolution, Single Consent: Resolve the platform exactly once and obtain ONE approval for the whole mutation set; the leaves must not re-prompt for platform or per-item confirmation during a seeded run.
- Write-Side Safety & Graceful Degradation: Honour `resolve-repository-platform`'s Write-Side Safety directive. If no authenticated CLI is available, stop at the plan and emit the full rendered backlog as portable Markdown instead of pushing — and say so plainly.
- Honest Provenance: Carry `[Inferred: Unverified]` through from the leaves when the FDS is absent; carry each item's `[Priority: MoSCoW]`.
- Gaps Closed at the Gate, Not Invented: Requirement gaps detected in planning are resolved at the approval gate — via targeted `interview-me` or a loop back through `gather-requirements` — never by leaves guessing mid-batch. An unresolved gap leaves its item flagged `[Inferred: Unverified]` or excluded, the user's choice.

## Backlog Seed Report Output Schema

# Backlog Seed Report
**Platform:** [GitHub | GitLab | Bitbucket | Self-hosted | None] (`[Confidence: Level]`)  |  **Tiers:** [epics | stories | full]  |  **FDS:** [present | absent — contracts `[Inferred: Unverified]`]  |  **Mode:** [push | Markdown-only fallback]

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
| *EPIC-001* | *epic* | *Created* | *—* | *<url/number>* | |
| *STORY-001* | *story* | *Updated* | *EPIC-001* | *<url/number>* | |

## Blockers
*(Anything that could not be actioned and why — e.g. a story whose parent epic was excluded by the tier selector. Empty if none.)*
