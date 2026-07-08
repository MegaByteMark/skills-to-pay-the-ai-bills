---
name: skill-authoring
description: Meta-skill for creating and modifying Agent Skills in this repository. Enforces naming, frontmatter, scope-gating, prose compaction, Mermaid diagrams, dependency validation, agent-markup/design-vocab compliance, and README registration. Load before any skill create or modify operation.
dependencies: [agent-markup, design-vocab]
user-invocable: true
argument-hint: [create|modify] <skill-name>
---

Gate every skill change (create or modify) through this workflow. Halt on any gate failure; fix before proceeding.

## Workflow

```mermaid
flowchart TD
    START(["Trigger: create or modify a skill"]) --> OP{Operation?}

    OP -->|create| C1{"name unique & matches<br>^[a-z0-9]+(-[a-z0-9]+)*$ ?"}
    C1 -->|no| HALT(["HALT: reject, suggest valid name"])
    C1 -->|yes| C2{"scope overlaps existing skill?"}
    C2 -->|yes| HALT2(["HALT: point to existing skill"])
    C2 -->|no| C3{"user-invocable explicitly set?"}
    C3 -->|no| HALT3(["HALT: must be true or false"])
    C3 -->|yes| C4{"all dependencies exist &<br>no circular chain?"}
    C4 -->|no| HALT4(["HALT: report missing dep or cycle"])
    C4 -->|yes| C5{"≥2 decision points or<br>multi-phase workflow?"}
    C5 -->|yes| MERMAID["Include Mermaid diagram:<br>expand all decisions,<br>challenge vague routes"]
    C5 -->|no| PROSE["Prose only"]
    MERMAID --> CREATE
    PROSE --> CREATE
    CREATE[["Write SKILL.md with<br>compacted prose"]] --> C6{"agent-markup tokens valid?"}
    C6 -->|no| HALT5(["HALT: fix tokens to enumeration"])
    C6 -->|yes| C7{"design-vocab terms valid?"}
    C7 -->|no| HALT6(["HALT: fix vocabulary"])
    C7 -->|yes| C8{"registered in README catalog?"}
    C8 -->|no| HALT7(["HALT: add to README"])
    C8 -->|yes| COMPACT["Run prose compaction"]
    COMPACT --> DONE(["Done"])

    OP -->|modify| M1["Change-impact sweep:<br>scan dependents across repo"]
    M1 --> M2{"change breaks dependent?"}
    M2 -->|yes| HALT8(["HALT: flag breaking, propose migration"])
    M2 -->|no| M3{"human request fully<br>reasoned through?"}
    M3 -->|no| HALT9(["HALT: challenge gaps,<br>ask clarifying questions"])
    M3 -->|yes| M4["Apply change"]
    M4 --> M5{"introduces ambiguous branching?"}
    M5 -->|yes| M6{"gated behind explicit<br>decision point?"}
    M6 -->|no| HALT10(["HALT: resolve ambiguity"])
    M6 -->|yes| M7
    M5 -->|no| M7{"description or scope<br>changed materially?"}
    M7 -->|yes| SYNC["Sync README entry"]
    M7 -->|no| M8
    SYNC --> M8["Run prose compaction:<br>strip non-behavioral prose"]
    M8 --> M9["Re-validate agent-markup<br>& design-vocab"]
    M9 --> COMPACT
```

## Rules

Apply on every create or modify. Halt on violation; fix before proceeding.

| # | Rule | Detail |
|---|------|--------|
| 1 | **Naming & discovery** | Folder == `name`, matches `^[a-z0-9]+(-[a-z0-9]+)*$`, unique across repo. `SKILL.md` upper-case, flat under `skills/<name>/`, no sub-folders. |
| 2 | **Frontmatter minimum** | `name`, `description` (≤1024 chars, specific enough for runtime selection). `dependencies` always present (`[]` if none). `user-invocable` explicitly `true` or `false`. |
| 3 | **Dependency validation** | Every dep must exist as a folder under `skills/`. No circular chains. Orchestrators depend on leaves, never reverse. |
| 4 | **Scope gate** | One purpose per skill. Before adding behavior: verify it doesn't belong in a dependency or new leaf. Push back on scope creep. |
| 5 | **Prose compaction** | Strip every sentence that doesn't change what the agent *does*. No qualifiers, justifications, or restated definitions. Target: smallest token count preserving instruction completeness without introducing ambiguity. Run compaction after every modification. |
| 6 | **Mermaid gate** | Use Mermaid iff ≥2 branching decisions or multi-phase workflow. Expand all decision points; challenge vague routes. Pure contracts and single-path skills: prose only. |
| 7 | **Anti-hallucination** | Never reference non-existent files, skills, or documents. Instruct runtime validation with graceful fallback on absence. |
| 8 | **Output determinism** | Same inputs → structurally identical outputs. Eliminate "you may also" branches unless gated behind explicit deterministic decision. |
| 9 | **Markup & vocabulary** | All `[...]` tokens from `agent-markup` enumeration. All architecture terms from `design-vocab` taxonomy. No skill-invented tokens or synonyms. |
| 10 | **State persistence** | If persisting state: `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/...`. Never working tree. Never temp. |
| 11 | **README registration** | New skills: add to README catalog under correct category. On modify: sync entry if description or scope changed. |
| 12 | **Challenge the human** | Reason whether request is fully thought through. Consider downstream impact. Prioritize anti-hallucination over compliance with incomplete request. |

## Compaction directive

After every SKILL.md modification, execute this compaction pass:
1. Remove any sentence that, if deleted, does not alter agent behavior.
2. Merge repeated constraints into a single canonical statement.
3. Replace multi-sentence explanations with a single imperative instruction.
4. Verify no instruction was lost or made ambiguous — if so, restore the minimum to resolve.
