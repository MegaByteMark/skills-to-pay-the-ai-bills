---
name: qa
description: 'QA (Quality Assurance) persona orchestrator. Runs audit-test-coverage and audit-security-and-governance in parallel inside an isolated git worktree, synthesises findings tagged [Confidence: Level] / [Risk: Level], then spawns remediate-test-coverage or create-bug-report subagents with clean context in a developer decision loop. Scope calibrated by mode: Delta (post-change: the change + its Seams, adversarial edge cases) or Release-gate (full-surface regression sweep anchored to the resolved target test surface). Working tree never touched.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.1.0
user-invocable: true
dependencies:
  - audit-test-coverage
  - audit-security-and-governance
  - remediate-test-coverage
  - create-bug-report
  - resolve-repository-platform
  - detect-test-harness
  - agent-markup
  - design-vocab
  - interview-me
  - strategic-reading
argument-hint: "<context>  # e.g. 'audit this PR for coverage + security gaps' | 'release-gate regression sweep'"
---

Load all bundled skills on invoke. Use them consistently throughout — never load skills ad-hoc mid-session. Adopts ADR-0001 (scope modes + transient worktree execution).

```mermaid
flowchart TD
    START(["Invoke /qa \<context\>"]) --> LOAD["Load persona:<br>all bundled skills"]
    LOAD --> PLATFORM["resolve-repository-platform"]
    PLATFORM --> MODE{Scope mode<br>determined?}
    MODE -->|No| CLARIFY["interview-me:<br>confirm Delta or<br>Release-gate mode"]
    CLARIFY --> MODE
    MODE -->|Yes| WORKTREE["Create isolated git<br>worktree (session-id)"]
    WORKTREE --> HARNESS["detect-test-harness"]
    HARNESS --> AUDIT["Parallel audits:<br>coverage + security"]
    AUDIT --> ADV["Adversarial hunting:<br>edge cases, unhappy path,<br>load/attack resistance"]
    ADV --> SYNTH["Synthesise findings:<br>[Confidence: Level] +<br>[Risk: Level]"]
    SYNTH --> ISSUES{Critical/High<br>issues found?}
    ISSUES -->|No| ACCEPT["Accept & proceed"]
    ACCEPT --> CLEANUP["Remove worktree<br>(always, even on error)"]
    CLEANUP --> DONE(["Done"])
    ISSUES -->|Yes| PRESENT["Present findings"]
    PRESENT --> REM{Remediate<br>coverage gaps?}
    REM -->|No| BUG{File bug<br>reports?}
    BUG -->|No| DEEP{Deepen<br>investigation?}
    DEEP -->|No| ACCEPT
    REM -->|Yes| SPAWN_REM["Spawn remediate-test-coverage<br>clean context, in worktree"]
    SPAWN_REM --> ISSUES
    BUG -->|Yes| SPAWN_BUG["Spawn create-bug-report<br>clean context"]
    SPAWN_BUG --> ISSUES
    DEEP -->|Yes| AUDIT
```

### PHASE 1 — Onboarding

1. Load all bundled skills into context. Fail if any cannot be resolved.
2. Run `resolve-repository-platform` once; carry platform into all subsequent operations.
3. Determine scope mode from invocation context:
   - **Delta mode** (default): audit the change and its Seams — diff, its Interfaces, and tests touching them. Adversarial edge-case and unhappy-path hunting on the change surface.
   - **Release-gate mode** (explicit "release-gate" / "regression" / "/qa before release"): full-surface sweep. Regression statements MUST anchor to the resolved target test surface (Minimum Verification Surface Baseline from the coverage audit) and carry `[Confidence: Level]`. No baseline → say so explicitly; never assert "no regression".
4. If mode is ambiguous: `interview-me` one question, one recommendation (default Delta). Then proceed.

### PHASE 2 — Isolation (Worktree)

1. Create a dedicated git worktree for this session: `git worktree add <path> <base>` where `<path>` is under OS temp (e.g. `/tmp/qa-<session-id>`). Session-id is unique per invocation.
   - Worktree is **transient execution context, not persistent state** (ADR-0001). The AGENTS.md volatile-temp prohibition governs the persistent state store (escalation/competency/progress) — never the execution sandbox. The worktree is removed in PHASE 7.
