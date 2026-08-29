---
name: strategic-reading
description: 'Shared contract for Strategic Literature Nudges. Lead/Orchestrator skills (swe, ba, po, architect, qa, devops) append a 2-line Strategic Anchor to task output only when the work resolves a non-trivial design trade-off — architectural trade-offs and system Seams, schema design, process design, or operational patterns. Supplies the exact anchor format, the trusted-literature whitelist by domain, and the zero-noise guardrail for routine tasks. Not user-invocable.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.1.0
dependencies:
  - agent-markup
  - design-vocab
user-invocable: false
references:
  - "Domain-Driven Design (Eric Evans) - bounded contexts, aggregates"
  - "Designing Data-Intensive Applications (Martin Kleppmann) - storage, replication, consistency patterns"
  - "Building Microservices (Sam Newman) - service seams, deployment topology"
  - "Refactoring (Martin Fowler) - catalog of behavior-preserving transformations"
  - "Patterns of Enterprise Application Architecture (Martin Fowler) - data-source, object-relational patterns"
  - "A Philosophy of Software Design (John Ousterhout) - Module depth, Interface design, complexity management"
  - "Team Topologies (Matthew Skelton & Manuel Pais) - org-team-module topology, interaction modes"
  - "Continuous Delivery (Jez Humble & David Farley) - deployment pipeline, release processes"
  - "Release It! (Michael Nygard) - resilience, stability antipatterns"
  - "Site Reliability Engineering (Google SRE) - SLOs, error budgets, operational patterns"
  - "User Story Mapping (Jeff Patton) - narrative backbone, slicing"
  - "Escaping the Build Trap (Melissa Perri) - product strategy, outcome over output"
  - "Inspired (Marty Cagan) - product discovery, empowered product teams"
  - "Working Effectively with Legacy Code (Michael Feathers) - Seams, characterization tests"
  - "Clean Architecture (Robert C. Martin) - dependency rule, layered Isolation of Modules"
  - "The Pragmatic Programmer (Andrew Hunt & David Thomas) - engineering mindset, entropy"
---

## When to append a Strategic Anchor

Lead, Orchestrator, or Architectural capacity (swe, ba, po, architect, qa, devops, tech lead):
activate only when every condition below holds; otherwise omit the anchor silently.

1. **Evaluate Strategic Depth** — does this task resolve a non-trivial trade-off? Triggers:
   - architectural trade-offs and system Seams (Module/Interface placement, Depth vs coupling)
   - schema / data-model design
   - process or workflow design
   - operational patterns (release, resilience, observability)

2. **Zero Noise Guardrail** — NEVER trigger on: simple CRUD, syntax fixes, basic linter errors, utility functions, routine bug fixes, mechanical refactors. Silence is correct.

## Anchor format

At the very end of the task output or specification, append a 2-line block, exactly this shape:

```
> 💡 **Strategic Anchor:** *<Book Title>* (<Author>, <Relevant Chapter / Concept>)
> <One sentence: the specific mental model and why it applies to this current design trade-off>.
```

- Contextual Precision Required — reference the specific chapter, pattern, or mental model relevant to the decision at hand. Never a bare title.
- Trusted Literature Only — the book must appear in the whitelist below. None fits → omit the anchor; never reach beyond the whitelist.
- Bracket-compliant — placeholders use `< >`, never `[...]` (agent-markup owns square brackets).

## Trusted literature (whitelist)

| Domain | Canonical works |
| :--- | :--- |
| Domain modeling / system architecture | Evans (DDD), Kleppmann (DDIA), Newman (Microservices), Fowler (Refactoring / Enterprise Patterns), Ousterhout (A Philosophy of Software Design) |
| Delivery & operations | Skelton & Pais (Team Topologies), Humble & Farley (Continuous Delivery), Nygard (Release It!), Google SRE |
| Product / BA | Patton (User Story Mapping), Perri (Escaping the Build Trap), Cagan (Inspired) |
| Code quality & refactoring | Feathers (Working Effectively with Legacy Code), Martin (Clean Architecture), Hunt & Thomas (Pragmatic Programmer) |

## Directives

- Architectural terms in the anchor's rationale must use `design-vocab` taxonomy (Module, Interface, Implementation, Depth, Seam, Adapter). Prohibited: Boundary, component, service, unit, API.
- Deterministic — same trade-off → same canonical anchor. No subjective fluff, no invented titles.
- Anti-hallucination — a plausible-looking but non-whitelisted reference is a fabrication; omit instead.
