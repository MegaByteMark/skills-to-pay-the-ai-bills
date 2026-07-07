---
name: create-bug-report
description: Generates a complete, high-signal bug report by auto-capturing every field it can evidence from the codebase, git, the runtime, and any error text the user pasted, then interviewing the user one question at a time (via interview-me) for only the human-centric gaps that cannot be derived. Renders the report against a fixed schema and optionally creates or amends it as a Work Item in the resolved tracking platform. Evidence-first and anti-hallucination: every field is either evidenced, user-answered, or an explicit "Unknown — requires verification" — never invented.
dependencies:
  - interview-me
  - resolve-repository-platform
  - agent-markup
argument-hint: "[title | pasted error/stack trace]  # optional seed; mode (create|amend) auto-detected from tracker"
user-invocable: true
---
Maximise auto-capture; minimise questions. NEVER fabricate a log line, version, or repro step. Unevidenced/unanswered = `Unknown — requires verification`.

1. PHASE 0 (Input & Mode): Take seed (title or pasted error/stack trace). Once platform resolved, if Work Item already carries this report's stable-ID marker → `amend`; else `create`. Never blind-create duplicate.
2. PHASE 1 (Evidence Auto-Capture): Silently derive: pasted error/stack → Diagnostics; manifests + `git describe`/tags/HEAD → Version; declared runtime/OS from manifests, lockfiles, CI config → Environment (host OS is agent's machine, not user's — only assert user's when stated). Tag each `[Confidence: Confirmed|Probable|Possible]`. Possible = requires-verification. Classify diagnostics `[Data: Classification]` and redact secrets/credentials/PII.
3. PHASE 2 (Gap Interview): For ONLY still-missing fields (Expected, Actual, exact repro steps, evidence links) → `interview-me`: one question at a time, each with recommended answer baseline. Never re-ask what Phase 1 evidenced.
4. PHASE 3 (Triage): Derive Severity `[Risk: Level]` from impact + frequency; record reproducibility (always/intermittent) and regression status if git evidence supports. Name suspected Seam only when code evidence points to one, tagged `[Confidence: Level]` — omit rather than speculate.
5. PHASE 4 (Render): Compile into Bug Report Schema. Append stable-ID marker footer.
6. PHASE 5 (Preview & Write): Run `resolve-repository-platform`; present report + intended action (create|amend, platform). Require explicit confirmation. On confirm, create/amend via adapter row. No authenticated CLI → emit Markdown. Report resulting Work-Item reference.

Directives:
- Evidence-First, Interview-Last.
- No Invention: unevidenced + unanswered = `Unknown — requires verification`.
- Honest Provenance: tag auto-derived `[Confidence: Level]`; ephemeral-baseline fields `[Inferred: Unverified]`.
- Redaction: classify diagnostics, strip secrets/tokens/PII.
- Reproduction Discipline: concrete, ordered steps. Unknown gap marked, never synthesised.
- Idempotent Identity: footer stable-ID marker. Match by marker, NEVER title.
- Write-Side Safety: No tracker command before resolution; no mutation before confirmation; degrade to Markdown when no CLI.

Schema:

# Bug: [Title]  `[Risk: Level]`

## Context
- **Expected:** [Intended behavior]
- **Actual:** [Current failure state]
- **Reproducibility:** [Always | Intermittent | Once] · **Regression:** [Yes since <ref> `[Confidence: Level]` | No | Unknown]

## Reproduction
1. [Trigger step 1]
2. [Trigger step 2]

## Environment
- **Device/OS:** [only if user stated; else `Unknown — requires verification`]
- **Version:** [build/tag/commit] `[Confidence: Level]`
- **Suspected Seam:** [code location] `[Confidence: Level]` *(omit if unevidenced)*

## Diagnostics
- **Logs:** [Stack trace / console errors — redacted] `[Data: Classification]`
- **Evidence:** [Screenshot / video / link]

<!-- skills:work-item kind=bug id=BUG-<slug> -->