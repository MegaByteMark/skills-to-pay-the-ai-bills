---
name: create-bug-report
description: Generates a complete, high-signal bug report by auto-capturing every field it can evidence from the codebase, git, the runtime, and any error text the user pasted, then interviewing the user one question at a time (via interview-me) for only the human-centric gaps that cannot be derived. Renders the report against a fixed schema and optionally creates or amends it as a Work Item in the resolved tracking platform. Evidence-first and anti-hallucination: every field is either evidenced, user-answered, or an explicit "Unknown — requires verification" — never invented.
dependencies:
  - interview-me  # Asks only for the gaps that cannot be auto-derived, one question at a time, with a recommended answer each time
  - resolve-repository-platform  # Resolves the platform and Work-Item Authoring map before any tracker write; supplies git-only degradation
  - agent-markup  # [Risk: Level], [Confidence: Level], [Inferred: Unverified], [Data: Classification]
argument-hint: "[title | pasted error/stack trace]  # optional seed; mode (create|amend) auto-detected from the tracker"
user-invocable: true
---

Generate a bug report that meets or exceeds the schema below. Maximise auto-capture; minimise questions. NEVER fabricate a log line, version, or reproduction step — an unevidenced, unanswered field is rendered as `Unknown — requires verification`, not guessed.

Operational Workflow:
1. PHASE 0 (Input & Mode): Take any seed (title or pasted error/stack trace). Once the platform is resolved (Phase 4), if a Work Item already carries this report's stable-ID marker, default to `amend`; else `create`. Never blind-create a duplicate.
2. PHASE 1 (Evidence Auto-Capture): Silently derive every field obtainable without the user. Sources: pasted error/stack trace → Diagnostics; package/build manifests and `git describe`/tags/`HEAD` → Version; declared runtime/OS from manifests, lockfiles, CI config → Environment (the host OS is an environment fact about the agent's machine, not the user's — only assert the user's when stated). Tag each derived field `[Confidence: Confirmed | Probable | Possible]`; a `Possible` field is phrased as requires-verification. Classify captured diagnostics `[Data: Classification]` and redact secrets/credentials/PII before they enter the report.
3. PHASE 2 (Gap Interview): For ONLY the fields still missing after Phase 1 — typically Expected, Actual, exact reproduction steps, evidence links — invoke `interview-me`: one question at a time, each with a recommended/most-likely answer as a baseline. Do not re-ask anything Phase 1 already evidenced.
4. PHASE 3 (Triage Enrichment): Derive Severity `[Risk: Level]` from impact + frequency; record reproducibility (always/intermittent) and whether it is a regression if git evidence supports it. Name the suspected Seam only when code evidence points to one, tagged with `[Confidence: Level]`; otherwise omit it rather than speculate.
5. PHASE 4 (Render): Compile into the Bug Report Schema. Append the stable-ID marker footer.
6. PHASE 5 (Preview & Write): Run `resolve-repository-platform`; present the rendered report and the intended action (create | amend, target platform). Require explicit confirmation before any write. On confirmation, create/amend the Work Item via the adapter row. If no authenticated CLI is available, emit the Markdown and say so. Report the resulting Work-Item reference.

## Operational Directives
- Evidence-First, Interview-Last: Resolve from code/git/runtime/pasted-error before asking a human anything. Every avoided question that the evidence already answers is the goal.
- No Invention: A field that is neither evidenced nor answered is `Unknown — requires verification`. Never assert a version, log line, repro step, or environment you did not observe.
- Honest Provenance: Tag auto-derived fields with `[Confidence: Level]`; mark any field reconstructed from an ephemeral baseline `[Inferred: Unverified]`. `Possible`-confidence fields are phrased as requiring verification, never asserted.
- Redaction: Classify diagnostics with `[Data: Classification]` and strip secrets/tokens/PII before they reach the report or tracker.
- Reproduction Discipline: Repro steps must be concrete and ordered. If the user cannot give exact steps, record what is known and mark the gap — do not synthesise plausible-looking steps.
- Idempotent Identity: Embed the stable-ID marker (footer). Match existing Work Items by that marker, NEVER by title.
- Resolve-Before-Invoke & Write-Side Safety: No tracker command before `resolve-repository-platform` resolves; no mutation before explicit user confirmation; degrade to portable Markdown when no authenticated CLI exists.
- Output Portability: Export-clean Markdown per the `agent-markup` Output Portability Convention.

## Bug Report Schema

# Bug: [Title]  `[Risk: Level]`

## Context
- **Expected:** [Intended behavior]
- **Actual:** [Current failure state]
- **Reproducibility:** [Always | Intermittent | Once] · **Regression:** [Yes since <ref> `[Confidence: Level]` | No | Unknown]

## Reproduction
1. [Trigger step 1]
2. [Trigger step 2]

## Environment
- **Device/OS:** [e.g. macOS / Chrome — only if stated by the user; else `Unknown — requires verification`]
- **Version:** [build/tag/commit] `[Confidence: Level]`
- **Suspected Seam:** [code location the evidence implicates] `[Confidence: Level]`  *(omit if unevidenced)*

## Diagnostics
- **Logs:** [Stack trace or console errors — redacted] `[Data: Classification]`
- **Evidence:** [Screenshot / video / link]

<!-- skills:work-item kind=bug id=BUG-<slug> -->
