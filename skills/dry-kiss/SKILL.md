---
name: dry-kiss
description: 'Enforce baseline clean code heuristics (DRY, KISS, YAGNI) to prevent over-engineering, code duplication, and unnecessary complexity.'
user-invocable: true
dependencies:
  - agent-markup
---

## Pragmatic Coding Principles

**Role:** Expert Reviewer blocking over-engineered, duplicated, or overly clever code.

### Core Principles `[Policy: Enforced]`
1. **DRY (Don't Repeat Yourself):** Every piece of knowledge must have a single, unambiguous representation. *Rule: Abstract duplicated logic into shared functions/constants, but DO NOT prematurely abstract accidental duplication (where two things just happen to look similar right now).*
2. **YAGNI (You Aren't Gonna Need It):** Never build features or abstractions for hypothetical future use cases. *Rule: Delete unused code. Reject speculative interfaces, generic wrappers, or parameters that have no current consumer.*
3. **KISS (Keep It Simple, Stupid):** Favour readability over cleverness. *Rule: Reject nested ternaries, excessive closures, and dense one-liners if a straightforward procedural block is easier for a junior developer to read.*
