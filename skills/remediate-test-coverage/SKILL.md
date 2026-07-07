---
name: remediate-test-coverage
description: Remediation counterpart to audit-test-coverage. Runs the audit to obtain an authoritative gap set, reconciles it against the Minimum Verification Surface Baseline, presents a risk-vs-effort remediation plan for per-tier human approval, then writes Interface-verifying tests with fake/mock Adapters at Seams, executes them, iterates to green, escalates any production defects it uncovers, and emits a versioned closure report. Builds the minimum sufficient verification surface — never gold-plates.
dependencies:
  - audit-test-coverage
  - design-vocab
  - agent-markup
  - interview-me
  - detect-test-harness
---
1. PHASE 1 (Gap Acquisition): Invoke `audit-test-coverage`; consume full output as single source of truth. Inherit audit's tiered missing-contract gate verbatim. Carry forward audit's fidelity marking into every artefact.
2. PHASE 2 (Minimum-Surface Reconciliation): Classify every gap into Minimum Verification Surface Baseline tier (M1–M5, C1–C6). Prune:
   - KEEP: absent Mandatory Floor + absent Conditional with fired trigger.
   - DROP: Conditional with absent trigger → "Excluded — trigger absent" + rationale.
   - DROP: proposed test coupling to volatile Implementation rather than stable Interface.
   Output: minimum sufficient remediation set.
3. PHASE 3 (Remediation Plan & Per-Tier Approval): Render risk-vs-effort plan (schema below). Announce approval protocol: walk ONE surface tier at a time; user must issue literal `move-next` to authorise writing that tier's tests. Other messages = continued discussion. User may `skip` (deferred, recorded) or `stop`. Writing before `move-next` prohibited.
4. PHASE 4 (Per-Tier Implementation):
   a. Harness: inherit audit's Resolution Record; if absent, resolve via `detect-test-harness`. Never assume.
   b. Author at Interface, fake at Seam: verify target Module's Interface; inject fake/mock Adapters at Seams per blueprint §6, using resolved framework's native test-double idiom. Prefer fakes; mocks only for genuine interaction ordering.
   c. Execute & iterate: run authored tests, then full suite. Fix flaky/incorrect tests YOU authored. Re-run until pass or genuine production defect isolated.
   d. Escalate: test cannot pass without production change → STOP. Record as discovered defect. Offer fix as separate approved action.
   Return to Phase 3 for next tier.
5. PHASE 5 (Closure & Report): Re-run `audit-test-coverage` to prove gap closure as before/after delta. Write versioned report (schema below). Deferred/excluded gaps listed with rationale.

Directives:
- Minimum-Sufficient: smallest verification surface covering high-Depth Interfaces + mandatory floor. Every kept item traces to Mandatory Floor or fired Conditional. When in doubt, exclude + record.
- Verify-at-Interface, Fake-at-Seam: verification on stable Interfaces, never volatile Impls. Test doubles only at defined Seams. Prefer fakes over mocks. Do not mock what you don't own — wrap behind Seam, fake wrapper.
- No Green-Washing: never weaken/skip/delete assertions to pass; never delete/disable failing test; never relax production to satisfy test. Test that cannot pass without production change = discovered defect to escalate.
- Don't-Break-Reality: existing passing tests are protected. De-brittling audit-flagged test allowed only with approval. Full suite before + after; must end green or every residual failure explained.
- Approval-Before-Write: literal `move-next` only trigger. State at Phase 3 start.
- Harness Discovery: resolve via `detect-test-harness`; confirm with ONE `interview-me` question when ambiguous. Never add new test dependency without approval.
- Strict `design-vocab`: Module, Interface, Implementation, Depth, Seam, Adapter. Avoid unit/component/service/API/boundary except naming physical folder path.
- Versioned Output: `docs/audit/test-remediation-YYYYMMDD-rNN.md`. NEVER overwrite.

Schema:

# Test Coverage Remediation
**Run:** [YYYYMMDD-rNN] | **Audit Source:** [audit run ref] | **Baseline Fidelity:** [Persisted | Inferred: Unverified]

## 1. Remediation Plan (Risk-vs-Effort)
| Order | Baseline Tier | Target Interface / Seam / Flow | Gap Severity (`[Risk: Level]`) | Effort (`[Remediation: Effort]`) | Fake/Mock Adapters Required | Approval State (Pending/Approved/Skipped) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |

## 2. Minimum Surface Coverage Ledger (Before → After)
| Baseline Tier | Coverage Before | Action Taken | Coverage After | Verifications Added | Requirement IDs Now Covered | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |

## 3. Deliberately Excluded / Deferred
| Surface | Excluded or Deferred | Rationale | Reconsider-When Trigger |
| :--- | :--- | :--- | :--- |

## 4. Discovered Production Defects (Escalations)
| Defect | Implicated Module / Interface | Evidence (failing verification) | Severity (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) | Proposed Fix (requires separate approval) |
| :--- | :--- | :--- | :--- | :--- | :--- |

## 5. Closure Verification (Re-Audit Delta)
* **Gaps closed:** [count + tiers]
* **Gaps remaining (deferred/excluded):** [count + §3 ref]
* **Suite status:** [green | residual failures explained]
* **Net minimum-floor compliance:** [M1–M5 / C-triggered status before→after]