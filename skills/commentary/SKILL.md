---
name: commentary
description: Shared contract defining when and how to write inline code comments. Consumed by skills that add or audit codebase commentary (document-a-codebase). Not user-invocable.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - agent-markup
user-invocable: false
---

## When to comment

| Category | Examples | Expectation |
|----------|----------|-------------|
| Architectural decisions | Why this pattern was chosen over alternatives | Always |
| Non-obvious complexity | Algorithmic nuance, tricky edge cases | Always |
| Known failure points | Race conditions, resource leaks, error states | Always |
| Performance implications | N+1 queries, O(n²) loops, cache invalidation | Always |
| Security considerations | Input sanitization, auth bypass paths | Always |
| Invariants & assumptions | pre/post-conditions, null policies, thread safety | Always |
| Non-obvious workarounds | Why a seemingly wrong approach is needed | Always |
| Third-party quirk | Patching around a library bug | Always |

## When NOT to comment

| Category | Reason |
|----------|--------|
| Obvious code | `i++  // increment i` adds zero value |
| Self-explanatory naming | Good names need no translation |
| Boilerplate | Hooks, lifecycle methods, getters |
| Standard patterns | Factory, Singleton, Observer — use naming |
| TODO/FIXME without context | Noisy unless paired with a reason |

## Comment style

- Explain WHY, not WHAT — the code says what it does.
- Concise, value-focused — one sentence preferred. Two max.
- Present tense, imperative tone.
- Reference the FDS requirement or blueprint Seam when applicable: `# FDS §3.2 — rate limit enforced here`.
- Place before the code they describe, on the same indentation level.

## Prohibited

- Redundant comments that restate the code.
- Commented-out code — delete it or keep it live.
- Noisy headers (`/*** FILE: foo.ts ***/`, `// ----- section -----`).
- `@author` tags — git blame is the source of truth.
- Block comments for single-line statements.
- Comments that would go stale — if the code is self-explanatory, no comment is needed.