---
name: remediate-test-coverage
description: Remediation counterpart to audit-test-coverage. Runs the audit to obtain an authoritative gap set, reconciles it against the Minimum Verification Surface Baseline, presents a risk-vs-effort remediation plan for per-tier human approval, then writes Interface-verifying tests with fake/mock Adapters at Seams, executes them, iterates to green, escalates any production defects it uncovers, and emits a versioned closure report. Builds the minimum sufficient verification surface — never gold-plates.
dependencies:
  - audit-test-coverage         # Authoritative gap source + owner of the Minimum Verification Surface Baseline
  - design-vocab                # Interface / Implementation / Seam / Adapter / Depth taxonomy
  - agent-markup                # [Risk: Level], [Confidence: Level], [Remediation: Effort], [Policy]
  - interview-me                # Drives the move-next per-tier approval gate and any harness-confirmation question
  - detect-test-harness         # Shared protocol for resolving the runner/framework + native test-double idiom
---

## Operational Workflow
1. PHASE 1 (Gap Acquisition — Do Not Re-Derive): Invoke `audit-test-coverage` and consume its full output as the single source of truth: the Interface/Seam alignment matrix, the Functional Requirement matrix, the Minimum Verification Surface Compliance matrix, and the coupling/brittleness deficits. Inherit the audit's tiered missing-contract gate verbatim — NEVER run a second contract gate of your own. Carry forward whatever fidelity marking the audit applied (e.g. `[Inferred: Unverified]` under an ephemeral baseline) into every artefact you produce.
2. PHASE 2 (Minimum-Surface Reconciliation & Anti-Over-Engineering Filter): Classify every confirmed gap into its Minimum Verification Surface Baseline tier (M1–M5, C1–C6 — owned by `audit-test-coverage`). Then prune:
     - KEEP: absent Mandatory Floor surfaces and absent Conditional surfaces whose trigger fired.
     - DROP: any surface whose Conditional trigger is absent, recording it explicitly as "Excluded — trigger absent" with a one-line rationale. Coverage maximalism is a defect.
     - DROP: any proposed test that would couple to a volatile Implementation rather than a stable Interface; prefer a single high-Depth Interface verification over many Implementation-coupled ones.
   The output of this phase is the minimum sufficient remediation set, not the maximal one.
3. PHASE 3 (Remediation Plan & Per-Tier Approval Gate): Render ONE prioritised risk-vs-effort remediation plan (schema below). Before any test code is written, announce the approval protocol: explain that you will walk the plan ONE surface tier at a time and that the user must issue the literal command `move-next` to authorise writing that tier's tests; any other message is continued discussion (re-prioritise, defer a tier, adjust scope), NOT authorisation. The user may also `skip` a tier (deferred, recorded in the report) or `stop`. Writing a test before its tier's `move-next` is strictly prohibited.
4. PHASE 4 (Per-Tier Implementation & Iteration to Green): For the authorised tier only:
     a. Harness Discovery: inherit the harness Resolution Record carried forward by `audit-test-coverage`; if absent, resolve it now via `detect-test-harness` (framework, run command, layout, native test-double idiom). Do NOT assume a framework. If none exists or the choice is ambiguous, `detect-test-harness` asks exactly ONE `interview-me` question to confirm the runner before scaffolding a harness; never introduce a new framework or mocking library silently.
     b. Author at the Interface, fake at the Seam: write tests that verify the target Module's Interface (behaviour, invariants, error modes), injecting fake/mock Adapters at its Seams per blueprint §6, using the resolved framework's native test-double idiom (per `detect-test-harness`). Prefer fakes; use mocks only to verify interaction ordering that genuinely matters.
     c. Execute & iterate: run the authored tests, then the full suite. Fix flaky or incorrect tests YOU authored. Re-run until the authored tests pass or a genuine production defect is isolated.
     d. Escalate, never green-wash: if a test cannot pass without changing production code, STOP — record it as a discovered defect (do not silently patch production, do not weaken the assertion). Offer the fix as a separate, explicitly-approved action.
   Return to Phase 3 for the next tier until the plan is exhausted, skipped, or stopped.
5. PHASE 5 (Closure Verification & Report): Re-run `audit-test-coverage` to prove gap closure as a before/after delta, then write the versioned remediation report (schema below). Residual gaps that were deferred or excluded-as-over-engineering are listed with rationale so the next pass is provable.

