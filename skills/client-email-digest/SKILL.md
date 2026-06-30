---
name: client-email-digest
description: Turns the work delivered between two git points into a client-facing weekly progress digest email — a warm, plain-language, non-technical sibling of generate-release-notes. Reuses generate-release-notes as the change-fact engine, then re-voices it for a paying client audience. Produces a fixed five-section email (TLDR, Change log as prose, Blockers, Release timeline, Upcoming leave) and maintains a lightweight, out-of-tree, per-project blocker tracker so a blockage that spans several digests reports how long it has been open. Interviews the user only for facts that cannot be inferred (release dates, comms channel, team leave) and never fabricates progress, dates, or stores.
dependencies:
  - generate-release-notes  # Change-fact engine: supplies the authoritative commit/PR delta, re-voiced into client prose for the Change log
  - resolve-repository-platform  # Resolves host/CLI before reading blocked-labelled Work Items and merged Change Proposals
  - interview-me  # One-question-at-a-time, recommendation-first gathering of the non-inferable inputs (timeline, comms channel, leave, blocker context)
  - design-vocab  # Keeps every change described at the capability boundary, not at raw file/Module level
  - agent-markup  # [Scope: Digest], [Risk: Level], [Confidence: Level]
argument-hint: "[since-ref] [until-ref]  # default since = last digest baseline in state; until = HEAD"
user-invocable: true
---

Write the weekly client digest email defined by the schema below. The audience is the **paying client / non-technical stakeholder**, NOT the engineering team — this is the audience pivot away from `generate-release-notes`. Maximise inference from git and the tracker; interview the user only for what cannot be derived. NEVER invent a release date, store, comms channel, or piece of progress.

Operational Workflow:
1. PHASE 0 (Window & State Load): Resolve the comparison window. `until-ref` defaults to `HEAD`. `since-ref` defaults to the **last digest baseline** recorded in the persisted digest state (Storage below); if no prior digest exists, ask the user for the starting point (a date, tag, or SHA) via a single `interview-me` question. Load the persisted digest state so prior blockers, the release timeline, and known leave are not re-asked.
2. PHASE 1 (Change Facts): Invoke `generate-release-notes` across the same window to obtain the authoritative, deduplicated change facts. Treat its output strictly as *evidence* — never paste its shorthand bullets or `[feat]/[fix]` tokens into the email. Re-voice each shipped change into warm, plain-language client prose at the capability boundary (`design-vocab`).
3. PHASE 2 (Blocker Reconciliation): Resolve the platform via `resolve-repository-platform`, then read Work Items carrying a blocked marker (e.g. on GitHub `gh issue list --label blocked`; the `glab` equivalent on GitLab). Reconcile against the tracked blockers in state: open new ones, close resolved ones (record resolution date), and recompute **days blocked** for those still active. If no authenticated CLI exists, fall back to the user-reported blockers held in state and ask only about changes.
4. PHASE 3 (Gap Interview): Via `interview-me`, ask one question at a time, recommendation-first, ONLY for the still-missing inputs: release timeline (which store/channel + date or window) when it cannot be inferred from milestones/tags/CI; the comms channel for the release; upcoming team leave or events that affect velocity; and any human context behind a blocker (what is needed, from whom). Do not re-ask anything the state or git already answers.
5. PHASE 4 (Render & Persist): Render the five-section email against the schema. Then write back the digest state: advance the baseline to `until-ref`, persist the updated blocker ledger (with first-seen dates and durations), the release timeline, comms channel, and known leave. State the resolved storage path the first time you touch it.
6. PHASE 5 (Preview): Present the rendered email plus a one-line summary of what changed in the tracker (blockers opened/closed, baseline advanced). Offer to adjust tone, greeting, or sign-off before the user sends it. This skill drafts the email; it never sends it.

