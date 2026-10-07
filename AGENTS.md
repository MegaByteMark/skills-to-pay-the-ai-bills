# AGENTS.md

Operational rules for any agent **authoring or maintaining skills in this repository**. (This governs work *on* the skills library itself — it is not a template for end-user projects.)

Rules that govern agents working in a **project where these skills are installed** (target repos) live in [`AGENTS.example.md`](AGENTS.example.md) — a template end users copy into their own repositories.

## What this repo is
A library of self-contained **persona skills**: one `SKILL.md` per persona under `skills/<name>/`. Each persona embeds every workflow section and shared contract it uses as a demarcated block — no leaf skills, no cross-file dependencies. Personas are loaded on demand by opencode / Claude / `.agents`-compatible runtimes.

## Authoring rules (discovery-critical)
- One folder per persona under `skills/`. The folder name MUST equal the frontmatter `name` and match `^[a-z0-9]+(-[a-z0-9]+)*$`.
- Discovery is **flat** (`skills/*/SKILL.md`). NEVER nest skills inside category sub-folders — grouping is expressed in the README only.
- `SKILL.md` must be upper-case. `name` must be unique across the repo.
- Required frontmatter: `name`, `description` (≤1024 chars, leads with the persona's sub-commands), `license`, `metadata` (with `author` and `version`). `user-invocable` and `argument-hint` are documentation-only fields. **`dependencies` is prohibited** — everything the persona needs is embedded.
- Bump `metadata.version` on every modification. Semantic versioning: major for breaking changes, minor for new features, patch for fixes and prose edits.
- New capability → a section inside the owning persona, never a new skill file. New persona only when a genuinely distinct role emerges — challenge the request first. New persona → add a row to the README scenario table.

## Demarcated blocks & the contract-sync rule
- Every embedded workflow section and shared contract is wrapped in `<!-- BEGIN SECTION|CONTRACT: <name> @ <semver> (canonical: <persona>) -->` … `<!-- END … -->`.
- Each block has exactly ONE canonical home (its owning persona). All other embeddings are byte-identical copies.
- To change a block: edit the canonical home, bump its `@ <semver>`, then `grep -rn "BEGIN CONTRACT: <name>" skills/` (or `BEGIN SECTION:`) and sync every copy byte-identical. Verify with `diff`.
- Changing a contract's tokens or spawn rules is a breaking change for every consuming persona — bump the contract's major version and flag it in the PR.

## Content rules
- **No Mermaid in `SKILL.md`.** Control flow lives in numbered phases plus a `## Commands` table. Mermaid remains welcome in *generated artefacts* (blueprints, ERDs, PRD/FDS).
- Bracket tokens: only the `agent-markup` contract enumerations. New token → extend the canonical `agent-markup` block and sync per the rule above; never invent a persona-local token.
- Architecture terms: only the `design-vocab` contract taxonomy (Module, Interface, Implementation, Depth, Seam, Adapter). Prohibited: unit/component/service/API/boundary except when naming a literal path.
- Every subagent spawn site declares `[Handoff: Clean]` or `[Handoff: Enriched]`; every spawned section carries the matching consume block; field tables match on both sides (see the `agent-handoff` contract, canonical in `swe`).
- Anti-hallucination: never reference non-existent files, personas, or documents; instruct graceful fallback on absence.
- Output determinism: same inputs → structurally identical output. No "you may also" branches unless gated behind an explicit decision.
- State persistence: `${XDG_STATE_HOME:-$HOME/.local/state}/ai-skills/...`. Never the working tree. Never temp.
- Prose compaction after every modification: delete any sentence whose removal doesn't change agent behavior; merge repeated constraints into one canonical statement; restore the minimum if ambiguity appears.
- Challenge the human: reason whether a request is fully thought through; consider downstream impact across every copy of a block before editing it.

## Self-contained skills (no in-repo ADRs)
- Do NOT create ADRs in `docs/adr/` for skills authored or modified in this repository. End users who install skills only receive the skill directories (`skills/<name>/`). Any design decision, operational contract, or architectural rule governing a persona must be documented directly inside its `SKILL.md`.

## Git & PR workflow
- Branch `feature/<slug>` off `develop`; open PRs **into `develop`** (not `main`).
- Reference the relevant issue (`Closes #N`) and write concise, descriptive commit messages.
- Only commit when asked. Never commit secrets or out-of-tree state files.
- Don't add build tooling, CI, or extra docs unless explicitly requested.
