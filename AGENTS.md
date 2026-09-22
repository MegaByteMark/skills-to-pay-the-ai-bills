# AGENTS.md

Operational rules for any agent **authoring or maintaining skills in this repository**. (This governs work *on* the skills library itself — it is not a template for end-user projects.)

Rules that govern agents working in a **project where these skills are installed** (target repos) live in [`AGENTS.example.md`](AGENTS.example.md) — a template end users copy into their own repositories.

## What this repo is
A library of Agent Skills: one `SKILL.md` per skill under `skills/<name>/`. Skills are loaded on demand by opencode / Claude / `.agents`-compatible runtimes.

## Authoring rules (discovery-critical)
- One folder per skill under `skills/`. The folder name MUST equal the frontmatter `name` and match `^[a-z0-9]+(-[a-z0-9]+)*$`.
- Discovery is **flat** (`skills/*/SKILL.md`). NEVER nest skills inside category sub-folders — grouping is expressed in the README only.
- `SKILL.md` must be upper-case. `name` must be unique across the repo.
- Required frontmatter: `name`, `description` (≤1024 chars, specific enough for correct selection), `license`, and `metadata` (with `author` and `version`). `compatibility` is also recognized; all other fields are ignored by the runtime.
- This repo additionally uses `dependencies`, `argument-hint`, and `user-invocable` as **documentation-only** fields. Reference other skills by bare `name` in `dependencies`.
- Bump `metadata.version` on every modification. Use semantic versioning: bump major for breaking changes, minor for new features, patch for fixes and prose edits.
- Skills embed the runtime state contract defined in `AGENTS.example.md` (§ State & persistence): never persist runtime state in the user's working tree; resolve to a persistent out-of-tree agent state store. See `competency-profile` for the canonical pattern.

## Vocabulary & markup compliance
- Use the `design-vocab` taxonomy for architecture: Module, Interface, Implementation, Depth, Seam, Adapter. Avoid unit/component/service/API/boundary except when naming a literal path.
- Restrict bracket tokens to the `agent-markup` enumerations (`[Risk: Level]`, `[Confidence: Level]`, `[Remediation: Effort]`, `[Competency: Level]`, …). Need a new token? Extend `agent-markup` rather than inventing a skill-local one.

## Architecture idioms
- Prefer **leaf + orchestrator** decomposition: a focused leaf does one job; an orchestrator sequences leaves (see `teach-me` → `teach-a-skill`, `audit-application-health` → its leaf audits).
- Factor cross-skill rules into a **shared-contract skill** (`design-vocab`, `agent-markup`, `commentary`, `competency-profile`, `resolve-repository-platform`) rather than duplicating them.
- Keep skills concise and high-signal; cut prose that doesn't change agent behavior.

## Self-contained skills (no in-repo ADRs)
- Skills in this library must be completely self-contained. Do NOT create ADRs in `docs/adr/` for skills authored or modified in this repository. End users who install skills only receive the skill directories (`skills/<name>/`) and do not have access to internal ADRs. Any design decisions, operational contracts, or architectural rules governing a skill must be documented directly and explicitly inside the skill's `SKILL.md`.

## Git & PR workflow
- Branch `feature/<slug>` off `develop`; open PRs **into `develop`** (not `main`).
- Reference the relevant issue (`Closes #N`) and write concise, descriptive commit messages.
- Only commit when asked. Never commit secrets or out-of-tree state files.
- Don't add build tooling, CI, or extra docs unless explicitly requested.
