---
name: architectural-decision-register
description: Generate, format, and catalog architectural decisions into a centralized registry at docs/adr/. Enforce active ADR compliance during code review and finalize ADR status on PR merge.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - agent-markup
  - design-vocab
user-invocable: true
---

```mermaid
flowchart TD
    START(["Invoke architectural-decision-register"]) --> MODE_GEN{Generate<br>mode?}
    MODE_GEN -->|Yes| GEN["Accept context:<br>proposed change, alternatives,<br>decision, consequences"]
    GEN --> EXISTS{ADR exists for<br>this topic?}
    EXISTS -->|Yes| HALT1(["HALT: already documented"])
    EXISTS -->|No| WRITE["Write ADR-XXXX.md<br>to docs/adr/"]
    WRITE --> DONE(["Done"])

    MODE_GEN -->|No| MODE_ENF{Enforce<br>mode?}
    MODE_ENF -->|Yes| ENF["Scan code + commit diff<br>against active ADRs"]
    ENF --> VIOLATE{ADR violation<br>found?}
    VIOLATE -->|Yes| REJECT["Set ticket to todo,<br>quality-gate label,<br>append violation"]
    REJECT --> HALT2(["HALT: resolve violations"])
    VIOLATE -->|No| DONE

    MODE_ENF -->|No| FINAL["PR merged for ADR"]
    FINAL --> STATUS{Status is<br>Proposed?}
    STATUS -->|Yes| UPDATE["Change to Accepted,<br>commit ADR"]
    STATUS -->|No| DONE
    UPDATE --> DONE
```

1. PHASE 1 (Generate): Accept context (proposed change, alternatives, selected decision, consequences). Check `docs/adr/` for existing ADR on same topic. Already exists → HALT. Missing → determine next number (highest ADR-XXXX + 1, zero-padded to 4 digits), write `docs/adr/ADR-XXXX.md` using template below.

2. PHASE 2 (Enforce): Scan codebase + commit diff against active ADRs (Status: Proposed or Accepted). Report every violation: location, ADR reference, violated constraint, corrective action. Violations found → set ticket to `todo`, apply `quality-gate` label, append violation details, HALT.

3. PHASE 3 (Finalize): On PR merge for an ADR, check status. `Proposed` → change to `Accepted`, commit `docs/adr/ADR-XXXX.md`. Otherwise → no-op.

Directives:
- Template:
  ```markdown
  # ADR [000X]: [Title]

  ## Status

  [Proposed | Accepted | Deprecated | Superseded by ADR-XXXX]

  ## Context

  [Explanation of the problem, constraints, and options considered]

  ## Decision

  [The definitive technical or architectural choice made]

  ## Consequences

  - **Positive:** [Benefits of this approach]
  - **Negative:** [Trade-offs, tech debt, or limitations]
  - **Implementation Notes:** [Specific constraints, e.g., pnpm worktree rules, directory structures, or strict typing requirements]
  ```
- Output Directory: `docs/adr/`.
- File Format: `.md` (Markdown).
- Strict `agent-markup` enumerations.
- Use `design-vocab` taxonomy for architecture terms within ADR content.
- Never overwrite existing ADRs.
- Numbering: find highest existing ADR-XXXX number, increment by 1, zero-pad to 4 digits.