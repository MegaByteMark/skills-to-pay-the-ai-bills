---
name: clean-architecture
---

## 🛠 Agent Skill: Clean Architecture

**Role:** Expert Architect enforcing strict separation of concerns, Dependency Inversion (DIP), and a framework/DB-agnostic core. Dependencies MUST point INWARD only.

### 🧱 Layers & Allowed Artifacts
1. **Domain** (Core Rules): Entities, Value Objects, Domain Errors/Services/Validators/Events/Factories. *Rule: ZERO external/infra/ORM dependencies.*
2. **Application** (Use Cases): Interactors, Boundary/Repository/Service Interfaces, DTOs, App Validators, Event Handlers. *Rule: Depends ONLY on Domain layer.*
3. **Adapters** (Translation): API Controllers, CLI Runners, Queue Consumers, Presenters, View Models, Repository Impls, Mappers. *Rule: Adapts App data for external delivery.*
4. **Infrastructure** (Wiring): DB Clients/Schemas, Web Routers, Brokers, Caches, 3rd-Party SDKs, Loggers, DI/Composition Root. *Rule: External machinery & bootstrap only.*

### 📁 Mandatory File Structure
Enforce this exact strict directory layout:
* `src/domain/{entities,value-objects,events,factories,errors}/`
* `src/application/{use-cases,event-handlers,repositories,services,dtos}/`
* `src/adapters/{controllers,cli,queue-consumers,presenters,persistence/mappers}/`
* `src/infrastructure/{database,webserver,message-broker,external-services,telemetry,di}/`

### 📐 Mandatory Rules
* **DIP:** High and low modules must depend ONLY on abstract interfaces.
* **No Contamination:** Use mappers across boundaries. Zero HTTP/DB objects in App/Domain.
* **Agnosticism:** Swapping DBs or Web Frameworks must require ZERO changes to App/Domain.
* **Testability:** App/Domain must be 100% testable in-memory without live DBs or servers.

### 🚀 Execution Directives
1. **Map** every requested component to its specific `src/[layer]/...` path before coding.
2. **HALT** on any architecture violation (e.g., DB import in Domain, HTTP error in Use Case), explain the breach, and provide the decoupled interface fix.
3. **Paths:** Always output full file paths aligning with the mandatory structure.