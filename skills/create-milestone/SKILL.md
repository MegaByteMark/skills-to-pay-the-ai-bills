---
name: create-milestone
description: Renders ONE release-aligned milestone work item from a single PO Release-Alignment plan entry (MS-###), enriched with the grouped work-item references and target date, and creates, amends, or closes it in the resolved workflow tracking system. PRD-primary, plan-enriched. Idempotent via an embedded stable-ID marker so re-runs amend in place rather than duplicate. The single-milestone leaf sequenced by the po persona; also invocable directly for one milestone.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - resolve-repository-platform
  - agent-markup
  - interview-me
argument-hint: "<MS-###> [create | amend | close]  # which milestone; mode auto-detected from tracker when omitted"
user-invocable: true
---
Single-milestone leaf. Takes ONE `MS-###`, gathers the PO Release-Alignment plan entry + assigned work-item refs, renders, writes to tracker. Set-level concerns → `po`.

1. PHASE 0 (Input & Mode): Resolve target `MS-###` from argument (required). Mode: tracker milestone already carries marker → `amend`; explicit `close` arg → `close`; else `create`. Never blind-create duplicate.
2. PHASE 1 (Source Ingestion): Load the PO Release-Alignment plan entry from the persistent state store (path supplied via clean-context payload). Pull assigned work-item references (epics, stories, bug refs) from the plan entry. Direct invocation without a plan entry → HALT and route to `po plan-release-milestones`.
3. PHASE 2 (Platform): Run `resolve-repository-platform` BEFORE any tracker command. Carry Resolution Record into output header.
4. PHASE 3 (Validation):
   - `create` mode: every assigned work-item ref must resolve on the tracker. Unresolved → HALT with the missing list; route back to PHASE 4 of the parent `po` run so epics/stories are created first.
   - `amend` mode: at least one tracked work-item ref required; removals only with a justification comment.
   - `close` mode: every assigned work item must be closed or explicitly justified as carried-over. Carried-over → emit a `<!-- skills:milestone-carryover -->` block on each carried item.
5. PHASE 4 (Render): Compile into Milestone Output Schema per Source-to-Section Map. Append footer marker.
6. PHASE 5 (Preview & Write): Present rendered milestone + intended action (create|amend|close, platform). Require explicit confirmation. On confirm, create/amend/close via adapter row. No authenticated CLI → emit portable Markdown. Report milestone reference.

Directives:
- Milestone ≠ Epic: milestones group work for a release target; epics group work for a feature/theme. The leaf never re-uses `epic` labels or titles on a milestone. The footer marker distinguishes: `kind=milestone` vs `kind=epic`.
- PRD-Primary: the PO Release-Alignment plan entry is authoritative for scope-in, scope-out, and target date. Tracker-only proposals without plan provenance → HALT; route back to `po`.
- Stable-ID: embed `MS-###` footer marker. Match by marker, NEVER title.
- Amend, Don't Clobber: existing milestone is baseline — update changed sections, preserve marker, do not reset unrelated fields.
- Close, Don't Delete: closing a milestone retains history; deletion is forbidden.
- Write-Side Safety: no tracker before resolution; no mutation before confirmation.
- No empty milestones: every `create` MUST reference at least one work item. Empty milestone = HALT.
- Reference validation: in `create` and `amend` modes, every assigned work-item ref must resolve on the tracker. Unresolved refs are a HALT, not a warning.

Source-to-Section Map:
| Milestone section | Primary source |
| :--- | :--- |
| Title | PO plan entry — Milestone Title |
| Description · Scope-In | PO plan entry — assigned work-item references (epics/stories/bugs) |
| Description · Scope-Out | PO plan entry — explicit exclusions |
| Due Date | PO plan entry — target date (developer-supplied) |
| Release Intent | PO plan entry — one-line release narrative |

Schema:

# Milestone: [Title]

## 1. Scope
- **In-Scope:** [Work-item references grouped by epic, each with `EPIC-###` / `STORY-###` / `BUG-###` marker]
- **Out-Scope:** [Explicit exclusions]

## 2. Target
- **Due Date:** [YYYY-MM-DD]
- **Release Intent:** [One-line release narrative from the PO plan entry]

<!-- skills:work-item kind=milestone id=MS-### source=PRD -->