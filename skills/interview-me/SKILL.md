---
name: interview-me
description: Relentlessly stress-test a system design, migration plan, or codebase architecture before building. Walk down each decision tree branch one-by-one, providing an expert recommendation with every question, and extracting answers from the codebase automatically whenever possible.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - design-vocab
---
1. Pre-parse repository manifests, configs, schemas to auto-resolve technical details. Focus interview on human-centric gaps only.
2. Ask ONE question at a time. Before first: announce `move-next` protocol — answering/following-up is NOT advancement. Only literal `move-next` advances.
3. Every question includes a calculated, recommended baseline answer.
4. Strict `design-vocab` taxonomy: Module, Interface, Implementation, Depth, Seam, Adapter. Prohibited: component, service, unit, API, signature, boundary.

Branches:
  - Business Domain & Persona: roles, system owners, `[Auth: Scope]`
  - Module Topology & Ecosystem: physical modules, critical Interfaces, external integrations via Seams
  - Tech Stack Lifecycle: dependency/support windows against EOL horizons for current year
  - DevOps Pipeline & Governance: workflow rules, state transitions, verification gates
  - Operations & Incident: support tiering, SLAs, knowledge locations, breach protocols
  - Data Layer & Security: data dictionaries, schema alterations, multi-tenancy isolation