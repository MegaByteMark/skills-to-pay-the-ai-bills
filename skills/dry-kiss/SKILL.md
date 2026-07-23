---
name: dry-kiss
description: 'Enforce baseline clean-code heuristics (DRY, KISS, YAGNI) to prevent over-engineering, code duplication, and unnecessary complexity. Apply during code generation and review.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
user-invocable: true
dependencies:
  - agent-markup
---
Role: Expert reviewer blocking over-engineered, duplicated, or overly clever code.

Principles `[Policy: Enforced]`:
1. DRY: Every piece of knowledge has one unambiguous representation. Abstract genuinely duplicated logic into a shared Module; do NOT abstract accidental duplication (code that merely looks alike).
2. YAGNI: Build nothing for hypothetical future use. Reject unused code, speculative Interfaces, generic wrappers, parameters with no current caller.
3. KISS: Favour readability over cleverness. Reject nested ternaries, control-flow nesting >3 levels, multi-statement one-liners where a flat procedural block is equivalent.

Execution:
- Writing new code: apply principles inline as you generate — no audit, no report.
- Reviewing/modifying code: run the loop:
  1. Audit only supplied code. Never infer a breach from code you cannot see.
  2. HALT on breach: name principle, tag `[Risk: Level]` + `[Confidence: Level]`, emit corrected code. `[Confidence: Possible]` = "requires verification", never asserted.
  3. Calibrate: live duplication across Modules / speculative abstraction with zero consumers = `[Risk: Medium]`; local cleverness / readability lapse = `[Risk: Low]`.