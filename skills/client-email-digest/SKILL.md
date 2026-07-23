---
name: client-email-digest
description: Turns the work delivered between two git points into a client-facing weekly progress digest email — a warm, plain-language, non-technical sibling of generate-release-notes. Reuses generate-release-notes as the change-fact engine, then re-voices it for a paying client audience. Produces a fixed five-section email (TLDR, Change log as prose, Blockers, Release timeline, Upcoming leave) and maintains a lightweight, out-of-tree, per-project blocker tracker so a blockage that spans several digests reports how long it has been open. Interviews the user only for facts that cannot be inferred (release dates, comms channel, team leave) and never fabricates progress, dates, or stores.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - generate-release-notes
  - resolve-repository-platform
  - interview-me
  - design-vocab
  - agent-markup
argument-hint: "[since-ref] [until-ref]  # default since = last digest baseline in state; until = HEAD"
user-invocable: true
---
Audience: paying client / non-technical stakeholder, NOT engineering. Maximise inference from git + tracker; interview only for non-derivable facts. NEVER invent release date, store, comms channel, or progress.

1. PHASE 0 (Window & State): `until-ref` → HEAD. `since-ref` → last digest baseline in state; no prior → `interview-me` ONE question for starting point (date/tag/SHA). Load persisted digest state.
2. PHASE 1 (Change Facts): Invoke `generate-release-notes` for same window. Treat output as evidence ONLY — never paste shorthand bullets/tokens into email. Re-voice each shipped change into warm, plain-language client prose at capability boundary (`design-vocab`).
3. PHASE 2 (Blocker Reconciliation): Resolve platform via `resolve-repository-platform`. Read Work Items carrying blocked marker (e.g. `gh issue list --label blocked`). Reconcile against tracked blockers in state: open new, close resolved (record date), recompute days blocked. No authenticated CLI → fall back to user-reported blockers in state, ask only about changes.
4. PHASE 3 (Gap Interview): `interview-me` ONE question at a time, recommendation-first, ONLY for still-missing: release timeline (store/channel + date/window) when not inferable; comms channel; upcoming leave/events affecting velocity; human context behind blockers. Never re-ask what state/git answers.
5. PHASE 4 (Render & Persist): Render five-section email against schema. Write back digest state: advance baseline → `until-ref`, persist blocker ledger (with first-seen dates + durations), release timeline, comms channel, leave. State resolved storage path on first touch.
6. PHASE 5 (Preview): Present rendered email + one-line summary of tracker changes (blockers opened/closed, baseline advanced). Offer to adjust tone/greeting/sign-off. Drafts; never sends.

Directives:
- Audience Pivot: translate every technical change into value/capability delivered. Strip jargon, codenames, Module/file names, SHAs, branches, author handles, ticket IDs.
- Fixed Five Sections: TLDR, Change log, Blockers, Release timeline, Upcoming leave ALWAYS present. No real content → honest "all clear" line; never fabricate, never drop.
- TLDR Hard Cap: ≤6 bullets, ≤6 words each, finished outcomes. No sub-bullets, no trailing prose.
- Change Log as Prose: flowing prose paragraphs, not bulleted list. Most client-visible first. Narrative of what now works — not commit-by-commit.
- Blocker Persistence: tracked across digests with stable id + first-seen date. Each active reports days blocked + what needed, from whom, tagged `[Risk: Level]`. Plain/constructive — surface ask, never assign blame.
- Inference-First, Interview-Last: derive window, change facts, blockers from git/tracker/state before asking anything.
- Anti-Fabrication: every progress claim traces to real shipped change. Never announce unshipped/planned. Timeline not inferable/confirmed = "to be confirmed".
- Tone Neutralisation & Warmth: re-voice all ingested text into single warm, professional voice. Strip sentiment, hedging, profanity, blame. Honest about slippage/blockers, never alarmist.
- Output Portability: export-clean Markdown, `[Scope: Digest]`.

Storage: out-of-tree, NEVER in workspace.
- `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/client-email-digest/<project-id>/digest-state.md`
- Migrate legacy `${TMPDIR:-/tmp}/ai-skills/client-email-digest/<project-id>/digest-state.md` on first read.
- Chat-only: memory + paste-back snapshot.

```markdown
# Client Digest State — <project-id>
**Last digest baseline:** <ref> | **Updated:** <ISO date>
**Release timeline:** <store/channel — date or window | to be confirmed>
**Comms channel:** <how notified>

## Blocker Ledger
| ID | Summary | [Risk: Level] | First seen | Days blocked | Needed from | Status |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |

## Known Leave / Events
| Who | Dates | Velocity impact |
| :-- | :-- | :-- |
```

Schema `[Scope: Digest]`:

**Subject:** [Project] — Weekly progress, week of [date]

Hi [client / team name],

Here's where things stand this week.

## TL;DR
- [Outcome ≤ 6 words]
*(up to 6 bullets, ≤ 6 words each)*

## What we shipped this week
[Short prose paragraphs, plain language, most client-visible first. No bullets, no SHAs, no jargon.]

## Blockers
- **[Blocker]** — blocked [N] days. Needs [what] from [whom]. `[Risk: Level]`
*(If none: "No blockers this week — everything is moving.")*

## Release timeline
- **[Store / channel]:** [date or window | to be confirmed]
- **How you'll hear about it:** [comms channel]

## Upcoming leave & events
- [Who] — [dates] — [impact on pace]
*(If none: "No planned leave affecting delivery.")*

Happy to jump on a call if anything here needs unpacking.

[Sign-off]