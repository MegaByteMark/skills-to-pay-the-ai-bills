# Skills to Pay the AI Bills

A library of composable **Agent Skills** — `SKILL.md` definitions that an AI coding agent loads on demand to take on a specific, repeatable job: auditing a codebase, running a requirements interview, teaching you a language, keeping you sharp while it builds, and more.

Skills are compatible with **opencode**, **Claude**, and any `.agents`-compatible runtime. They range from small shared *contracts* (a fixed vocabulary, a markup token set) up to full interactive *workflows* with their own slash commands.

---

## Install

Each skill is a folder containing a `SKILL.md`. Copy or symlink the folders you want into one of the locations the agent searches:

| Scope | opencode | Claude | agents |
| :-- | :-- | :-- | :-- |
| Project | `.opencode/skills/<name>/` | `.claude/skills/<name>/` | `.agents/skills/<name>/` |
| Global | `~/.config/opencode/skills/<name>/` | `~/.claude/skills/<name>/` | `~/.agents/skills/<name>/` |

```sh
# Install everything globally for any agent
cp -R skills/* ~/.agents/skills/

# …or symlink a single skill (handy while iterating)
ln -s "$PWD/skills/teach-me" ~/.config/opencode/skills/teach-me
```

Rules that matter for discovery:

- Keep each skill's folder name exactly as-is — it must equal the `name` in its frontmatter and match `^[a-z0-9]+(-[a-z0-9]+)*$`.
- Discovery is **flat** (`skills/*/SKILL.md`); do **not** nest skills inside category sub-folders or they won't be found.
- `SKILL.md` must be upper-case, and `name` must be unique across every install location.

---

## How invocation works

