---
name: solid-principles
description: 'Enforce SOLID object-oriented design principles to ensure scalable, maintainable, and decoupled code. Apply during code generation and structural audits to detect God classes, tight coupling, and brittle inheritance.'
user-invocable: true
dependencies:
  - agent-markup   # [Policy: Enforced] rule stance and [Risk: Level] breach severity
references:
  - "Agile Software Development, Principles, Patterns, and Practices (Robert C. Martin)"
---

## SOLID Principles

**Role:** Expert Engineer enforcing object-oriented design that is resilient to change, highly cohesive, and loosely coupled.

**Scope:** Apply universally when generating, refactoring, or reviewing OOP code.

### Core Principles `[Policy: Enforced]`
1. **SRP (Single Responsibility):** A class/module must have only ONE reason to change. *Rule: Split "God classes" or modules mixing business logic with cross-cutting concerns (logging, UI, DB).*
2. **OCP (Open/Closed):** Open for extension, closed for modification. *Rule: Add new behaviour via new implementations/plugins, NOT by mutating existing tested code or adding massive `switch` statements.*
3. **LSP (Liskov Substitution):** Subtypes must be substitutable for base types without breaking correctness. *Rule: Subclasses must not throw unexpected `NotImplementedException`s or mutate base constraints.*
4. **ISP (Interface Segregation):** Clients must not be forced to depend on interfaces they do not use. *Rule: Split "fat" interfaces into smaller, role-specific client interfaces.*
5. **DIP (Dependency Inversion):** High-level modules must not depend on low-level modules; both must depend on abstractions. *Rule: Inject dependencies. Concrete classes never instantiate their own heavy collaborators.*

### Execution Directives
1. **Audit:** Scan requested or existing code for SOLID breaches before generating logic.
2. **HALT:** On violation, stop execution, explain the breach, and provide the decoupled fix. Tag severity with `[Risk: Level]`:
   * Hardcoded external instantiation (DB/Network) = `[Risk: High]` (DIP breach).
   * Massive multi-responsibility God class = `[Risk: High]` (SRP breach).
   * Interface forcing unimplemented methods = `[Risk: Medium]` (ISP/LSP breach).
3. **Refactor:** Favour composition and interface injection over deep inheritance trees. Always output the isolated, refactored components.