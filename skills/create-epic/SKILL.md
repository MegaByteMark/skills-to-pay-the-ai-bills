---
name: create-epic
description: Renders ONE epic work item from a single PRD Epic Register entry (EPIC-###), enriched with the traced FDS technical contract, and creates or amends it in the resolved workflow tracking system. PRD-primary, FDS-enriched. Idempotent via an embedded stable-ID marker so re-runs amend in place rather than duplicate. The single-epic leaf sequenced by the seed-backlog orchestrator; also invocable directly for one epic.
dependencies:
  - resolve-repository-platform  # Resolves the platform once and supplies the Work-Item Authoring adapter map; no tracker write before it resolves
  - agent-markup  # [Priority: MoSCoW], [Confidence: Level], [Inferred: Unverified], [Policy]
  - gather-requirements  # Produces the PRD/FDS this skill consumes; defines EPIC-### identity and the source schemas; the fallback when a gap cannot be closed by interview
  - interview-me  # Extracts a missing/ambiguous detail from the user before rendering, one question at a time
argument-hint: "<EPIC-###> [create | amend]  # which epic to publish; mode auto-detected from the tracker when omitted"
user-invocable: true
---

This is the single-epic leaf. It takes exactly ONE `EPIC-###`, gathers everything the PRD and FDS trace to it, renders the Epic Output Schema below, and writes it to the resolved tracking system — creating a new work item or amending the existing one in place. Set-level concerns (creating every epic, deprecating removed ones) belong to the `seed-backlog` orchestrator, not here.

Operational Workflow:
1. PHASE 0 (Input & Mode): Resolve the target `EPIC-###` from the argument (required — never guess which epic). Resolve mode: if a tracker work item already carries this epic's stable-ID marker, default to `amend`; otherwise `create`. A user may force either mode. Never blind-create a duplicate.
2. PHASE 1 (Source Ingestion): Load the PRD at `docs/requirements/product-requirements.md` and read the Epic Register row for this `EPIC-###`, plus the §1 vision/business intent and §5 scope boundaries. Load the FDS at `docs/requirements/functional-requirements.md` and select every requirement whose `Source (PRD)` traces to this epic or to a `STORY-###` under it. If the FDS is absent, proceed PRD-only and mark every FDS-sourced section `[Inferred: Unverified]` rather than inventing a contract.
3. PHASE 2 (Platform Resolution): Run `resolve-repository-platform` to obtain the platform and the Work-Item Authoring adapter row BEFORE any tracker command. Carry the Resolution Record into the output header.
4. PHASE 3 (Render): Compile the gathered material into the Epic Output Schema. Map each section to its source per the Source-to-Section Map. Append the stable-ID marker footer.
5. PHASE 4 (Preview & Write): Present the rendered epic and the intended action (create | amend, target platform) and require explicit confirmation before issuing any write. On confirmation, create or amend via the adapter row. If no authenticated CLI is available, emit the rendered Markdown instead of pushing and say so plainly. Report the resulting work-item reference (URL/number).

[Operational Directives]
- PRD-Primary, FDS-Enriched: The PRD decides the epic's existence, title, scope, and priority. The FDS supplies the Technical Contract and the E2E Definition of Done. Never let an FDS detail invent scope the PRD does not justify.
- Ambiguity Escalation, Never Invention: If the PRD/FDS leaves a required section underspecified or contradictory, do NOT guess. Escalate up this ladder: (1) trigger `interview-me` for the specific missing detail, one question at a time; (2) if the user supplies it, render the section and note that the detail was captured interactively (not yet persisted to the PRD/FDS); (3) if the gap cannot be closed, recommend re-running `gather-requirements` in `amend` mode to properly improve the PRD/FDS, and HALT this epic rather than pushing a half-specified work item. When invoked by `seed-backlog`, do not fire ad-hoc interviews mid-batch — report the gap to the orchestrator instead (see that skill's gate).
- Stable-ID Identity: Embed `EPIC-###` as a machine marker in the work-item body (the footer below). Match existing items by that marker, NEVER by title — titles drift, IDs do not. This marker is the contract `seed-backlog` and `amend` mode rely on.
- Amend, Don't Clobber: In `amend` mode the existing work item is the baseline. Update changed sections, preserve the marker, and do not reset unrelated fields (assignees, labels added by humans, discussion).
- Honest Provenance: Tag any section derived without a persisted FDS as `[Inferred: Unverified]`. Carry the epic's `[Priority: MoSCoW]` through; if the PRD marks the epic `[Priority: Wont]` or `Status: Deprecated`, do NOT create it — defer to `seed-backlog`'s deprecation handling.
- Resolve-Before-Invoke & Write-Side Safety: No tracker command before `resolve-repository-platform` resolves; no mutation before user confirmation (per that skill's Write-Side Safety directive).
- Output Portability: The rendered epic is export-clean Markdown per the `agent-markup` Output Portability Convention.

Source-to-Section Map:
| Epic section | Primary source |
| :--- | :--- |
| Title | PRD Epic Register — Epic Title |
| Context · Goal | PRD §1 Core Business Intent + Epic Register Outcome |
| Context · Target State | PRD Epic Register Outcome / Definition of Done |
| Scope · In-Scope | PRD stories under this epic (their deliverables) |
| Scope · Out-Scope | PRD §5 Out of Scope items relevant to this epic |
| Technical Contracts · Schema Lock-in | FDS §3 Data Invariants / Validation traced to this epic |
| Technical Contracts · Dependencies | PRD §5 Assumptions & Dependencies + FDS |
| Definition of Done · E2E Criteria | FDS §2 functional requirements traced here + story acceptance criteria |
| Definition of Done · Handoff | Docs the merge must update (README / Interface docs) |

================================================================================
[Epic Output Schema]
================================================================================

# Epic: [Title]  `[Priority: MoSCoW]`

## 1. Context
- **Goal:** [Business / user value this epic delivers]
- **Target State:** [Expected system behaviour once the epic is done]

## 2. Scope Boundaries
- **In-Scope:** [Specific deliverables — the stories grouped under this epic]
- **Out-Scope:** [Explicit exclusions]

## 3. Technical Contracts
- **Schema Lock-in:** [Data-model / persistence contract that must be fixed — the stable Interfaces dependent work relies on. Mark `[Inferred: Unverified]` if no FDS]
- **Dependencies:** [External blockers / services / other epics]

## 4. Definition of Done
- **E2E Criteria:** [Required functional flows, sourced from the traced FDS requirements]
- **Handoff:** [README / Interface docs to update upon Change Proposal merge]

<!-- skills:work-item kind=epic id=EPIC-### source=PRD -->
