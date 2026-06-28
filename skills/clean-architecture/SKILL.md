---
name: clean-architecture
description: 'Scaffold, place, and enforce a layered Clean Architecture. Map each artifact (Entities, Use Cases, Interface Adapters, Infrastructure) to its layer, keep dependencies pointing INWARD via Dependency Inversion, keep the Domain/Application core framework- and DB-agnostic, and HALT on inward-dependency violations. Use when generating a new layered structure, deciding which layer code belongs in, or reviewing a layered codebase for dependency-rule breaches. Not a universal mandate — apply only to projects that have adopted (or are adopting) a layered/clean design.'
user-invocable: true
dependencies:
  - design-vocab   # Architectural taxonomy (Interface, Seam, Adapter, Implementation)
  - agent-markup   # [Policy: Enforced] rule stance and [Risk: Level] breach severity
references:
  - "Clean Architecture (Robert C. Martin) - the Dependency Rule and the four concentric layers"
  - "Hexagonal Architecture / Ports & Adapters (Alistair Cockburn) - Port Interfaces driven by Adapters at Seams"
---

## Clean Architecture

**Role:** Expert Architect enforcing strict separation of concerns, Dependency Inversion (DIP), and a framework/DB-agnostic core. Dependencies MUST point INWARD only.

**Scope:** Clean Architecture is one option, not a universal default. Apply this skill only when the project has adopted (or is adopting) a layered, dependency-inverted design. It is the *prescriptive* counterpart to `analyze-a-codebase`, which *describes* whatever architecture already exists — do not retrofit these layers onto a codebase that deliberately uses another style.

### Layers & Allowed Artifacts
1. **Domain** (Core Rules): Entities, Value Objects, Domain Errors/Services/Validators/Events/Factories. *Rule: ZERO external/infra/ORM dependencies.*
2. **Application** (Use Cases): Interactors, Port Interfaces (Seams) for Repositories & external collaborators, DTOs, App Validators, Event Handlers. *Rule: Depends ONLY on Domain layer.*
3. **Interface Adapters** (Translation): HTTP Controllers, CLI Runners, Queue Consumers, Presenters, View Models, Repository Impls, Mappers. *Rule: Adapts App data for external delivery; concrete Repository Impls/Presenters here are the design-vocab Adapters that satisfy the Application's Port Interfaces.*
4. **Infrastructure** (Wiring): DB Clients/Schemas, Web Routers, Brokers, Caches, 3rd-Party SDKs, Loggers, DI/Composition Root. *Rule: External machinery & bootstrap only.*

### design-vocab Mapping
Clean Architecture names the layer roles; `design-vocab` names the mechanics that connect them. Describe structural mechanics in these terms (and NEVER use "boundary"):
| Clean Architecture | design-vocab | Meaning |
| :--- | :--- | :--- |
| "boundary" between layers | **Seam** | physical location where a crossing Interface lives |
| Port / boundary interface | **Interface** | abstract surface that crosses the Seam |
| Repository Impl / Gateway / Presenter | **Adapter** | concrete artifact satisfying an Interface at a Seam |
| inner-layer business logic | **Implementation** | the body behind an Interface |

### Canonical Structure (translate to language idioms)
The **Dependency Rule is the invariant**; the directory names below are a *reference grouping, not a literal cross-language mandate*. Reproduce this grouping and the inward dependency direction using each language's idiomatic packaging (e.g. Go packages, C# projects/assemblies, Java/Kotlin modules, Python packages) — never force a JS-style `src/` tree onto a language that packages differently.
* Domain core: `…/domain/{entities,value-objects,events,factories,errors}`
* Application: `…/application/{use-cases,event-handlers,repositories,services,dtos}`
* Interface Adapters: `…/adapters/{controllers,cli,queue-consumers,presenters,persistence/mappers}`
* Infrastructure: `…/infrastructure/{database,webserver,message-broker,external-services,telemetry,di}`

### Mandatory Rules `[Policy: Enforced]`
* **DIP:** High- and low-level Modules must depend ONLY on abstract Interfaces.
* **No Contamination:** Use mappers across Seams. Zero HTTP/DB objects in App/Domain.
* **Agnosticism:** Swapping DBs or Web Frameworks must require ZERO changes to App/Domain.
* **Testability:** App/Domain must be 100% testable in-memory without live DBs or servers.

### Execution Directives
1. **Map** every requested Module to its layer (and idiomatic path) before coding.
2. **HALT** on any inward-dependency violation, then explain the breach and provide the decoupled Interface fix. Tag severity with `[Risk: Level]`: a DB/ORM import in Domain or an HTTP type leaking into a Use Case is `[Risk: High]`; a missing mapper at a Seam is typically `[Risk: Medium]`.
3. **Paths:** Always output full, idiomatic file paths aligned with the canonical grouping.
