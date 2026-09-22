# AGENTS.example.md

Operational rules for agents working in a **project where this skills library is installed**. Copy this file to `AGENTS.md` in your own repository and adjust to taste — the installed skills read these conventions at runtime.

Rules for authoring/maintaining the skills library itself live in the library's own [`AGENTS.md`](AGENTS.md), not here.

## State & persistence

- NEVER persist runtime state inside the working tree, and never commit it. Resolve to a single **persistent** out-of-tree path — the agent state store. Platform base: Linux `${XDG_STATE_HOME:-$HOME/.local/state}/`, macOS `~/Library/Application Support/`, Windows `%LOCALAPPDATA%/`. Do NOT fall back to volatile OS temp (`${TMPDIR}`/`/tmp`): state that governs escalation, competency, or progress must survive OS cleanup and reboots. In chat-only runtimes with no writable out-of-tree location, hold state in memory and emit paste-back snapshots — never a workspace file, never temp. Skills that previously wrote a `${TMPDIR}` fallback must migrate any legacy file up to the persistent path on first read. See the `competency-profile` skill for the canonical pattern.
- State that belongs to the *person* (skill competency) is shared via `competency-profile`; project- or course-specific state stays with its owning skill.

## Domain glossary maintenance

- When agents add features, modify schemas, or introduce new domain concepts in this project, keep `docs/domain-glossary.json` in sync (see the `domain-glossary` skill). Update the glossary as you work to prevent drift between explicit `domain-glossary` invocations and the actual codebase.

## Architectural decisions & records

- Architectural decisions surfaced during work go to `docs/adr/ADR-XXXX.md` via the `architectural-decision-register` skill (or the `architect` persona). Do not let design rationale live only in chat or commit messages.
- Blueprint, data model, and related architecture artefacts live at `docs/architecture/` (e.g. `system-blueprint.md`, `data-model.md`) and are the reference points for blueprint-vs-implementation audits.
