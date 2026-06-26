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
    Note: Written bare as [Policy] as a schema/column marker; written in instance shorthand as [Policy: Enforced] etc. when tagging a concrete rule inline (parallel to [Risk: High]).

  [Data: Classification]:
    Description: Classifies the sensitivity tier of data handled at a code path, store, or seam. Drives PII inventory and data-protection findings.
    Allowed Values: [Public, Internal, Confidential, PII, Special-Category]
    Note: Special-Category maps to GDPR Article 9 sensitive data (health, biometrics, race, religion, sexual orientation, etc.) and carries the strictest handling obligations.

  [Confidence: Level]:
    Description: Declares how certain a finding is, calibrated against directly observed code evidence. Curbs speculative or hallucinated findings.
    Allowed Values: [Confirmed, Probable, Possible]
    Note: Confirmed = directly observed exploitable pattern; Probable = strong indicator missing one corroborating link; Possible = heuristic signal requiring human verification. Possible findings must be phrased as "requires verification", never asserted.

  [Remediation: Effort]:
    Description: Estimates the implementation cost to resolve a finding, enabling risk-vs-effort prioritisation in remediation roadmaps.
    Allowed Values: [Low, Medium, High]

  [Competency: Level]:
    Description: Declares a human's demonstrated ability in a single skill area, calibrated against observed work — not self-report. Shared across every skill that teaches, assesses, or hands work to a human so the same baseline travels with the person.
    Allowed Values: [Not-Ready, Paired, Guided, Solo]
    Note: Ordered weakest-to-strongest. Solo = wrote comparable work unaided; Guided = succeeds with a full spec/acceptance criteria; Paired = succeeds only with a worked scaffold and step-by-step prompts; Not-Ready = cannot currently do it OR cannot explain what it does. A claim from self-report is provisional until corroborated by observed work; pair with [Confidence: Level] to express calibration certainty. Owned/persisted by the competency-profile skill.

  [Inferred: Unverified]:
    Description: Marks output derived from an ephemeral, in-context baseline that was reconstructed rather than read from a persisted contract. Signals reduced fidelity to downstream readers and automation.
    Allowed Values: [true]
    Note: Applied when a skill proceeds on a simulated FDS baseline instead of a saved contract. Never applied to blueprint-drift findings, which require a persisted, non-self-derived baseline.

  [Doc: Archetype]:
    Description: Selects the documentation archetype a generation run targets, binding it to a fixed output path.
    Allowed Values: [QuickStart, Technical, Troubleshooting, Installation]
    Note: Written in shorthand as [Doc: QuickStart] etc. Path bindings are owned by the document-a-codebase skill.

  [Scope: Artefact]:
    Description: Declares the artefact archetype an output document represents, enabling downstream tooling to route or aggregate it.
    Allowed Values: [Release, Security-Governance, Health]
    Note: Written in shorthand as [Scope: Release] etc. This enumeration grows as new report-producing skills are added; extend it here rather than introducing undeclared scope values.

Output Portability Convention:
  All client-facing artefacts (FDS, system blueprint, audit reports, and documentation guides) must be authored as export-clean Markdown suitable for downstream PDF conversion under an organisation document template/letterhead. Use a standard heading hierarchy, avoid renderer-fragile constructs, and ensure tables degrade gracefully. The concrete PDF/branding mechanism is defined outside this skill.
