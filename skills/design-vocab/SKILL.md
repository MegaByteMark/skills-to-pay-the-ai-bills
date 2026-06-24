---
name: design-vocab
description: Enforce a rigid, shared architectural vocabulary across all skills to eliminate semantic drift during code design, analysis, and restructuring.
references:
  - "A Philosophy of Software Design (John Ousterhout) - Core concepts of Module, Interface, Depth, Leverage, and Locality"
  - "Working Effectively with Legacy Code (Michael Feathers) - Concept of a Seam"
---
Core Philosophy:
  Leverage for callers. Locality for maintainers. Testability for everyone.

Strict Architectural Glossary:
  Module:
    Definition: Scale-agnostic unit possessing an interface and an implementation (e.g., function, class, package, or slice).
    Prohibited: Unit, component, service.
  Interface:
    Definition: The complete surface a caller must know (type signatures, invariants, ordering constraints, error modes, configuration, performance traits).
    Prohibited: API, signature.
  Implementation:
    Definition: The internal body of code within a module. Use over "Adapter" unless the seam itself is the topic.
  Depth:
    Definition: Ratio of behavior to interface complexity (high behavior behind a small interface).
  Seam:
    Definition: The physical location where an interface lives; allows altering behavior without editing the call site.
    Prohibited: Boundary.
  Adapter:
    Definition: A concrete artifact satisfying an interface at a seam. Describes the role/slot filled, not internal substance.
  Leverage:
    Definition: Caller-side benefit of depth (more capability per unit of learned interface).
  Locality:
    Definition: Maintainer-side benefit of depth (concentration of change, bugs, and knowledge in one place).