## Operational Directives
- Audience Pivot: Write for a client who is paying for outcomes, not reading code. Translate every technical change into the value or capability it delivers. Strip jargon, internal codenames, Module/file names, SHAs, branch names, author handles, and ticket IDs. If a sentence only makes sense to an engineer, rewrite or cut it.
- Fixed Five Sections: TLDR, Change log, Blockers, Release timeline, and Upcoming leave are ALWAYS present — clients rely on a consistent shape week to week. When a section has nothing real, render an honest "all clear" line (e.g. "No blockers this week."); never pad with fabricated content and never silently drop a section.
- TLDR Hard Cap: The TLDR is at most **6 bullets**, each at most **6 words**, each a finished outcome ("Checkout flow now live", "Search 40% faster"). No sub-bullets, no trailing prose. Fewer real bullets beats six padded ones.
- Change Log as Prose: The Change log is flowing prose (short paragraphs), not a bulleted shorthand list — this is the format break from `generate-release-notes`. Lead with the most client-visible progress. Group related work into a narrative of what now works, not a commit-by-commit recount.
- Blocker Persistence & Duration: A blocker is tracked across digests with a stable id and a first-seen date. Each active blocker reports **how long it has been blocked** (calendar days from first-seen) and **what is needed to clear it, from whom**, tagged `[Risk: Level]` for delivery impact. State blockers plainly and constructively — surface the ask, never assign blame.
- Inference-First, Interview-Last: Derive the window, change facts, and blockers from git/tracker/state before asking anything. Pull the timeline, comms channel, and leave from prior state and only re-interview when they are absent or the user signals a change.
- Anti-Fabrication: Every claim of progress traces to a real shipped change in the window. Never announce unshipped or planned work as done. Never invent a release date, store, comms channel, holiday, or blocker resolution. A timeline that cannot be inferred or confirmed is rendered as "to be confirmed", not guessed.
- Tone Neutralisation & Warmth: Re-voice all ingested text (commits, Change Proposal discussion, tracker comments) into a single warm, professional voice. Strip sentiment, hedging, profanity, and blame. Honest about slippage and blockers, but never alarmist.
- Output Portability: Export-clean Markdown per the `agent-markup` Output Portability Convention, suitable for pasting into an email client or rendering under a letterhead. `[Scope: Digest]`.

Storage — per-project, out-of-tree, NEVER in the working tree:
The digest state (baseline pointer + blocker ledger + timeline + leave) is project-specific and MUST NOT live in the user's repo, be committed, or appear in `git status`. Namespace it per project (by remote URL slug, else top-level directory name) and resolve the path in order:
1. Agent-owned state store — `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/client-email-digest/<project-id>/digest-state.md`.
2. Fallback OS temp dir — `${TMPDIR:-/tmp}/ai-skills/client-email-digest/<project-id>/digest-state.md`.
If the runtime is chat-only with no writable out-of-tree location, hold the state in agent memory and emit it as a paste-back snapshot at the end of the run so the next digest can resume duration tracking — never substitute a workspace file.

```markdown
# Client Digest State — <project-id>
**Last digest baseline:** <ref> | **Updated:** <ISO date>
**Release timeline:** <store/channel — date or window | to be confirmed>
**Comms channel:** <how the client is notified of the release>

## Blocker Ledger
| ID | Summary | [Risk: Level] | First seen | Days blocked | Needed from | Status |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| BLK-1 | App Store review pending | High | 2026-06-12 | Apple review | Active |

## Known Leave / Events
| Who | Dates | Velocity impact |
| :-- | :-- | :-- |
```

## Client Digest Email Schema  `[Scope: Digest]`

**Subject:** [Project] — Weekly progress, week of [date]

Hi [client / team name],

Here's where things stand this week.

## TL;DR
- [Outcome ≤ 6 words]
- [Outcome ≤ 6 words]
*(up to 6 bullets, ≤ 6 words each; drop unused slots)*

## What we shipped this week
[Short prose paragraphs describing the progress in plain language, most client-visible first. No bullets, no SHAs, no jargon.]

## Blockers
- **[Plain-language blocker]** — blocked [N] days. Needs [what] from [whom]. `[Risk: Level]`
*(If none: "No blockers this week — everything is moving.")*

## Release timeline
- **[Store / channel]:** [date or window | to be confirmed]
- **How you'll hear about it:** [comms channel]

## Upcoming leave & events
- [Who] — [dates] — [brief impact on pace]
*(If none known: "No planned leave affecting delivery this period.")*

Happy to jump on a call if anything here needs unpacking.

[Sign-off]
