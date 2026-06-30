---
name: audit-security-and-governance
description: Scans the physical codebase for security vulnerabilities, data-protection/GDPR governance exposures, and repository custody/visibility/licensing posture, anchoring every finding to OWASP/CWE/GDPR references with calibrated confidence. Runs standalone on raw code or enriches against the blueprint/FDS when present.
dependencies:
  - design-vocab  # Uses Module, Interface, Implementation, Seam, and Adapter definitions
  - agent-markup  # Pulls [Risk: Level], [Auth: Scope], [Data: Classification], [Confidence: Level], [Remediation: Effort]
  - resolve-repository-platform  # Resolves host/tooling before any platform-specific repository lookup
---

Operational Workflow:
1. PHASE 1 (Optional Contract Enrichment & Code Ingestion): This skill NEVER aborts on missing contracts — it is the one leaf audit safe to run cold on a discovery or sales engagement.
     - Attempt to read `docs/architecture/system-blueprint.md` and `docs/requirements/functional-requirements.md`.
     - IF present: ingest declared `[Auth: Scope]` tiers, data-isolation/multi-tenancy rules, and seam topology to detect drift between intended and actual security posture.
     - IF absent: proceed standalone against the physical codebase and emit a single notice that findings were produced without a contract baseline (reduced context, structural intent inferred from code only).
     - Always scan the physical repository regardless of contract availability.
2. PHASE 2 (Security & Data-Protection Scan): Walk the physical codebase, configuration, dependency manifests, and data stores.
     - Security pass: inspect input handling, authentication/authorization paths, injection surfaces, cryptographic usage, session handling, access control at seams, and configuration/secrets hygiene.
     - Governance pass: build the PII inventory, map data flows across seams to third parties, and inspect lawful-basis/consent, retention/erasure, cross-border transfer, sub-processor exposure, audit-logging/accountability, and breach-readiness.
     - Repository posture pass: assess how and where the repository is hosted and held, independent of its contents (see Repository Posture & Custody Rule).
     - Supplementary flags: hardcoded secrets committed to the repository, and dependency-license exposure (see Copyleft Contamination Rule).
3. PHASE 3 (Evidence-Bound Synthesis): Classify and tabulate findings into the schema below. Every finding is anchored to observed code, a recognized standard reference, a severity, and a calibrated confidence, with a prescribed remediation and effort estimate.

