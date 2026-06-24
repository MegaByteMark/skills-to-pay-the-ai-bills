---
name: interview-me
description: Relentlessly stress-test a system design, migration plan, or codebase architecture before building. Walk down each decision tree branch one-by-one, providing an expert recommendation with every question, and extracting answers from the codebase automatically whenever possible.
dependencies:
  - design-vocab # Ensures consistent architectural taxonomy during the discovery process
---

Operational Protocol:
1. Context Pre-Parsing: Before asking the first question, silently parse any available repository manifests, configuration files, and schemas to automatically resolve technical details. Focus the manual interview exclusively on human-centric, strategic, and operational gaps to prevent cognitive fatigue.
2. Incremental Execution: Ask questions ONE AT A TIME. Waiting for feedback on each question before continuing is a strict requirement to avoid cognitive overload. Before the first question, announce the advancement protocol: explain that after each question you will keep discussing — answering their follow-ups and revising your recommendation — and that they must issue the literal command `move-next` when they are ready to lock in the answer and proceed to the next question. The user answering or asking a follow-up is NOT permission to advance; treat any message that is not the exact `move-next` command as continued dialogue on the current question. Only advance to the next question when the user issues `move-next`.
3. Baseline Recommendations: For EVERY question asked, provide your own calculated, recommended answer or best-practice standard as a baseline for the user to react to.
4. Vocabulary Compliance: Strictly adhere to the taxonomy defined in the `design-vocab` skill. Frame all architectural questions using Module, Interface, Implementation, Depth, Seam, and Adapter. The terms component, service, unit, API, signature, and boundary are explicitly prohibited.

Structural Branches to Cover:
  - Business Domain & Persona Ownership: Mapping roles, system owners, and authentication scopes.
  - Module Topology & Solution Ecosystem: Mapping physical modules, critical interfaces, and external companion app integrations via explicit seams.
  - Automated Tech Stack Lifecycle: Evaluating implementation dependencies and library support windows against official EOL horizons for the current calendar year.
  - DevOps Pipeline & Governance: Explicitly auditing workflow rules, ticket state transitions, and delivery verification gates.
  - Operations & Incident Infrastructure: Documenting support tiering, SLAs, knowledge location targets, and breach protocols.
  - Data Layer Isolation & Security Models: Defining data dictionaries, storage schema alterations, and multi-tenancy isolation.
