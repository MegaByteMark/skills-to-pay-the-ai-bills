---
name: agent-handoff
description: 'Shared contract defining the two modes of agent-to-agent context handoff at spawn sites: Clean (isolation — parent context would taint the leaf) and Enriched (bag — parent context enriches the leaf beyond what repo artefacts provide). Defines the [Handoff: Mode] token, declaration syntax for both modes, validation rules (absent declared fields = graceful fallback; undeclared fields = HALT), and the mode selection rule. Consumed by every orchestrator and leaf that spawns or receives a subagent. Enforced by skill-authoring Rule 14.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - agent-markup
  - design-vocab
user-invocable: false
---

Every orchestrator→leaf spawn in this repo passes context. Two modes exist, serving opposite purposes. The choice is principled, not arbitrary.

## Mode Selection Rule

> **Use Clean (isolation) when the leaf's output must be free of parent bias** — reviews, audits, adversarial checks, any spawn where the parent's reasoning could taint the leaf's independence.
>
> **Use Enriched (bag) when the leaf's output benefits from parent context that isn't yet persisted to repo artefacts** — PR creation, teaching, remediation, any spawn where the parent holds in-context knowledge that improves the leaf's output and won't survive a clean-context handoff.

Criterion: *does parent context taint or enrich?*

## Clean Mode `[Handoff: Clean]`

Isolation — the orchestrator passes a fixed, small item set. No parent reasoning, no conversation history, no intermediate state. The leaf starts fresh.

**Orchestrator declaration (spawn site):**
```markdown
**Handoff:** `[Handoff: Clean]` → `<leaf-name>`
Passed: <comma-separated item list>.
```

**Leaf declaration (consume site):**
```markdown
**Accepts:** `[Handoff: Clean]` from `<orchestrator>` PHASE <N>
Accepted: <comma-separated item list>.
```

Both sides list the same items. The orchestrator is the source of truth; the leaf restates for self-contained readability.

## Enriched Mode `[Handoff: Enriched]`

Enrichment — the orchestrator passes a typed field bag holding in-context knowledge not yet persisted to repo artefacts. Each field is optional (absent fields trigger the leaf's stated fallback). The entire bag is optional (absent bag triggers the leaf's whole-bag fallback, enabling headless mode).

**Orchestrator declaration (spawn site):**
```markdown
**Handoff:** `[Handoff: Enriched]` → `<leaf-name>`

| Field | Type | Source |
|---|---|---|
| <field_name> | <type> | <where the orchestrator constructs it> |
```

**Leaf declaration (consume site):**
```markdown
**Accepts:** `[Handoff: Enriched]` from `<orchestrator>` PHASE <N>

| Field | Required? | Fallback if absent |
|---|---|---|
| <field_name> | no | <what the leaf does without it> |
```

Required? is always `no` — absent fields are expected (headless mode, partial context). If a field is truly required, the leaf HALTs with a clear message rather than degrading silently.

## Validation Rules

| Condition | Action |
|---|---|
| Declared field present, correct type | Leaf consumes it |
| Declared field absent | Leaf fires its stated fallback for that field — not an error |
| Undeclared field in bag (not in either party's schema) | HALT: orchestrator is passing context the leaf didn't agree to accept — contract violation |
| Type mismatch (e.g. array expected, string received) | HALT: malformed declaration |
| Bag entirely absent when Enriched mode declared | Leaf's stated whole-bag fallback fires — not an error, enables headless mode |

Key distinction: **absent declared fields are expected** (headless mode, partial context, interactive vs non-interactive). **Undeclared fields are violations** (the orchestrator is bleeding unagreed context into the leaf — same principle as Clean mode's "NEVER pass parent reasoning").

## Declaration Site

The orchestrator is the source of truth — it constructs the bag, so it owns the declaration. The leaf references it and restates only the fields it materially relies on (for self-contained readability). Cross-reference is explicit: the leaf names the orchestrator and phase.

## Migration

The following spawn sites were retrofitted from informal prose to formal `[Handoff: Mode]` declarations in the introducing PR:

**Clean mode:**
- `swe` → `adversarial-review` (PHASE 3)
- `po` → `create-epic`, `create-user-story`, `create-bug-report`, `create-milestone` (PHASE 4)
- `qa` → `remediate-test-coverage`, `create-bug-report` (PHASE 6)
- `devops` → `create-release`, `create-hotfix`, `scaffold-ci-cd` (PHASE 3)

**Enriched mode:**
- `swe` → `create-pr` (PHASE 5)
- `teach-me` → `teach-a-skill` (Step 4)
- `vibe-code-antidote` → `teach-a-skill` (Escalation step 3)

`[Handoff: Clean]` replaces prior "clean context only" / "Consume clean context" prose. `[Handoff: Enriched]` replaces prior informal context bag descriptions.

## Directives

- Every skill that spawns a subagent MUST declare `[Handoff: Clean]` or `[Handoff: Enriched]` at the spawn site. Undeclared spawn sites fail skill-authoring compliance (Rule 14).
- Every leaf that receives a subagent spawn MUST declare a matching consume block. Missing consume blocks fail compliance.
- Cross-pair field matching: every field in the orchestrator's declare table MUST appear in the leaf's accept table. Mismatches fail compliance.
- `[Handoff: Mode]` token declared in `agent-markup`, owned by `agent-handoff`.
- All architectural terminology: `design-vocab` taxonomy.
