---
name: audit-application-health
description: Overarching audit orchestrator that runs the security/governance, blueprint-implementation, and test-coverage leaf audits, then synthesises a single client-facing Application Health Audit with an executive risk rollup, cross-cutting findings, a risk-vs-effort remediation roadmap, and versioned export-ready output.
dependencies:
  - audit-security-and-governance
  - audit-blueprint-implementation
  - audit-test-coverage
  - design-vocab
  - agent-markup
  - resolve-repository-platform
  - interview-me
---
1. PHASE 1 (Central Gate): Resolve contract availability + hosting platform ONCE, up front.
   - Run `resolve-repository-platform`; carry resolved platform/tooling into every leaf.
   - Check `docs/architecture/system-blueprint.md` + `docs/requirements/functional-requirements.md`.
   - Either missing → `interview-me` ONE decision: "Generate missing contract(s)?"
     - YES: hand off to `analyze-a-codebase`/`gather-requirements`, then proceed with all three leaves.
     - NO: contract-dependent leaves (blueprint-implementation, test-coverage) EXCLUDED. Security/governance ALWAYS runs.
2. PHASE 2 (Leaf Execution): Run available leaves in order:
     1. `audit-security-and-governance` (always)
     2. `audit-blueprint-implementation` (if contracts)
     3. `audit-test-coverage` (if contracts)
3. PHASE 3 (Cross-Cutting Synthesis): Correlate findings ACROSS leaves — e.g. bypassed Seam that is simultaneously a security exposure, or untested deep Interface that is also a PII data flow. Each correlated finding inherits highest `[Risk: Level]` of constituents.
4. PHASE 4 (Two-Register Report): Assemble versioned Application Health Audit per schema — plain-language executive layer over technical appendices. Write to non-overwriting versioned path.

Directives:
- Two-Register: Executive Summary + Remediation Roadmap = plain business/legal language (risk as regulatory/financial/reputational exposure). Technical appendices retain full `design-vocab` + `agent-markup`. Executive layer ONLY place `No Narrative Fluff` is relaxed; appendices stay terse.
- Composition: orchestrate + synthesise; do NOT re-run raw analysis leaves already performed. Preserve leaf findings faithfully — never soften, drop, or invent. Cross-cutting adds correlations on top; does not edit underlying tables.
- Risk-vs-Effort Roadmap: every finding pairs `[Risk: Level]` + `[Remediation: Effort]`. Order: high-risk/low-effort quick wins → high-risk/high-effort → remainder. All `Critical` first.
- Versioned Output: `docs/audit/application-health-audit-YYYYMMDD-rNN.md`. NEVER overwrite — corrected re-audit = new versioned file.
- Export-Ready Markdown per Output Portability Convention.
- Confidence Honesty: carry `[Confidence: Level]`. `Possible` = "requires verification" in both registers.

Schema:

# Application Health Audit (`[Scope: Health]`)
**Run:** [YYYYMMDD-rNN] | **Platform:** [resolved] | **Coverage:** [leaves executed] | **Excluded:** [sections skipped + reason]

## Executive Summary
*(Plain business/legal language. No design-vocab jargon, no tokens.)*
* **Overall posture:** [headline]
* **Critical exposures:** [headline Critical findings in business/regulatory/reputational terms]
* **Risk rollup:**

| Severity (`[Risk: Level]`) | Security | Data-Protection/GDPR | Implementation | Test Coverage | Total |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Critical | | | | | |
| High | | | | | |
| Medium | | | | | |
| Low | | | | | |

## Remediation Roadmap
*(Risk-vs-effort ordered; quick wins first. Plain language.)*
| Priority | Finding | Severity (`[Risk: Level]`) | Effort (`[Remediation: Effort]`) | Recommended Action |
| :--- | :--- | :--- | :--- | :--- |

## Cross-Cutting Findings
| Correlated Finding | Contributing Leaves | Combined Severity (`[Risk: Level]`) | Why It Compounds |
| :--- | :--- | :--- | :--- |

---

## Appendix A — Security & Governance Detail
*(Full `audit-security-and-governance` output, verbatim.)*

## Appendix B — Blueprint Implementation Detail
*(Full `audit-blueprint-implementation` output. Omit if excluded.)*

## Appendix C — Test Coverage Detail
*(Full `audit-test-coverage` output. Omit if excluded.)*