1. **Just ask.** The agent sees every installed skill's name + description and loads the right one with its `skill` tool when your request matches — e.g. *"audit this repo's health"*, *"teach me Rust"*, *"keep me sharp while we build this"*.
2. **Steer with slash commands.** Interactive skills define commands you type mid-session (see [Interactive commands](#interactive-commands)).
3. **Some skills are plumbing.** Shared contracts and leaf skills are pulled in by *other* skills; you rarely invoke them directly.

---

## Skill catalog

Grouping is by convention only (the files stay flat for discovery).

### Foundations & shared contracts
*Pure contracts other skills build on — no standalone workflow.*
- **design-vocab** — a rigid architectural vocabulary (Module, Interface, Implementation, Depth, Seam, Adapter) to stop semantic drift.
- **agent-markup** — the machine-readable bracket-token schema (`[Risk: Level]`, `[Confidence: Level]`, `[Competency: Level]`, …).
- **competency-profile** — the shared, out-of-tree, per-user record of a human's demonstrated skill, so calibration is continuous across skills.
- **resolve-repository-platform** — figures out the hosting platform (GitHub/GitLab/…) before any platform-specific tooling runs.
- **detect-test-harness** — resolves the project's test runner/framework, layout, and native test-double idiom from signal files before any test is read or written; asks one question only when inconclusive and never introduces a new framework silently.

### Requirements & discovery
- **interview-me** — relentless one-question-at-a-time design interrogation with a recommendation on every question.
- **gather-requirements** — drives `interview-me` across two streams: a **product stream** producing a Product Requirements Document (PRD) of vision, personas, Epics and MoSCoW-prioritised user stories that seed the backlog, then a **functional stream** producing the Functional Design Specification (FDS), with every FDS requirement traced back to its originating PRD Epic/story.

### Backlog seeding *(publish-side of `gather-requirements`)*
*Turn the PRD/FDS into tracked work items. Re-runnable: a second pass reconciles the tracker against amended requirements (create/update/close) via embedded stable-ID markers — never duplicating.*
- **seed-backlog** — *orchestrator*; resolves the platform once and sequences the two leaves across the whole Epic Register and Story Backlog, wiring each story to its parent epic, then emits an auditable seed report.
  - **create-epic** — *leaf*; renders/writes one epic (PRD-primary, FDS-enriched).
  - **create-user-story** — *leaf*; renders/writes one story as a child of its epic.

### Architecture, analysis & documentation
- **clean-architecture** — *prescriptive counterpart to `analyze-a-codebase`*; scaffolds and enforces a layered, dependency-inverted structure (Domain → Application → Interface Adapters → Infrastructure), mapping each artifact to a strict path and HALTing on inward-dependency violations. Speaks `design-vocab`.
- **analyze-a-codebase** — ingests a repo and produces a structured system blueprint.
- **document-a-codebase** — generates user / technical / installation docs from the FDS, blueprint, and code.
- **db-normalisation** — turns the FDS (or a direct spec) into a fully normalised relational data model — or reverse-engineers and audits an existing database — walking `interview-me` through the normal forms (UNF→1NF→2NF→3NF, optionally BCNF), sweeping for the canonical persistence anti-patterns, and writing a Mermaid ERD plus a documented data dictionary to `docs/architecture/data-model.md`.

### Code-quality enforcement
*Standalone review overlays — load whichever fits the codebase and task. Both audit only supplied code and calibrate every finding with `[Confidence: Level]` to curb false positives.*
- **dry-kiss** — enforces DRY / KISS / YAGNI to block duplication, over-engineering, and gratuitous cleverness.
- **solid-principles** — enforces SOLID OOP design; HALTs on God classes, tight coupling, and brittle inheritance with a `[Risk: Level]` tag.

### Audit & remediation
- **audit-application-health** — *orchestrator*; runs the three leaf audits and synthesises one client-facing health report.
  - **audit-security-and-governance** — security + GDPR/data-protection scan.
  - **audit-blueprint-implementation** — code-vs-blueprint/FDS drift.
  - **audit-test-coverage** — real coverage vs the target test surface.
- **remediate-test-coverage** — closes gaps found by the coverage audit, writing the minimum sufficient tests.

  Both test skills resolve the runner/framework through the shared **detect-test-harness** contract.

### Learning & skill-retention
- **teach-me** — *orchestrator*; an end-to-end language course (intake, syllabus, sequencing, spaced repetition) that delegates each lesson to `teach-a-skill`.
- **teach-a-skill** — *leaf*; closes **one** knowledge gap to a target competency. Promptable by an agent or a human.
- **vibe-code-antidote** — a session overlay that hands you self-contained slices of a real build at random to fight skill atrophy, escalating to `teach-a-skill` when it detects a gap.

### Release & ops
- **generate-release-notes** — high-density release notes from commits, diffs, and merged PR discussions.
- **client-email-digest** — *client-facing sibling of `generate-release-notes`*; reuses it as the change-fact engine, then re-voices the work between two git points into a warm, non-technical weekly progress email (TLDR, prose change log, blockers, release timeline, upcoming leave). Keeps a lightweight, out-of-tree per-project blocker tracker so a blockage spanning several digests reports how long it's been open, and interviews only for the non-inferable inputs (release dates, comms channel, team leave).
- **create-bug-report** — auto-captures every evidenced field (git/build version, runtime, pasted stack traces) and interviews only for the human-centric gaps, then renders a fixed bug-report schema and optionally files it as a Work Item via `resolve-repository-platform`. Evidence-first and anti-hallucination: unevidenced, unanswered fields stay `Unknown — requires verification`.

---

## Interactive commands

Type these mid-session once the relevant skill is active.

### teach-me
| Command | Effect |
| :-- | :-- |
| `/syllabus` | Reprint the full syllabus + progress dashboard |
| `/progress` | Show the progress dashboard only |
| `/quiz me` | Pop quiz on a random completed topic |
| `/recap [topic]` | 3-sentence refresher, no challenge |
| `/restart [phase\|topic]` | Uncheck and restart that section |
| `/skip` | Mark the current topic skipped, advance |
| `/harder` · `/easier` | Raise / lower lesson difficulty |
| `/mobile` · `/desktop` | Switch input mode |
| `/pause` | Mark a stopping point and emit a resume snapshot |

### vibe-code-antidote
| Command | Effect |
| :-- | :-- |
| `/handoff` | Hand me the next eligible slice now |
| `/take-over` | You write this one; I review or skip |
| `/skip` | Skip this handoff, keep building |
| `/pause-antidote` · `/resume-antidote` | Stop / restart all handoffs |
| `/intensity light\|normal\|intense` | Change handoff frequency |
| `/review-only` | Never hand me writing; I review your code |
| `/profile` | Show competency + comprehension dashboard |
| `/calibrate [area]` | Quick calibration probe on an area |
| `/harder` · `/easier` | Adjust handoff difficulty |

### teach-a-skill
No slash commands — invoke it with a target concept, e.g. *"teach-a-skill: TypeScript async/await"*. It teaches that one concept, then stops.

---

## First time "vibing"? 🛟

1. **Install the skills** (see [Install](#install)) — `cp -R skills/* ~/.agents/skills/` is the quickest start.
2. **Just talk to your agent.** Describe what you want built; it'll load skills as needed. You don't memorise anything.
3. **Want to stay sharp instead of going full autopilot?** Say *"use the vibe-code-antidote"*. It keeps doing the heavy lifting but hands you safe, self-contained pieces to write — after checking you actually understand the surrounding code.
4. **Hit something you don't know?** The antidote will offer to hand that exact gap to `teach-a-skill`. Five focused minutes later you're back to building, gap closed — and the skill it taught you is remembered.
5. **Want to learn a language properly?** Say *"be my programming tutor for Go"*. The course and the antidote share the **same** competency baseline, so a study evening and a build day reinforce each other.

> The point isn't to slow you down — it's to make sure that when you ship code, you can still read, debug, and defend it.

### Make your agent proactive (optional)

Skills auto-discover, so nothing below is required. But the behavioral overlays (`vibe-code-antidote`, `teach-me`) are opt-in by nature — if you want your agent to *offer* them without being asked, drop a snippet like this into your own project's `AGENTS.md` (or `CLAUDE.md`):

```markdown
## Skill usage
- When I'm building something non-trivial, proactively offer the `vibe-code-antidote`
  skill so I keep writing some of the code myself and stay sharp.
- If you detect I don't understand part of the implementation, use `teach-a-skill`
  to close that one gap before continuing.
- If I ask to learn a language or topic from scratch, use `teach-me`.
- Before adding features to an inherited or unfamiliar codebase, check for
  `docs/requirements/functional-requirements.md` and `docs/architecture/system-blueprint.md`.
  If either is missing, treat that as a finding, not a reason to skip discovery: run
  `analyze-a-codebase` (which seeds the FDS via `gather-requirements`) and surface the
  scope gaps before writing any new feature. Do not jump straight into new work on a
  system we don't yet have a picture of.
```

---

## How skills compose

Skills reference each other by `name` (declared in a doc-only `dependencies:` block). The clusters with real orchestration:

```
audit-application-health ──> audit-security-and-governance
                         ├──> audit-blueprint-implementation
                         └──> audit-test-coverage  <── remediate-test-coverage
                                   └──────────┬──────────────┘
                                              └──> detect-test-harness  (shared harness resolution)

seed-backlog ──> create-epic ───────┐
             └──> create-user-story ─┴──> resolve-repository-platform  (write-side adapter)

client-email-digest ──> generate-release-notes  (change-fact engine, re-voiced for the client)

teach-me          ──> teach-a-skill ──┐
vibe-code-antidote ───────────────────┼──> competency-profile  (shared baseline)
                                       └──> agent-markup / design-vocab  (shared contracts)
```

State that a skill persists (course progress, competency baseline, capability profiles) always lives **outside the project tree** — it is never committed to your repo.

---

## Conventions for new skills

- One folder per skill; folder name == `name`; `SKILL.md` upper-case.
- Frontmatter: `name` and `description` are required and read by opencode. This repo also uses `dependencies`, `argument-hint`, and `user-invocable` as **documentation-only** fields (ignored by the runtime but useful to readers and authors).
- Speak `design-vocab` for architecture and restrict bracket tokens to the `agent-markup` enumerations.
- Never persist runtime state inside a user's working tree.

---

## License

MIT — see [LICENSE](LICENSE).
