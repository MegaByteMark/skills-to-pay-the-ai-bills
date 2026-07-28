---
name: red-green-refactor-tdd
description: 'Enforce Test-Driven Development via strict Red-Green-Refactor cycles. Guides the agent through writing a failing test (Red), implementing minimal production code under YAGNI (Green), and cleaning up via DRY/KISS (Refactor) — delegating to dry-kiss and refactor skills for code-quality enforcement.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
user-invocable: true
dependencies:
  - dry-kiss
  - refactor
  - detect-test-harness
  - agent-markup
  - design-vocab
---

Never write production code before a failing test exists or refactor without a green suite. Each cycle: seconds or minutes.

```mermaid
flowchart TD
    START(["Invoke TDD cycle"]) --> HAS_TASK{Target behavior<br>specified?}
    HAS_TASK -->|No| ASK["Ask user for<br>single behavior"]
    ASK --> RED_GATE{Red phase?}
    HAS_TASK -->|Yes| RED_GATE

    RED_GATE -->|Yes| WRITE_TEST["Write smallest<br>failing test"]
    WRITE_TEST --> DETECT["Resolve test harness"]
    DETECT --> RUN_TEST["Run test suite"]
    RUN_TEST --> FAIL_REASON{Fails for<br>right reason?}
    FAIL_REASON -->|No| HALT_RED(["HALT: fix test intent"])
    FAIL_REASON -->|Yes| GREEN_GATE{Green phase?}

    GREEN_GATE -->|Yes| YAGNI["Invoke dry-kiss: YAGNI"]
    YAGNI --> WRITE_CODE["Write minimal code<br>to pass"]
    WRITE_CODE --> RUN_TEST2["Run test suite"]
    RUN_TEST2 --> ALL_GREEN{All tests pass?}
    ALL_GREEN -->|No| HALT_GREEN(["HALT: fix implementation"])
    ALL_GREEN -->|Yes| REFACTOR_GATE{Refactor phase?}

    REFACTOR_GATE -->|Yes| DRY_KISS["Invoke dry-kiss: DRY/KISS"]
    DRY_KISS --> VIOLATIONS{Found violations?}
    VIOLATIONS -->|Yes| RESOLVE["Resolve violations"]
    RESOLVE --> VIOLATIONS
    VIOLATIONS -->|No| NEEDS_REFACTOR{Needs structural<br>cleanup?}
    NEEDS_REFACTOR -->|Yes| REFACT["Invoke refactor skill"]
    REFACT --> RUN_TEST3["Run test suite"]
    NEEDS_REFACTOR -->|No| RUN_TEST3
    RUN_TEST3 --> STILL_GREEN{Still green?}
    STILL_GREEN -->|No| HALT_REFACT(["HALT: revert, fix"])
    STILL_GREEN -->|Yes| ALL_CLEAN{All clean?}
    ALL_CLEAN -->|No| VIOLATIONS
    ALL_CLEAN -->|Yes| MORE_IMPL{More behavior<br>to implement?}
    MORE_IMPL -->|Yes| RED_GATE
    MORE_IMPL -->|No| DONE(["Done"])
```

### Red Phase — Write a failing test

- Write the smallest test for one behavior or edge case.
- Load `detect-test-harness`, run the suite, confirm the new test fails for the right reason (assertion failure, not syntax/import error). Tag `[Confidence: Level]`.
- HALT if test passes or fails for the wrong reason.

### Green Phase — Make it pass

- Load `dry-kiss`. Apply YAGNI: write the simplest code to pass. Hardcoded returns, "fake it 'til you make it", sub-optimal implementations permitted.
- Run the full suite. All tests must pass. HALT otherwise.

### Refactor Phase — Clean up

- Load `dry-kiss`. Evaluate for DRY/KISS violations.
- Resolve each violation. Loop until clean.
- If structural cleanup is needed: load `refactor` and delegate.
- Run the suite after every change. Revert and HALT if tests break.
- When clean: if more behavior remains, start a new cycle.

### Integration

- `dry-kiss` governs Green (YAGNI) and Refactor (DRY/KISS). Do not duplicate.
- `refactor` handles structural transformations, delegating to `dry-kiss` and `solid-principles` internally.
- `detect-test-harness` resolves the test runner. If unresolved: ask user.
- Never skip or combine phases. `[Policy: Enforced]`.