## Operational Directives
- Vocabulary Compliance: Strictly adhere to the `design-vocab` taxonomy. Describe exposures in terms of leaky Interfaces, bypassed Seams, and unguarded Modules. The terms component, service, unit, API, signature, and boundary are prohibited.
- Standards Anchoring: Every security finding MUST cite its OWASP Top 10 (2021) category AND a CWE identifier. Every governance finding MUST cite the specific GDPR article or principle it implicates (e.g. Art. 5 data minimisation, Art. 9 special-category data, Art. 32 security of processing, Art. 33 breach notification). Only real, existing CWE/CVE/GDPR identifiers may be cited — never fabricate an identifier.
- Evidence-Bound Findings: Every finding MUST anchor to a real file path plus the concrete observed pattern that evidences it. Any suspicion that cannot be tied to actual observed code is DROPPED, never speculated into the report. Calibrate `[Confidence: Level]`: Confirmed = directly observed exploitable pattern; Probable = strong indicator missing one corroborating link; Possible = heuristic signal needing human verification. Phrase every `Possible` finding as "requires verification" — never assert it. For a client-facing audit, a missed finding is recoverable; a fabricated accusation is not.
- Copyleft Contamination Rule: When scanning dependency licenses, explicitly flag copyleft and network-copyleft licenses (GPL, AGPL, LGPL and variants) where they risk contaminating the project's distribution or commercial model. State the obligation triggered (e.g. AGPL source-disclosure on network use, GPL derivative-work copyleft, LGPL dynamic-linking conditions) rather than merely naming the license.
- Repository Posture & Custody Rule: Independently of code contents, assess where and how the repository is held. This is OUTBOUND exposure of the client's own asset, distinct from the inbound dependency exposure of the Copyleft Contamination Rule. First resolve the hosting platform and tooling via the `resolve-repository-platform` protocol — never assume GitHub. Inspect three axes:
     - Custody/Ownership: derive the hosting namespace from `git remote -v`; a personal/individual account rather than a company or organization namespace is a custody, key-person/bus-factor, and access-control governance risk — the client may not control their own asset. Confirm account-vs-organization via the resolved platform's owner lookup (adapter map) where available.
     - Visibility: a repository that is public for a project never scoped as open-source exposes proprietary source and anything committed to it. Repository Visibility is NOT knowable from a local clone — confirm via the resolved platform's visibility lookup or explicit human confirmation; otherwise report as `Possible — requires verification`, never asserted.
     - Licensing posture: flag absence of a `LICENSE` file (directly observable from the clone). On a public repository this is a compounded legal exposure — default "all rights reserved" copyright while the source is openly readable and third parties may wrongly assume reuse rights. Conversely, an unexpected open-source license on proprietary work is itself a finding.
   Calibrate confidence per axis: `LICENSE` absence = `Confirmed`; account ownership = `Confirmed`/`Probable` from remote namespace and platform owner lookup; visibility = `Confirmed` only with platform/human confirmation, else `Possible`. Cite GDPR where public exposure also implicates personal data at rest in the repository (e.g. Art. 32 security of processing); otherwise frame as IP/confidentiality governance without forcing a GDPR article.
- Data Classification: Tag every data store, flow, and PII element with `[Data: Classification]`. Treat `Special-Category` (GDPR Art. 9) data as the highest governance priority.
- Table-First Reporting: Deliver findings in high-density Markdown tables structured exactly like the schema below.
- No Narrative Fluff: Keep output technical, direct, and actionable.

## Security & Governance Audit Output Schema

### 0. Scan Context (`[Scope: Security-Governance]`)
* **Baseline:** [Contract-enriched | Standalone — no contract baseline, reduced context]
* **Platform:** [GitHub | GitLab | Bitbucket | Self-hosted:<host> | None] (`[Confidence: Level]`) — [tooling authenticated | git-only fallback]
* **Surface scanned:** [modules / config / dependency manifests / data stores / repository posture covered]

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
* **Hardcoded Secrets:** List credentials, tokens, or keys committed to the repository, with file evidence and `[Risk: Level]`.
* **Copyleft / License Exposure:** Flag GPL/AGPL/LGPL (and variant) dependencies that risk contaminating the distribution or commercial model, naming the specific obligation triggered and `[Risk: Level]`.

### 5. Repository Posture & Custody Exposure
*(Platform resolved via `resolve-repository-platform`; record the resolution in §0 Scan Context.)*
* **Custody / Ownership:** Hosting namespace from `git remote -v` and (where available) the platform owner lookup; flag personal/individual-account custody of a client asset with `[Risk: Level]` and `[Confidence: Level]`.
* **Visibility:** Repository Visibility and whether the project was scoped as open-source; flag proprietary source exposed on a public repository. State the evidence source (platform lookup / human-confirmed / unverified) and calibrate `[Confidence: Level]` accordingly — never assert public visibility without confirmation.
* **Licensing Posture:** Presence/absence of a `LICENSE` file and its fit with the intended commercial model; flag missing-license-on-public exposure or an unexpected open-source license on proprietary work, with `[Risk: Level]`.

### 6. Contract Drift (Enriched Runs Only)
* **Auth Scope Drift:** Code paths where the implemented `[Auth: Scope]` diverges from the blueprint/FDS-declared scope.
* **Data-Isolation Drift:** Multi-tenancy or data-isolation rules declared in the contract but missing or incorrectly applied in practice.
* *(Omit this section entirely on standalone runs.)*
