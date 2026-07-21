---
name: audit-security-and-governance
description: Scans the physical codebase for security vulnerabilities, data-protection/GDPR governance exposures, and repository custody/visibility/licensing posture, anchoring every finding to OWASP/CWE/GDPR references with calibrated confidence. Runs standalone on raw code or enriches against the blueprint/FDS when present.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - design-vocab
  - agent-markup
  - resolve-repository-platform
---
1. PHASE 1 (Contract Enrichment): NEVER abort on missing contracts.
   - Attempt `docs/architecture/system-blueprint.md` + `docs/requirements/functional-requirements.md`.
   - Present → ingest `[Auth: Scope]`, data-isolation, seam topology to detect posture drift.
   - Absent → proceed standalone, emit notice "no contract baseline, reduced context".
   - Always scan physical repo regardless.
2. PHASE 2 (Scan):
   - Security: input handling, auth/authz paths, injection surfaces, cryptographic usage, session handling, access control at seams, config/secrets hygiene.
   - Governance: PII inventory, data flows across seams to third parties, lawful-basis/consent, retention/erasure, cross-border transfer, sub-processor exposure, audit-logging, breach-readiness.
   - Repository posture: custody/ownership, visibility, licensing posture (see Repository Posture Rule).
   - Supplementary: hardcoded secrets, copyleft license exposure (see Copyleft Rule).
3. PHASE 3 (Synthesis): Tabulate findings. Every finding anchored to observed code + standard reference + severity + calibrated confidence + prescribed remediation + effort.

Directives:
- Strict `design-vocab`: leaky Interfaces, bypassed Seams, unguarded Modules. Prohibited: component, service, unit, API, signature, boundary.
- Standards Anchoring: Security → OWASP Top 10 (2021) category + CWE. Governance → specific GDPR article/principle. Only real identifiers — never fabricate.
- Evidence-Bound: Every finding anchored to real file path + observed pattern. Suspicion without code evidence = DROPPED. `[Confidence: Level]`: Confirmed = directly observed exploitable pattern; Probable = strong indicator missing one link; Possible = requires verification.
- Copyleft Contamination: Flag GPL/AGPL/LGPL (and variants) where they risk contaminating distribution/commercial model. State obligation triggered (e.g. AGPL source-disclosure, GPL derivative-work copyleft, LGPL dynamic-linking conditions).
- Repository Posture (three axes):
  - Custody/Ownership: derive hosting namespace from `git remote -v`. Personal/individual account (not company/org) = custody/bus-factor/access-control governance risk. Confirm via resolved platform's owner lookup where available.
  - Visibility: public repo for not-open-source project = proprietary exposure. NOT knowable from local clone — confirm via platform lookup or explicit human confirmation; else `Possible — requires verification`. Never assert.
  - Licensing: `LICENSE` absence = `Confirmed` (directly observable). Public + no license = "all rights reserved" while openly readable. Unexpected open-source license on proprietary work = finding.
  - Calibrate: LICENSE absence = `Confirmed`; account ownership = `Confirmed`/`Probable`; visibility = `Confirmed` only with confirmation, else `Possible`.
- Data Classification: tag every store/flow/PII with `[Data: Classification]`. `Special-Category` = highest priority.
- Table-First. No Narrative Fluff.

Schema:

### 0. Scan Context (`[Scope: Security-Governance]`)
* **Baseline:** [Contract-enriched | Standalone] * **Platform:** [resolved] (`[Confidence: Level]`) — [authenticated | git-only]
* **Surface scanned:** [modules / config / dependency manifests / data stores / repository posture]

### 1. Security Vulnerability Findings
| Finding | Affected Module / File Evidence | OWASP (2021) | CWE | Severity (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) | Recommended Remediation | Effort (`[Remediation: Effort]`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |

### 2. Data-Protection & GDPR Governance Findings
| Finding | Data Flow / Module Evidence | Data (`[Data: Classification]`) | GDPR Article / Principle | Severity (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) | Recommended Remediation | Effort (`[Remediation: Effort]`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |

### 3. PII Inventory & Data-Flow Map
| Data Element | Classification (`[Data: Classification]`) | Storage / Module Location | Crosses Seam To (third party / sub-processor) | Lawful Basis Evident? | Retention / Erasure Evident? |
| :--- | :--- | :--- | :--- | :--- | :--- |

### 4. Secrets & Dependency-License Exposure
* **Hardcoded Secrets:** credentials/tokens/keys committed to repo, file evidence + `[Risk: Level]`.
* **Copyleft / License Exposure:** GPL/AGPL/LGPL dependencies, obligation triggered + `[Risk: Level]`.

### 5. Repository Posture & Custody Exposure
* **Custody/Ownership:** namespace + owner lookup; flag personal account custody.
* **Visibility:** public/private + evidence source; flag proprietary exposure.
* **Licensing Posture:** LICENSE presence/fit; flag missing-on-public or unexpected open-source license.

### 6. Contract Drift (Enriched Runs Only)
* **Auth Scope Drift:** code paths where `[Auth: Scope]` diverges from blueprint/FDS.
* **Data-Isolation Drift:** declared rules missing/incorrectly applied.
*(Omit entirely on standalone runs.)*