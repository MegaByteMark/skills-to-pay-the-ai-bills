---
name: create-epic
description: Renders ONE epic work item from a single PRD Epic Register entry (EPIC-###), enriched with the traced FDS technical contract, and creates or amends it in the resolved workflow tracking system. PRD-primary, FDS-enriched. Idempotent via an embedded stable-ID marker so re-runs amend in place rather than duplicate. The single-epic leaf sequenced by the seed-backlog orchestrator; also invocable directly for one epic.
dependencies:
  - resolve-repository-platform
  - agent-markup
  - gather-requirements
  - interview-me
argument-hint: "<EPIC-###> [create | amend]  # which epic; mode auto-detected from tracker when omitted"
user-invocable: true
---
Single-epic leaf. Takes ONE `EPIC-###`, gathers PRD + FDS trace, renders, writes to tracker. Set-level concerns → `seed-backlog`.

1. PHASE 0 (Input & Mode): Resolve target `EPIC-###` from argument (required). Mode: tracker work item already carries marker → `amend`; else `create`. Never blind-create duplicate.
2. PHASE 1 (Source Ingestion): Load PRD at `docs/requirements/product-requirements.md` — Epic Register row + §1 vision/intent + §5 scope boundaries. Load FDS at `docs/requirements/functional-requirements.md` — select every requirement whose `Source (PRD)` traces to this epic or its stories. FDS absent → proceed PRD-only, mark FDS-sourced sections `[Inferred: Unverified]`.
3. PHASE 2 (Platform): Run `resolve-repository-platform` BEFORE any tracker command. Carry Resolution Record into output header.
4. PHASE 3 (Render): Compile into Epic Output Schema per Source-to-Section Map. Append footer marker.
5. PHASE 4 (Preview & Write): Present rendered epic + intended action (create|amend, platform). Require explicit confirmation. On confirm, create/amend via adapter row. No authenticated CLI → emit Markdown. Report work-item reference.

Directives:
- PRD-Primary, FDS-Enriched: PRD decides existence, title, scope, priority. FDS supplies Technical Contract + E2E Definition of Done. Never let FDS invent scope PRD doesn't justify.
- Ambiguity Escalation: underspecified/contradictory section → (1) `interview-me` for specific gap; (2) if answered, render + note captured interactively; (3) if gap cannot close → recommend `gather-requirements` `amend` mode, HALT. When invoked by `seed-backlog`, report gap to orchestrator instead of interviewing mid-batch.
- Stable-ID: embed `EPIC-###` footer marker. Match by marker, NEVER title.
- Amend, Don't Clobber: existing work item is baseline — update changed sections, preserve marker, do not reset unrelated fields.
- Honest Provenance: no persisted FDS → `[Inferred: Unverified]`. `[Priority: Wont]`/`Status: Deprecated` → do NOT create — defer to `seed-backlog` deprecation handling.
- Write-Side Safety: no tracker before resolution; no mutation before confirmation.

Source-to-Section Map:
| Epic section | Primary source |
| :--- | :--- |
| Title | PRD Epic Register — Epic Title |
| Context · Goal | PRD §1 Business Intent + Epic Register Outcome |
| Context · Target State | PRD Epic Register Outcome / Definition of Done |
| Scope · In-Scope | PRD stories under this epic (deliverables) |
| Scope · Out-Scope | PRD §5 Out of Scope relevant to this epic |
| Technical Contracts · Schema Lock-in | FDS §3 Data Invariants / Validation traced here |
| Technical Contracts · Dependencies | PRD §5 Assumptions & Dependencies + FDS |
| Definition of Done · E2E Criteria | FDS §2 requirements traced here + story acceptance criteria |
| Definition of Done · Handoff | Docs merge must update |

Schema:

# Epic: [Title]  `[Priority: MoSCoW]`

## 1. Context
- **Goal:** [Business / user value]
- **Target State:** [Expected system behaviour once done]

## 2. Scope Boundaries
- **In-Scope:** [Specific deliverables]
- **Out-Scope:** [Explicit exclusions]

## 3. Technical Contracts
- **Schema Lock-in:** [Data-model / persistence contract. `[Inferred: Unverified]` if no FDS]
- **Dependencies:** [External blockers / services / other epics]

## 4. Definition of Done
- **E2E Criteria:** [Required functional flows from traced FDS requirements]
- **Handoff:** [README / Interface docs to update on Change Proposal merge]

<!-- skills:work-item kind=epic id=EPIC-### source=PRD -->