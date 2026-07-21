---
name: solid-principles
description: 'Enforce SOLID object-oriented design principles to ensure scalable, maintainable, and decoupled code. Apply during code generation and structural audits to detect God classes, tight coupling, and brittle inheritance.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
user-invocable: true
dependencies:
  - agent-markup
references:
  - "Agile Software Development, Principles, Patterns, and Practices (Robert C. Martin)"
---
Role: Enforce OOP that is resilient to change, cohesive, and loosely coupled.

Principles `[Policy: Enforced]`:
1. SRP: A class/Module must have ONE reason to change. Split "God classes" mixing business logic with cross-cutting concerns (logging, UI, DB).
2. OCP: Open for extension, closed for modification. Add new behaviour via new Impls/plugins, NOT mutating existing tested code or adding massive `switch` statements.
3. LSP: Subtypes must be substitutable for base types without breaking correctness. Subclasses must not throw unexpected exceptions or mutate base constraints.
4. ISP: Clients must not depend on Interfaces they do not use. Split "fat" Interfaces into smaller, role-specific client Interfaces.
5. DIP: High-level Modules must not depend on low-level Modules; both depend on abstractions. Inject dependencies. Concrete classes never instantiate their own heavy collaborators.

Execution:
- Writing new code: apply inline — no audit, no report.
- Reviewing/modifying: run loop:
  1. Audit only supplied code. Never infer breach from unseen code.
  2. HALT on breach: name principle, tag `[Risk: Level]` + `[Confidence: Level]`, emit refactored components. `[Confidence: Possible]` = "requires verification".
  3. Calibrate: Hardcoded external instantiation (DB/Network) = `[Risk: High]` (DIP); Multi-responsibility God class = `[Risk: High]` (SRP); Subtype breaking base contract = `[Risk: High]` (LSP); Interface forcing unimplemented methods = `[Risk: Medium]` (ISP).
  4. Refactor: favour composition and Interface injection over deep inheritance trees. Output isolated, refactored components.