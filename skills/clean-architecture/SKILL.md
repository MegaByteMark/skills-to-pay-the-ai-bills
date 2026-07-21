---
name: clean-architecture
description: 'Scaffold, place, and enforce a layered Clean Architecture. Map each artifact (Entities, Use Cases, Interface Adapters, Infrastructure) to its layer, keep dependencies pointing INWARD via Dependency Inversion, keep the Domain/Application core framework- and DB-agnostic, and HALT on inward-dependency violations. Use when generating a new layered structure, deciding which layer code belongs in, or reviewing a layered codebase for dependency-rule breaches. Not a universal mandate — apply only to projects that have adopted (or are adopting) a layered/clean design.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
user-invocable: true
dependencies:
  - design-vocab
  - agent-markup
references:
  - "Clean Architecture (Robert C. Martin) — the Dependency Rule and four concentric layers"
  - "Hexagonal Architecture / Ports & Adapters (Alistair Cockburn) — Port Interfaces driven by Adapters at Seams"
---
Apply ONLY when the project has adopted layered, dependency-inverted design. Prescriptive counterpart to `analyze-a-codebase` — do not retrofit onto deliberately different styles.

Layers:
1. Domain (Core Rules): Entities, Value Objects, Domain Errors/Services/Validators/Events/Factories. ZERO external/infra/ORM dependencies.
2. Application (Use Cases): Interactors, Port Interfaces (Seams) for Repositories & external collaborators, DTOs, App Validators, Event Handlers. Depends ONLY on Domain.
3. Interface Adapters (Translation): HTTP Controllers, CLI Runners, Queue Consumers, Presenters, View Models, Repository Impls, Mappers. Adapts App data for external delivery; Repository Impls/Presenters are the design-vocab Adapters satisfying Application's Port Interfaces.
4. Infrastructure (Wiring): DB Clients/Schemas, Web Routers, Brokers, Caches, 3rd-Party SDKs, Loggers, DI/Composition Root. External machinery & bootstrap only.

design-vocab Mapping (NEVER use "boundary"):
| Clean Architecture | design-vocab | Meaning |
| :--- | :--- | :--- |
| "boundary" between layers | Seam | physical location where Interface lives |
| Port / boundary interface | Interface | abstract surface crossing the Seam |
| Repository Impl / Gateway / Presenter | Adapter | concrete artifact satisfying Interface at Seam |
| inner-layer business logic | Implementation | body behind Interface |

Canonical grouping (translate to language idioms — never force JS-style onto non-JS languages):
- Domain: `…/domain/{entities,value-objects,events,factories,errors}`
- Application: `…/application/{use-cases,event-handlers,repositories,services,dtos}`
- Interface Adapters: `…/adapters/{controllers,cli,queue-consumers,presenters,persistence/mappers}`
- Infrastructure: `…/infrastructure/{database,webserver,message-broker,external-services,telemetry,di}`

Rules `[Policy: Enforced]`:
- DIP: High- and low-level Modules depend ONLY on abstract Interfaces.
- No Contamination: Use mappers across Seams. Zero HTTP/DB objects in App/Domain.
- Agnosticism: Swapping DBs/Web Frameworks requires ZERO changes to App/Domain.
- Testability: App/Domain 100% testable in-memory without live DBs/servers.

Execution:
1. Map every Module to its layer and idiomatic path before coding.
2. HALT on inward-dependency violation, explain breach, provide decoupled Interface fix. Tag: DB/ORM in Domain or HTTP type in Use Case = `[Risk: High]`; missing mapper at Seam = `[Risk: Medium]`.
3. Output full, idiomatic file paths aligned with canonical grouping.