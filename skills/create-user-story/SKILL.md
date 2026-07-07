---
name: create-user-story
description: Renders ONE user story from a single PRD User Story Backlog entry (STORY-###), enriched with its traced FDS functional requirements and validation rules, and creates or amends it in the resolved workflow tracking system as a child of its parent epic work item. PRD-primary, FDS-enriched. Idempotent via an embedded stable-ID marker so re-runs amend in place rather than duplicate. The single-story leaf sequenced by the seed-backlog orchestrator; also invocable directly for one story.
dependencies:
  - resolve-repository-platform
  - agent-markup
  - create-epic
  - gather-requirements
  - interview-me
argument-hint: "<STORY-###> [create | amend]  # which story; mode auto-detected from tracker when omitted"
user-invocable: true
---
Single-story leaf. Takes ONE `STORY-###`, gathers PRD story + FDS behaviour traced to it, renders, writes to tracker as child of parent epic. Set-level concerns → `seed-backlog`.

1. PHASE 0 (Input & Mode): Resolve target `STORY-###` from argument (required). Mode: tracker work item carries marker → `amend`; else `create`. Never blind-create duplicate.
2. PHASE 1 (Source Ingestion): Load PRD — Backlog row for this story (As a/I want/So that, acceptance criteria, `[Priority: MoSCoW]`, parent `EPIC-###`). Load FDS — every requirement whose `Source (PRD)` traces to this story, + §3 validation rules + §4 data-sensitivity entries. FDS absent → PRD-only, mark FDS-sourced sections `[Inferred: Unverified]`.
3. PHASE 2 (Platform & Parent): Run `resolve-repository-platform` for platform + Work-Item Authoring row (including Parent Link). Resolve parent epic work item by `EPIC-###` marker. Parent not yet in tracker → report as blocker (orchestrator creates epics before stories).
4. PHASE 3 (Render): Compile into Story Output Schema. Map acceptance criteria to checkbox items. Populate Technical Contract from traced FDS. Append footer marker.
5. PHASE 4 (Preview & Write): Present rendered story + intended action (create|amend, parent link, platform). Require explicit confirmation. On confirm, create/amend + establish Parent Link via adapter row. No authenticated CLI → emit Markdown. Report work-item reference.

Directives:
- PRD-Primary, FDS-Enriched: PRD owns narrative, acceptance criteria, priority, parent. FDS supplies Technical Contract — Schema changes + verification surface. Never let FDS introduce a story PRD doesn't contain.
- Ambiguity Escalation: underspecified/contradictory → (1) `interview-me` for specific gap; (2) if answered, render + note captured interactively; (3) if cannot close → recommend `gather-requirements` `amend`, HALT. When invoked by `seed-backlog`, report gap to orchestrator.
- Parent-Link Discipline: every story = child of exactly one epic. (Re)establish Parent Link to `EPIC-###` on every run. Never orphan.
- Stable-ID: embed `STORY-###` footer marker. Match by marker, NEVER title.
- Verification Framing: express as Interface-level checks at story's Seams, sourced from traced FDS. Tag each rule `[Policy: Enforced|Advisory|Audit-Only]`. Never invent stack-specific test tooling.
- Honest Provenance: `[Inferred: Unverified]` where no FDS. `[Priority: Wont]`/`Status: Deprecated` → do NOT create.
- Write-Side Safety: no tracker before resolution; no mutation before confirmation.

Schema:

# Story: [Title]  `[Priority: MoSCoW]`

> **As a** [Persona], **I want** [Action], **so that** [Value].

## Acceptance Criteria
- [ ] [Functional requirement — testable]
- [ ] [Functional requirement — testable]

## Technical Contract
- **Schema:** [Data-model / Interface changes. `[Inferred: Unverified]` if no FDS]
- **Verification:** [Interface-level tests at story's Seams, from traced FDS; enforcement `[Policy: Enforced|Advisory|Audit-Only]`]

<!-- skills:work-item kind=story id=STORY-### parent=EPIC-### source=PRD -->