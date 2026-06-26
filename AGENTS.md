# AGENTS.md

Operational rules for any agent **authoring or maintaining skills in this repository**. (This governs work *on* the skills library itself — it is not a template for end-user projects.)

## What this repo is
A library of Agent Skills: one `SKILL.md` per skill under `skills/<name>/`. Skills are loaded on demand by opencode / Claude / `.agents`-compatible runtimes.

## Authoring rules (discovery-critical)
- One folder per skill under `skills/`. The folder name MUST equal the frontmatter `name` and match `^[a-z0-9]+(-[a-z0-9]+)*$`.
- Discovery is **flat** (`skills/*/SKILL.md`). NEVER nest skills inside category sub-folders — grouping is expressed in the README only.
- `SKILL.md` must be upper-case. `name` must be unique across the repo.
- Required frontmatter read by the runtime: `name`, `description` (≤1024 chars, specific enough for correct selection). `license`, `compatibility`, and `metadata` are also recognized; all other fields are ignored by the runtime.
- This repo additionally uses `dependencies`, `argument-hint`, and `user-invocable` as **documentation-only** fields. Reference other skills by bare `name` in `dependencies`.

## Vocabulary & markup compliance
- Use the `design-vocab` taxonomy for architecture: Module, Interface, Implementation, Depth, Seam, Adapter. Avoid unit/component/service/API/boundary except when naming a literal path.
- Restrict bracket tokens to the `agent-markup` enumerations (`[Risk: Level]`, `[Confidence: Level]`, `[Remediation: Effort]`, `[Competency: Level]`, …). Need a new token? Extend `agent-markup` rather than inventing a skill-local one.

## State & persistence
- NEVER persist runtime state inside a user's working tree, and never commit it. Resolve an out-of-tree path: agent state store (e.g. `${XDG_STATE_HOME:-$HOME/.local/state}/…`) → OS temp fallback (`${TMPDIR:-/tmp}/…`). See `competency-profile` for the canonical pattern.
- State that belongs to the *person* (skill competency) is shared via `competency-profile`; project- or course-specific state stays with its owning skill.

## Architecture idioms
- Prefer **leaf + orchestrator** decomposition: a focused leaf does one job; an orchestrator sequences leaves (see `programming-tutor` → `teach-a-skill`, `audit-application-health` → its leaf audits).
- Factor cross-skill rules into a **shared-contract skill** (`design-vocab`, `agent-markup`, `competency-profile`, `resolve-repository-platform`) rather than duplicating them.
- Keep skills concise and high-signal; cut prose that doesn't change agent behavior.

## Git & PR workflow
- Branch `feature/<slug>` off `develop`; open PRs **into `develop`** (not `main`).
- Reference the relevant issue (`Closes #N`) and write concise, descriptive commit messages.
- Only commit when asked. Never commit secrets or out-of-tree state files.
- Don't add build tooling, CI, or extra docs unless explicitly requested.
