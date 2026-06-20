---
name: agent-markup
description: Defines the strict token syntax, bracket-enclosed schema fields, and machine-readable value enumerations used for cross-agent parsing and automation.
---
Markup Syntax Rule:
  All machine-readable tokens must be enclosed in square brackets `[...]` to separate data fields from human-readable prose, enabling regex extraction and downstream automation.

Token Taxonomy & Allowed Values:
  [Auth: Scope]:
    Description: Defines the explicit access control tier or permissions matrix.
    Allowed Values: [Read, Write, Admin, None]
    
  [Risk: Level]:
    Description: Dictates technical debt, lifecycle urgency, or security vulnerability severity.
    Allowed Values: [Low, Medium, High, Critical]
    
  [Policy]:
    Description: Specifies the enforcement stance of a development rule or verification gate.
    Allowed Values: [Enforced, Advisory, Audit-Only]