## Operational Directives
- Minimum-Sufficient Mandate: The goal is the smallest verification surface that covers high-Depth Interfaces and the mandatory floor — NOT maximal coverage. Every kept item must trace to a Mandatory Floor tier or a fired Conditional trigger. When in doubt, exclude and record the rationale rather than build.
- Verify-at-the-Interface, Fake-at-the-Seam: Place verification on stable Interfaces, never on volatile Implementation bodies. Test doubles live only at defined Seams. Prefer fake Adapters over mocks (lower brittleness); reserve mocks for genuine interaction-ordering verification. Do not mock what you do not own — wrap the foreign Interface behind a Seam, then fake the wrapper.
- No Green-Washing: Never weaken, skip, or delete assertions to force a pass; never delete or disable a failing test to clear the suite; never relax production behaviour purely to satisfy a test. A test that cannot pass without a production change is a discovered defect to escalate, not a test to soften.
- Don't-Break-Reality: Treat existing passing tests as protected. De-brittling a test the audit flagged is permitted only with explicit approval. Run the full suite before and after; the run must end green or every residual failure must be explained and attributed.
- Approval-Before-Write Contract: The ONLY trigger that authorises writing a tier's tests is the literal `move-next` command. State this at the start of Phase 3. Answering a user question is not authorisation; if in doubt, stay on the current tier and wait.
- Harness Discovery, Not Assumption: Resolve the runner/framework through `detect-test-harness` (inheriting the audit's Resolution Record when present); confirm via its single `interview-me` question when ambiguous. Never add a new test dependency, framework, or mocking library without approval.
- Vocabulary Compliance: Strictly adhere to `design-vocab`. Use Module, Interface, Implementation, Depth, Seam, Adapter. Avoid unit/component/service/API/boundary except when naming a physical folder path (e.g. `tests/unit`).
- Markup Compliance: Restrict bracket tokens to the `agent-markup` enumerations. Pair every roadmap item with `[Risk: Level]` and `[Remediation: Effort]`; carry each finding's `[Confidence: Level]` through unchanged.
- Versioned, Non-Overwriting Output: Write the report to `docs/audit/test-remediation-YYYYMMDD-rNN.md`, where `rNN` is the next free run number for that date. NEVER overwrite a prior report — remediation progress must stay provable across passes.

## Test Remediation Output Schema

# Test Coverage Remediation
**Run:** [YYYYMMDD-rNN]  |  **Audit Source:** [audit run reference]  |  **Baseline Fidelity:** [Persisted | Inferred: Unverified]

## 1. Remediation Plan (Risk-vs-Effort)
*(Presented in Phase 3 for per-tier approval. Lead with high-risk / low-effort quick wins. Each row is one approvable surface tier.)*
| Order | Baseline Tier | Target Interface / Seam / Flow | Gap Severity (`[Risk: Level]`) | Effort (`[Remediation: Effort]`) | Fake/Mock Adapters Required | Approval State (Pending / Approved / Skipped) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |

## 2. Minimum Surface Coverage Ledger (Before -> After)
*(Proves movement against the Minimum Verification Surface Baseline.)*
| Baseline Tier | Coverage Before | Action Taken | Coverage After | Verifications Added | Requirement IDs Now Covered | Confidence (`[Confidence: Level]`) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |

## 3. Deliberately Excluded / Deferred (Anti-Over-Engineering Ledger)
*(The explicit record of what was NOT built and why — equally important as what was. Excluded = trigger absent or Implementation-coupling avoided; Deferred = user skipped.)*
| Surface | Excluded or Deferred | Rationale | Reconsider-When Trigger |
| :--- | :--- | :--- | :--- |

## 4. Discovered Production Defects (Escalations)
*(Tests that could not pass without a production change. Reported, never silently patched.)*
| Defect | Implicated Module / Interface | Evidence (failing verification) | Severity (`[Risk: Level]`) | Confidence (`[Confidence: Level]`) | Proposed Fix (requires separate approval) |
| :--- | :--- | :--- | :--- | :--- | :--- |

## 5. Closure Verification (Re-Audit Delta)
*(Result of re-running `audit-test-coverage` after remediation.)*
* **Gaps closed:** [count + tiers]
* **Gaps remaining (deferred/excluded):** [count + reference to §3]
* **Suite status:** [green | residual failures explained]
* **Net minimum-floor compliance:** [M1–M5 / C-triggered status before -> after]
