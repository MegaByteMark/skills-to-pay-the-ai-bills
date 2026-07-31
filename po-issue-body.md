## Description

**Objective:** Build the PO persona orchestrator skill that bundles PO-specific reasoning patterns and handles requirements-to-backlog orchestration with intelligent gap analysis and ticket lifecycle management.

**Context:** 
This is the third persona skill in the orchestration pattern. PO is a hands-off orchestrator (like SWE) that spawns subagents for ticket creation/amendment, but adds gap analysis logic to prevent ticket duplication and maintain requirements coherence across the issue tracker.

**Scope:**
- Persona skill name: `po`
- Bundled skills: `agent-markup`, `design-vocab`, `resolve-repository-platform`
- Subagent spawning skills: `create-epic`, `create-user-story`, `create-bug`
- Orchestration flow: 
  1. Developer invokes `/po <PRD/FDS reference>` with access to current issue tracker
  2. PO analyzes PRD/FDS against existing tickets (gap analysis)
  3. PO spawns `create-epic`, `create-user-story`, `create-bug` subagents (clean context) only for:
     - New requirements not yet tracked
     - Existing tickets requiring amendment to match current spec
  4. PO maintains issue tracker structure: categorizations, release milestone cadence, interdependencies
  5. Developer reviews proposed changes (new/amended tickets), approves or requests adjustments
  6. Flow completes with issue tracker in sync with requirements
- Output: Issue tracker is the source of truth, synced to PRD/FDS with zero orphaned or duplicated tickets

**Dependencies:**
- All bundled skills must be discoverable and loadable
- `create-epic` — leaf skill to raise/amend one epic
- `create-user-story` — leaf skill to raise/amend one user story as child of epic
- `create-bug` — leaf skill to raise/amend one bug report
- `resolve-repository-platform` to detect GitHub/GitLab before platform-specific ticket operations
- `agent-markup` for consistent tagging (`[Confidence: Level]`, `[Risk: Level]`)
- `design-vocab` for architectural consistency across ticket descriptions
- Expects PRD/FDS artifact from BA or `gather-requirements` skill as input reference

---

## Acceptance Criteria

- [ ] **Skill definition:** `po` SKILL.md exists with frontmatter (`name`, `description`, `dependencies`, `user-invocable: true`)
- [ ] **Skill loads with context:** Developer invokes `/po <PRD/FDS reference>` and persona loads with bundled skills available
- [ ] **Gap analysis:** PO reads current PRD/FDS, scans existing issue tracker, identifies:
  - Requirements not yet tracked (new tickets needed)
  - Tracked tickets no longer in requirements (candidates for closure/archival)
  - Existing tickets requiring amendment to match spec changes
- [ ] **Subagent spawning:** PO spawns `create-epic`, `create-user-story`, `create-bug` subagents with **clean context only** (requirement text + platform adapter, no intermediate reasoning)
- [ ] **Smart ticket creation:** Subagents create/amend tickets only for actual gaps; no duplicates or orphaned work
- [ ] **Issue tracker structure maintained:**
  - Categorizations (labels, assignees) preserved across amendments
  - Release milestone cadence respected (no tickets moved to wrong milestone without justification)
  - Issue interdependencies tracked and updated (parent/child, blocks/blocked-by relationships)
- [ ] **Developer decision loop:** Developer can review proposed changes:
  - Approve all changes and proceed
  - Request re-ranking, re-categorization, or re-amendment (loops back to gap analysis)
  - Accept current state and close flow
- [ ] **Context persistence:** Requirements and ticket artifacts persist via issue tracker / PRD docs, not agent-to-agent handoff
- [ ] **No skill drift:** Bundled skills used consistently; no ad-hoc skill loading mid-session
- [ ] **Clean context window for subagents:** Create-epic/story/bug subagents receive only: requirement text, platform-specific formatting rules, reference links—zero bleed from parent agent state
- [ ] **Coherence reporting:** PO surfaces any conflicts (e.g., "Ticket #42 violates milestone cadence" or "Story #15 is orphaned—no parent epic in current PRD") before presenting to developer

---

## Notes

- This establishes the pattern for hands-off orchestrator personas with gap analysis (like SWE but adds reconciliation logic)
- Persona skill should *not* create tickets directly; subagents keep ticket-raising isolated
- PO is the reconciliation layer between requirements and tracked work—prevents chaos in the backlog
- Success here unblocks QA and Writer personas using similar orchestration + subagent patterns
