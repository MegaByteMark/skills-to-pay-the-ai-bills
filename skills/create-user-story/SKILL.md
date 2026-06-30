---
name: create-user-story
description: Renders ONE user story from a single PRD User Story Backlog entry (STORY-###), enriched with its traced FDS functional requirements and validation rules, and creates or amends it in the resolved workflow tracking system as a child of its parent epic work item. PRD-primary, FDS-enriched. Idempotent via an embedded stable-ID marker so re-runs amend in place rather than duplicate. The single-story leaf sequenced by the seed-backlog orchestrator; also invocable directly for one story.
dependencies:
  - resolve-repository-platform  # Resolves the platform once and supplies the Work-Item Authoring map, including the Parent Link mechanism; no tracker write before it resolves
  - agent-markup  # [Priority: MoSCoW], [Policy], [Confidence: Level], [Inferred: Unverified], [Data: Classification]
  - create-epic  # Produces the parent epic work item this story links to and shares the stable-ID marker convention
  - gather-requirements  # Produces the PRD/FDS this skill consumes; defines STORY-### identity and the source schemas; the fallback when a gap cannot be closed by interview
  - interview-me  # Extracts a missing/ambiguous detail from the user before rendering, one question at a time
argument-hint: "<STORY-###> [create | amend]  # which story to publish; mode auto-detected from the tracker when omitted"
user-invocable: true
---

This is the single-story leaf. It takes exactly ONE `STORY-###`, gathers the PRD story and the FDS behaviour traced to it, renders the Story Output Schema below, and writes it to the resolved tracking system as a child of its parent epic — creating or amending in place. Set-level concerns belong to the `seed-backlog` orchestrator.

Operational Workflow:
1. PHASE 0 (Input & Mode): Resolve the target `STORY-###` from the argument (required). Resolve mode: if a tracker work item already carries this story's stable-ID marker, default to `amend`; otherwise `create`. Never blind-create a duplicate.
2. PHASE 1 (Source Ingestion): Load the PRD and read the Backlog row for this `STORY-###` — its `As a / I want / So that` statement, acceptance criteria, `[Priority: MoSCoW]`, and parent `EPIC-###`. Load the FDS and select every requirement whose `Source (PRD)` traces to this story, plus the §3 validation rules and §4 data-sensitivity entries those requirements touch. If the FDS is absent, proceed PRD-only and mark FDS-sourced sections `[Inferred: Unverified]`.
3. PHASE 2 (Platform & Parent Resolution): Run `resolve-repository-platform` for the platform and Work-Item Authoring row (including the Parent Link mechanism). Resolve the parent epic's work item by its `EPIC-###` marker. If the parent epic does not yet exist in the tracker, do not invent one — report it as a blocker (the orchestrator creates epics before stories).
4. PHASE 3 (Render): Compile into the Story Output Schema. Map acceptance criteria to checkbox items; populate the Technical Contract from the traced FDS. Append the stable-ID marker footer.
5. PHASE 4 (Preview & Write): Present the rendered story and the intended action (create | amend, parent link, target platform); require explicit confirmation before any write. On confirmation, create or amend and establish the Parent Link via the adapter row. If no authenticated CLI is available, emit Markdown instead and say so. Report the resulting work-item reference.

[Operational Directives]
- PRD-Primary, FDS-Enriched: The PRD owns the story narrative, acceptance criteria, priority, and parent epic. The FDS supplies the Technical Contract — the Schema changes locked in and the verification surface. Never let FDS detail introduce a story the PRD does not contain.
- Ambiguity Escalation, Never Invention: If the PRD/FDS leaves a required section underspecified or contradictory (e.g. untestable acceptance criteria, missing validation rule), do NOT guess. Escalate: (1) trigger `interview-me` for the specific gap, one question at a time; (2) if answered, render and note the detail was captured interactively (not yet persisted); (3) if it cannot be closed, recommend re-running `gather-requirements` in `amend` mode and HALT this story rather than pushing a half-specified work item. When invoked by `seed-backlog`, report the gap to the orchestrator instead of interviewing mid-batch.
- Parent-Link Discipline: Every story is a child of exactly one epic. (Re)establish the Parent Link to the `EPIC-###` work item on every run via the resolved mechanism — never leave a story orphaned.
- Stable-ID Identity: Embed `STORY-###` as a machine marker in the body (footer below). Match existing items by that marker, NEVER by title.
- Verification Framing: Express the verification surface as Interface-level checks at the story's Seams, sourced from the traced FDS requirements. Tag each rule's enforcement stance with `[Policy: Enforced | Advisory | Audit-Only]`. Do not invent stack-specific test tooling.
- Honest Provenance: Tag sections lacking a persisted FDS source `[Inferred: Unverified]`. If the PRD marks the story `[Priority: Wont]` / `Status: Deprecated`, do NOT create it — defer to `seed-backlog`'s deprecation handling.
- Resolve-Before-Invoke & Write-Side Safety: No tracker command before resolution; no mutation before user confirmation.
- Output Portability: Export-clean Markdown per the `agent-markup` Output Portability Convention.

================================================================================
[Story Output Schema]
================================================================================

# Story: [Title]  `[Priority: MoSCoW]`

> **As a** [Persona], **I want** [Action], **so that** [Value].

## Acceptance Criteria
- [ ] [Functional requirement 1 — testable]
- [ ] [Functional requirement 2 — testable]

## Technical Contract
- **Schema:** [Data-model / Interface changes this story locks in (persistence schema, request/response contract). Mark `[Inferred: Unverified]` if no FDS]
- **Verification:** [Interface-level tests at the story's Seams, derived from the traced FDS functional requirements; enforcement stance `[Policy: Enforced | Advisory | Audit-Only]`]

<!-- skills:work-item kind=story id=STORY-### parent=EPIC-### source=PRD -->