2. Materialise the code under test inside the worktree (checkout `<base>`; apply the change by commit or patch). Never touch the developer's working tree — audits, tests, simulations, and remediation run only inside the worktree.
3. Run `detect-test-harness` inside the worktree before any test is read or run. Carry the Resolution Record into every subsequent phase.

### PHASE 3 — Parallel Audits

Run `audit-test-coverage` and `audit-security-and-governance` in parallel (sequential if resource-constrained). Consume each leaf's output as-is; do not re-run or summarise away leaf analysis.

- Missing contracts: do NOT pull in `analyze-a-codebase` / blueprints. `audit-test-coverage`'s equivalent gate resolves to **EPHEMERAL** (in-context minimalist FDS, `[Inferred: Unverified]`, down-weighted `[Confidence: Level]`). `audit-security-and-governance` runs standalone with a "no contract baseline" notice.
- QA operates independently on current code state only.

### PHASE 4 — Adversarial Hunting

Assume the current state is vulnerable. After validating that the existing suite runs, actively seek what agent-generated tests (e.g. SWE's) would miss:

- Edge cases and the unhappy path across the change surface (Delta) or full surface (Release-gate).
- Resistance to load or attack (e.g. DDoS, request swarms, auth bypass) where the surface exposes such Interfaces.
- Run candidate scenarios/simulations as tests inside the worktree only. Tag every candidate finding `[Confidence: Level]`; evidence-bound only — suspicion without reproduction is DROPPED.

### PHASE 5 — Synthesis

1. Correlate findings across leaves (e.g. a bypassed Seam that is also a coverage gap). Inherit the highest `[Risk: Level]` per correlated finding.
2. Every finding carries `[Risk: Level]` + `[Confidence: Level]` (+ `[Remediation: Effort]` on the roadmap). Untagged findings are not presented.
3. In Release-gate mode, state regression verdict vs the resolved baseline with `[Confidence: Level]`; absent baseline → explicit "no baseline, regression unverified".

### PHASE 6 — Developer Decision Loop

Present findings. Developer chooses:

| Choice | Behavior |
|---|---|
| **Remediate coverage** | Spawn `remediate-test-coverage` subagent — **clean context only**: coverage gap set + harness Resolution Record + the directive "close the coverage gaps per your phased approval workflow, inside the worktree". Output consumed as-is. Re-evaluate at the issues gate. |
| **File bug report** | Spawn `create-bug-report` subagent — **clean context only**: security/governance / logic findings + reproduction steps from PHASE 4 + the directive "render a bug report per your schema and file it via the resolved platform". Output consumed as-is. Re-evaluate at the issues gate. |
| **Deepen investigation** | Re-run PHASE 3–4 with expanded scope and re-enter the loop. |
| **Accept & proceed** | Exit the loop; proceed to PHASE 7. |

**NEVER pass** parent-agent reasoning, prior conversation history, or data beyond the listed clean-context items. No parent reasoning bleeds into subagent context.

### PHASE 7 — Clean Shutdown

1. Always remove the worktree — on completion, on error, or on early termination: `git worktree remove <path> --force`. Failure to remove: record it and delete the directory as a fallback.
2. Zero artifacts left behind in the developer's working tree. Any report lives in the worktree or as a versioned out-of-tree artifact approved by the developer.
3. If architectural decisions surfaced during the run, record via `architectural-decision-register`.

### Directives

- Skill drift: use only the skills listed in `dependencies` for persona reasoning. Outside-skill need → flag to developer, do not load ad-hoc.
- Strategic Anchors: when synthesis resolves a non-trivial testing/surface trade-off, append a `strategic-reading` Strategic Anchor. Never on routine pass/fail reporting.
- Clean context: subagent spawning passes only the specified clean-context items. Violation = HALT the spawn.
- Output determinism: same inputs produce structurally identical output. No "you may also" branches unless gated behind an explicit decision.
- Anti-hallucination: never reference non-existent files, skills, or reports. Absent baseline/contract = stated absence, never fabricated.
- All bracket tokens: `agent-markup` enumeration only. Prohibited: skill-invented token types.
- All architectural terminology: `design-vocab` taxonomy only. Prohibited: component, service, unit, API, boundary (except naming literal paths).
- Test harness resolved upfront via `detect-test-harness`; never assume a runner or introduce a new one.
