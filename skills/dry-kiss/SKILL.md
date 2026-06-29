---
name: dry-kiss
description: 'Enforce baseline clean-code heuristics (DRY, KISS, YAGNI) to prevent over-engineering, code duplication, and unnecessary complexity. Apply during code generation and review.'
user-invocable: true
dependencies:
  - agent-markup   # [Policy: Enforced] rule stance, [Risk: Level] breach severity, [Confidence: Level] finding calibration
---

## Pragmatic Coding Principles

**Role:** Expert reviewer blocking over-engineered, duplicated, or overly clever code.

### Core Principles `[Policy: Enforced]`
1. **DRY (Don't Repeat Yourself):** Every piece of knowledge has one unambiguous representation. *Rule: Abstract genuinely duplicated logic into a shared module; do NOT abstract accidental duplication (code that merely looks alike today).*
2. **YAGNI (You Aren't Gonna Need It):** Build nothing for hypothetical future use. *Rule: Reject unused code, speculative interfaces, generic wrappers, and parameters with no current caller.*
3. **KISS (Keep It Simple):** Favour readability over cleverness. *Rule: Reject nested ternaries, control-flow nesting deeper than 3 levels, and multi-statement one-liners where a flat procedural block is equivalent.*

### Execution Directives
**Mode:** When writing new code, apply the principles inline as you generate — no audit, no report. Run the loop below only when reviewing or modifying existing code, or when a generated artifact would breach a principle.
1. **Audit** only the supplied code before generating logic. Never infer a breach from code you cannot see.
2. **HALT** on a breach: name the principle, tag `[Risk: Level]` and `[Confidence: Level]`, then emit the corrected code. Phrase any `[Confidence: Possible]` finding as "requires verification" — never assert it.
3. **Calibrate severity:** live duplication across modules or speculative abstraction with zero consumers = `[Risk: Medium]`; local cleverness or readability lapse = `[Risk: Low]`.
