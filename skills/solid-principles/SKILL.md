---
name: solid-principles
description: 'Enforce SOLID object-oriented design principles to ensure scalable, maintainable, and decoupled code. Apply during code generation and structural audits to detect God classes, tight coupling, and brittle inheritance.'
user-invocable: true
dependencies:
  - agent-markup   # [Policy: Enforced] rule stance, [Risk: Level] breach severity, [Confidence: Level] finding calibration
references:
  - "Agile Software Development, Principles, Patterns, and Practices (Robert C. Martin)"
---

## SOLID Principles

**Role:** Expert engineer enforcing OOP that is resilient to change, cohesive, and loosely coupled. Apply when generating, refactoring, or reviewing OOP code.

### Core Principles `[Policy: Enforced]`
1. **SRP (Single Responsibility):** A class/module must have only ONE reason to change. *Rule: Split "God classes" or modules mixing business logic with cross-cutting concerns (logging, UI, DB).*
2. **OCP (Open/Closed):** Open for extension, closed for modification. *Rule: Add new behaviour via new implementations/plugins, NOT by mutating existing tested code or adding massive `switch` statements.*
3. **LSP (Liskov Substitution):** Subtypes must be substitutable for base types without breaking correctness. *Rule: Subclasses must not throw unexpected `NotImplementedException`s or mutate base constraints.*
4. **ISP (Interface Segregation):** Clients must not be forced to depend on interfaces they do not use. *Rule: Split "fat" interfaces into smaller, role-specific client interfaces.*
5. **DIP (Dependency Inversion):** High-level modules must not depend on low-level modules; both must depend on abstractions. *Rule: Inject dependencies. Concrete classes never instantiate their own heavy collaborators.*

### Execution Directives
**Mode:** When writing new code, apply the principles inline as you generate — no audit, no report. Run the loop below only when reviewing or modifying existing code, or when a generated artifact would breach a principle.
1. **Audit** only the supplied code for SOLID breaches before generating logic. Never infer a breach from code you cannot see.
2. **HALT** on a breach: name the principle, tag `[Risk: Level]` and `[Confidence: Level]`, then emit the isolated, refactored components. Phrase any `[Confidence: Possible]` finding as "requires verification" — never assert it.
3. **Calibrate severity:**
   * Hardcoded external instantiation (DB/Network) = `[Risk: High]` (DIP breach).
   * Multi-responsibility God class = `[Risk: High]` (SRP breach).
   * Subtype breaking the base contract = `[Risk: High]` (LSP breach).
   * Interface forcing unimplemented methods = `[Risk: Medium]` (ISP breach).
4. **Refactor:** favour composition and interface injection over deep inheritance trees. Always output the isolated, refactored components.