---
name: resolve-repository-platform
description: Shared protocol for determining which hosting platform a repository lives on (GitHub, GitLab, Bitbucket, self-hosted, or none) before any platform-specific tooling is invoked. Provides host inference, a user confirmation gate when inference is inconclusive, a platform->CLI/terminology adapter map, and a git-only graceful-degradation fallback. Consumed by any skill that would otherwise hard-code GitHub assumptions.
dependencies:
  - interview-me
  - agent-markup
---
Neutral Vocabulary (use in prose; map to platform terms only at tooling layer):
- Change Proposal: umbrella for Pull Request / Merge Request. Never hard-code "PR".
- Review Discussion: comment/review thread on a Change Proposal.
- Repository Custody: namespace owning repo (individual vs. company/org).
- Repository Visibility: public, internal, or private.
- Work Item: umbrella for issue/epic/story. Never hard-code "issue".
- Parent Link: hierarchy relation (epic→story / parent→sub-issue).

Resolution Protocol:
1. PHASE 1 (Host Inference): `git remote -v` → parse origin/first remote host:
   - `github.com` (or GHE host) → GitHub
   - `gitlab.com` (or self-managed) → GitLab
   - `bitbucket.org` (or Data Center) → Bitbucket
   - other (Azure DevOps, Gitea, self-hosted, unknown) → UNRESOLVED
   - no remote → NO-REMOTE
   Recognised public host = `Confirmed`; heuristic naming only = `Probable`.
2. PHASE 2 (Confirmation Gate): UNRESOLVED, or confidence < `Confirmed`, or consuming skill needs platform-only data (Change Proposal threads, Visibility) → `interview-me` ONE question: declare platform (+ base host for self-managed). Never guess self-hosted from naming. NO-REMOTE → skip, proceed git-only.
3. PHASE 3 (Capability Probe): Verify CLI installed + authenticated (`gh auth status`, `glab auth status`). Unauthenticated/absent = unavailable platform.
4. PHASE 4 (Graceful Degradation): No authenticated CLI → degrade to git-only (commit log, remote namespace, filesystem). Platform-only data (Change Proposal discussion, Visibility via API) → `Possible — requires verification`, NEVER asserted.

Platform Adapter Map:
| Platform | Host | CLI | Auth Check | Change Proposal | Illustrative Thread Retrieval | Visibility/Owner Lookup |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| GitHub | github.com / GHE | `gh` | `gh auth status` | Pull Request | `gh pr list --state merged`, `gh pr view <n> --comments` | `gh repo view --json visibility,owner` |
| GitLab | gitlab.com / self-managed | `glab` | `glab auth status` | Merge Request | `glab mr list --merged`, `glab mr view <n>` | `glab repo view` / project API `visibility` |
| Bitbucket | bitbucket.org / Data Center | none universal | n/a | Pull Request | REST API (token required) | REST API `is_private` (token required) |
| Self-hosted/Other | resolved via gate | per user | per user | per platform | per user-declared, else git-only | git-only |
| No remote | n/a | n/a | n/a | n/a | unavailable (git-only) | git-only |
*(Commands illustrative — verify at runtime.)*

Work-Item Authoring Adapter Map (write-side):
| Platform | Work Item | Epic Representation | Illustrative Create/Amend/Close | Illustrative Parent Link |
| :--- | :--- | :--- | :--- | :--- |
| GitHub | Issue | Issue (`epic`-labelled; no first-class Epic) | `gh issue create/edit/close` | GraphQL `addSubIssue` (else task-list/tracking issue) |
| GitLab | Issue | Native Epic (group-level; tier-gated) | `glab issue create/update/close` (Epics via API) | epic↔issue association or `/epic` quick action |
| Bitbucket | Issue | none native | REST API (token required) | REST API only; no native epic hierarchy |
| Self-hosted/Other | per user | per user | per user | per user |
| No remote | n/a | n/a | unavailable → portable Markdown | n/a |

Directives:
- Resolve-Before-Invoke: consuming skill MUST run this protocol before any platform-specific command. Never assume GitHub.
- Single-Gate: at most ONE `interview-me` question. Re-use answer for remainder of run.
- Terminology Neutrality: use Neutral Vocabulary in shared/client-facing prose. Substitute platform term only inside platform-specific tooling steps.
- Honest Unavailability: when platform tooling unavailable, state plainly and degrade — never fabricate Change Proposal or Visibility.
- Write-Side Safety: consumer performing creates/amends/closes MUST get explicit confirmation of planned mutation set before any write. No authenticated CLI → emit portable Markdown, never silently mutate tracker.

Resolution Record (consumers embed in Scan Context / output header):
* **Platform:** [GitHub | GitLab | Bitbucket | Self-hosted:<host> | None] (`[Confidence: Level]`)
* **Evidence:** [git remote host | user-confirmed]
* **Tooling:** [<CLI> authenticated | git-only fallback — platform-dependent findings marked requires verification]