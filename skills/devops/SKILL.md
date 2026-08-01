---
name: devops
description: 'DevOps persona orchestrator — hands-off gitflow release coordination with full worktree isolation. Loads bundled release/pipeline skills (generate-release-notes, agent-markup, design-vocab, resolve-repository-platform), routes /devops actions (release <version> | hotfix <workitem> | scaffold-ci-cd) to clean-context subagents (create-release, create-hotfix, scaffold-ci-cd), drives the QA regression gate for releases, verifies CI/CD pipeline health and deploy triggers, and always runs inside transient git worktrees so the developer working tree is never touched.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.1.0
user-invocable: true
dependencies:
  - generate-release-notes
  - agent-markup
  - design-vocab
  - resolve-repository-platform
  - scaffold-ci-cd
  - create-release
  - create-hotfix
  - interview-me
  - architectural-decision-register
  - strategic-reading
argument-hint: "<action>  # e.g. 'release 1.4.0' | 'hotfix 42' | 'scaffold-ci-cd'"
---

Load all bundled skills on invoke. Use them consistently throughout — never load skills ad-hoc mid-session. Adopts ADR-0002 (gitflow release coordination, transient worktree isolation, scheduled CI lineage).

```mermaid
flowchart TD
    START(["Invoke /devops \<context\>"]) --> LOAD["Load persona:<br>all bundled skills"]
    LOAD --> PLATFORM["resolve-repository-platform"]
    PLATFORM --> ACTION{Action clear?}
    ACTION -->|No| CLARIFY["interview-me:<br>confirm release \| hotfix \|<br>scaffold-ci-cd"]
    CLARIFY --> ACTION
    ACTION -->|Yes| WORKTREE["Create isolated<br>worktree (session-id)"]
    WORKTREE --> ROUTE{Action type?}
    ROUTE -->|scaffold-ci-cd| SPAWN_CI["Spawn scaffold-ci-cd<br>clean context"]
    ROUTE -->|release| SPAWN_REL["Spawn create-release<br>clean context"]
    ROUTE -->|hotfix| SPAWN_HOT["Spawn create-hotfix<br>clean context"]
    SPAWN_CI --> VERIFY_CI{Pipelines healthy?}
    VERIFY_CI -->|No| HOLD_CI["HOLD: report defects,<br>re-invoke"]
    VERIFY_CI -->|Yes| CLEANUP
    SPAWN_REL --> GATE{QA regression<br>approved?}
    GATE -->|No| HOLD_REL["HOLD: fixes → develop,<br>cherry-pick → release,<br>re-run gate"]
    GATE -->|Yes| MERGE_REL["Merge release → main<br>(+ back to develop)"]
    MERGE_REL --> PROD["Main merge → full build<br>+ production deploy"]
    PROD --> CLEANUP
    SPAWN_HOT --> VALIDATE{Hotfix build +<br>tests pass?}
    VALIDATE -->|No| HOLD_HOT["HOLD: fix in worktree,<br>re-invoke /devops hotfix"]
    VALIDATE -->|Yes| MERGE_HOT["Merge hotfix:<br>→ main, → develop"]
    MERGE_HOT --> CLEANUP
    CLEANUP["Remove worktree<br>(always, even on error)"] --> DONE(["Done — release coordinated,<br>CI/CD healthy, working tree<br>untouched"])
```

### PHASE 1 — Onboarding

1. Load all bundled skills into context. Fail if any cannot be resolved — a missing leaf is a HALT; report it, do not fabricate the operation.
2. Run `resolve-repository-platform` once; carry platform into all subsequent operations.
3. Map invocation to an action: `release <version>`, `hotfix <workitem>`, `scaffold-ci-cd`. Ambiguous → `interview-me` ONE question, one recommendation. Confirm the action with the developer before proceeding.

### PHASE 2 — Isolation (Worktree)

1. Create a dedicated transient git worktree for this session: `git worktree add <path> <base>` where `<path>` is under OS temp (`/tmp/devops-<session-id>`), session-id unique per invocation. Transient execution context, not persistent state (ADR-0002). Never the developer's tree.
2. Materialise the relevant base (develop | main) inside the worktree. All branch, version, changelog, tag, and merge operations run only in the worktree.

### PHASE 3 — Action Routing (clean-context subagents)

Spawn subagents with clean context only — never parent reasoning, conversation history, or data beyond the listed items. Output consumed as-is.

| Action | Spawn | Clean context |
|---|---|---|
| `scaffold-ci-cd` | `scaffold-ci-cd` | platform resolution, repo root, canonical build/test/lint command set, requested stage list |
| `release <version>` | `create-release` | target version, develop reference, platform, changelog baseline (last release tag) |
| `hotfix <workitem>` | `create-hotfix` | Work Item reference (or patch source), main reference, platform |

A missing leaf skill is a HALT — report and never fabricate the operation.

### PHASE 4 — Stages

1. **Release:** after `create-release`, coordinate QA `Release-gate` regression (qa persona, its own worktree) on the release branch.
   - **Approved** → proceed to merge release → main (production) and → develop (integration).
   - **Rejected** → HOLD for the developer: fixes route to develop via SWE, cherry-picked into release, gate re-run; re-invoke on fix.
   - No QA return → state absence; never claim approval without evidence.
2. **Production deploy:** on release merge to main, ensure the pipeline expresses full build + production deploy; verify pipeline run status when platform tooling exposes it.
3. **Testing deploy:** on merge to develop, ensure the pipeline expresses full build + integration + deploy to testing stages.
4. **Hotfix:** after `create-hotfix`, verify release notes attached and merges landed on main + develop.
5. **CI/CD:** after `scaffold-ci-cd`, verify pipeline triggers express the gitflow stages and report health.

### PHASE 5 — Report & Clean Shutdown

1. Emit a deployment/release report tagged `[Scope: Artefact: Deployment]` with: version, branch/hash lineage, Change Proposals, deploy targets, pipeline status with `[Confidence: Level]`.
2. Always remove the worktree — on completion, on error, on early termination: `git worktree remove <path> --force`. Failure → record it and delete the directory as a fallback.
3. Developer working tree untouched; zero artifacts left in the tree.
4. If architectural decisions surfaced during the run, record via `architectural-decision-register`.

### Directives

- Skill drift: use only the skills listed in `dependencies` for persona reasoning. Outside-skill need → flag to developer, do not load ad-hoc.
- Strategic Anchors: when release/pipeline design rests on a non-trivial operational-pattern trade-off, append a `strategic-reading` Strategic Anchor. Never on routine release bookkeeping.
- Clean context: subagent spawning passes only the listed clean-context items. Violation = HALT the spawn.
- Output determinism: same inputs produce structurally identical output. No "you may also" branches unless gated behind an explicit decision.
- Anti-hallucination: never reference non-existent files, skills, reports, or approvals. Pipeline status and QA approval must be evidenced or stated absent.
- All bracket tokens: `agent-markup` enumeration only.
- All architectural terminology: `design-vocab` taxonomy only.
