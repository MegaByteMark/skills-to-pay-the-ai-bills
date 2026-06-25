---
name: audit-application-health
description: Overarching audit orchestrator that runs the security/governance, blueprint-implementation, and test-coverage leaf audits, then synthesises a single client-facing Application Health Audit with an executive risk rollup, cross-cutting findings, a risk-vs-effort remediation roadmap, and versioned export-ready output.
dependencies:
  - audit-security-and-governance     # Always-on leaf; highest client weight
  - audit-blueprint-implementation    # Contract-gated leaf
  - audit-test-coverage               # Contract-gated leaf
  - design-vocab                      # Technical register vocabulary
  - agent-markup                      # [Risk: Level], [Confidence: Level], [Remediation: Effort], [Data: Classification]
  - resolve-repository-platform       # Resolves host/tooling ONCE up front so leaves never re-prompt
  - interview-me                      # Single-question gate resolution if contracts are missing
---

Operational Workflow:
1. PHASE 1 (Central Gate Resolution): Resolve contract availability AND hosting platform ONCE, up front, so the leaf skills never re-prompt mid-run.
     - Run the `resolve-repository-platform` protocol once and carry the resolved platform/tooling (or git-only fallback) into every leaf, so no leaf fires its own platform gate.
     - Check for `docs/architecture/system-blueprint.md` and `docs/requirements/functional-requirements.md`.
     - IF either is missing: trigger `interview-me` for ONE decision — "Generate the missing contract(s) before auditing?"
         - YES: hand off to `analyze-a-codebase` / `gather-requirements` to generate and persist the contract(s), then proceed with all three leaves.
         - NO: confirm to the user that the contract-dependent sections (blueprint-implementation and/or test-coverage) will be EXCLUDED from the health report, and proceed with the leaves that can run.
     - The security/governance leaf ALWAYS runs regardless of this decision, since it never gates on contracts.
2. PHASE 2 (Sequential Leaf Execution): Run the available leaves in this deterministic order and capture each one's full table output:
     1. `audit-security-and-governance` (always)
     2. `audit-blueprint-implementation` (if contracts present / generated)
     3. `audit-test-coverage` (if contracts present / generated)
3. PHASE 3 (Cross-Cutting Synthesis): Correlate findings ACROSS leaves to surface insight no single leaf can see — e.g. a bypassed Seam from the implementation audit that is simultaneously the security exposure, or an untested deep Interface that is also a PII data flow. Each correlated finding inherits the highest `[Risk: Level]` of its constituents.
4. PHASE 4 (Two-Register Report Assembly): Assemble the versioned Application Health Audit document per the schema below — a plain-language executive layer over technical appendices — and write it to a non-overwriting versioned path.

[Operational Directives]
- Two-Register Mandate: The Executive Summary and Remediation Roadmap are written in plain business/legal language a non-technical decision-maker grasps — risk framed as regulatory, financial, and reputational exposure (a Critical GDPR finding is stated as such, not as a CWE number). The technical appendices retain full `design-vocab` taxonomy and `agent-markup` tokens for the client's engineers. This Executive layer is the ONLY place the `No Narrative Fluff` directive is relaxed; the appendices stay terse.
- Composition, Not Re-Derivation: This skill orchestrates and synthesises; it does NOT re-run raw analysis the leaves already perform. Preserve each leaf's findings faithfully — never soften, drop, or invent findings during synthesis. The cross-cutting pass adds correlations on top; it does not edit the underlying tables.
- Risk-vs-Effort Roadmap: Every finding carried into the roadmap pairs `[Risk: Level]` with `[Remediation: Effort]`. Order the roadmap to lead with high-risk / low-effort quick wins, then high-risk / high-effort major remediations, then the remainder. Surface all `Critical` findings before anything else.
- Versioned, Non-Overwriting Output: Write to `docs/audit/application-health-audit-YYYYMMDD-rNN.md`, where `rNN` is the next available run number for that date (r01, r02, …). NEVER overwrite an existing report — a corrected re-audit must produce a new versioned file so remediation progress is provable across passes.
- Export-Ready Markdown: Keep the document export-clean for downstream PDF conversion under the organisation's document template/letterhead — standard heading hierarchy, no renderer-fragile constructs, tables that degrade gracefully. (Exact PDF/branding mechanism is defined separately.)
- Confidence Honesty: Carry each finding's `[Confidence: Level]` into the report. `Possible` findings are presented as "requires verification" in both registers — never asserted to the client as fact.

================================================================================
[Application Health Audit Output Schema]
================================================================================

# Application Health Audit (`[Scope: Health]`)
**Run:** [YYYYMMDD-rNN]  |  **Platform:** [GitHub | GitLab | Bitbucket | Self-hosted | None]  |  **Coverage:** [leaves executed]  |  **Excluded:** [sections skipped + reason, e.g. "Test Coverage — contracts not generated"]

---

## Executive Summary
*(Plain business/legal language. No design-vocab jargon, no tokens.)*
* **Overall posture:** [one-paragraph headline assessment]
* **Critical exposures:** [the headline Critical findings stated in business/regulatory/reputational terms]
* **Risk rollup:**

| Severity (`[Risk: Level]`) | Security | Data-Protection/GDPR | Implementation | Test Coverage | Total |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Critical | | | | | |
| High | | | | | |
| Medium | | | | | |
| Low | | | | | |

## Remediation Roadmap
*(Risk-vs-effort ordered; quick wins first. Plain language.)*
| Priority | Finding (plain language) | Severity (`[Risk: Level]`) | Effort (`[Remediation: Effort]`) | Recommended Action |
| :--- | :--- | :--- | :--- | :--- |

## Cross-Cutting Findings
*(Correlations spanning multiple leaves — the orchestrator's unique value.)*
| Correlated Finding | Contributing Leaves | Combined Severity (`[Risk: Level]`) | Why It Compounds |
| :--- | :--- | :--- | :--- |

---

## Appendix A — Security & Governance Detail
*(Full output of `audit-security-and-governance`, verbatim tables.)*

## Appendix B — Blueprint Implementation Detail
*(Full output of `audit-blueprint-implementation`. Omit entirely if excluded at the gate.)*

## Appendix C — Test Coverage Detail
*(Full output of `audit-test-coverage`. Omit entirely if excluded at the gate.)*
