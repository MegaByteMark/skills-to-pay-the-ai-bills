---
name: resolve-repository-platform
description: Shared protocol for determining which hosting platform a repository lives on (GitHub, GitLab, Bitbucket, self-hosted, or none) before any platform-specific tooling is invoked. Provides host inference, a user confirmation gate when inference is inconclusive, a platform->CLI/terminology adapter map, and a git-only graceful-degradation fallback. Consumed by any skill that would otherwise hard-code GitHub assumptions.
dependencies:
  - interview-me  # Drives the single-question confirmation gate when the platform cannot be inferred
  - agent-markup  # Pulls [Confidence: Level] for the resolution record
---

Purpose:
  No skill may invoke platform-specific tooling (e.g. the GitHub CLI `gh`) without first resolving the repository's hosting platform through this protocol. Clients run on GitHub, GitLab, Bitbucket, self-managed instances, or no remote at all; tooling and terminology are NOT consistent across them. This skill is the single source of truth for that resolution so the behaviour never drifts between consumers.

Neutral Vocabulary (use these terms in prose; map to platform terms only at the tooling layer):
  - Change Proposal: the platform-neutral umbrella for a GitHub/Bitbucket "Pull Request" and a GitLab "Merge Request". Never hard-code "Pull Request" or "PR" in shared prose.
  - Review Discussion: the comment/review thread attached to a Change Proposal.
  - Repository Custody: the namespace owning the repository (individual account vs. company/organisation).
  - Repository Visibility: whether the repository is public, internal, or private.
  - Work Item: the platform-neutral umbrella for a tracked unit of work — a GitHub/GitLab/Bitbucket "issue", a GitLab "Epic", a Jira "story/epic". Never hard-code "issue" in shared prose.
  - Parent Link: the hierarchy relation between work items (epic -> story / parent issue -> sub-issue). Term and mechanism vary by platform; resolve it here, not in consumers.

Resolution Protocol:
1. PHASE 1 (Host Inference): Read the remote with `git remote -v`. Parse the host of the origin (or first) remote:
     - `github.com` (or a known GitHub Enterprise host) -> GitHub
     - `gitlab.com` (or a known self-managed GitLab host) -> GitLab
     - `bitbucket.org` (or a Bitbucket Data Center host) -> Bitbucket
     - any other host (Azure DevOps, Gitea, self-hosted, unknown) -> UNRESOLVED
     - no remote configured -> NO-REMOTE
   Record the inference and the confidence it carries (`[Confidence: Level]`): a recognised public host is `Confirmed`; a host matched only by heuristic naming is `Probable`.
2. PHASE 2 (Confirmation Gate): IF the result is UNRESOLVED, or inference confidence is below `Confirmed`, or the consuming skill needs platform-only data (Change Proposal threads, Repository Visibility) that git alone cannot supply — trigger `interview-me` for exactly ONE question: ask the user to declare the platform (and, for self-managed instances, the base host). Do NOT guess a self-hosted platform from naming alone. For NO-REMOTE, skip the gate and proceed git-only.
3. PHASE 3 (Capability Probe): Once the platform is known, verify its CLI is installed AND authenticated before relying on it (e.g. `gh auth status`, `glab auth status`). Treat an unauthenticated or absent CLI exactly as an unavailable platform.
4. PHASE 4 (Graceful Degradation): If no authenticated platform CLI is available, degrade to platform-neutral `git`-only evidence (commit log, remote namespace, filesystem contents). Any finding that depends on platform-only data — Change Proposal discussion, Repository Visibility via API — is then reported as `Possible — requires verification` and NEVER asserted.

Platform Adapter Map:
| Platform | Recognised Host | CLI | Authenticated? | Change Proposal Term | Illustrative Thread Retrieval | Illustrative Visibility / Owner Lookup |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| GitHub | github.com / GHE | `gh` | `gh auth status` | Pull Request (PR) | `gh pr list --state merged`, `gh pr view <n> --comments` | `gh repo view --json visibility,owner` |
| GitLab | gitlab.com / self-managed | `glab` | `glab auth status` | Merge Request (MR) | `glab mr list --merged`, `glab mr view <n>` (notes) | `glab repo view` / project API `visibility` |
| Bitbucket | bitbucket.org / Data Center | none first-party universal | n/a | Pull Request (PR) | REST API if a token is supplied, else unavailable | REST API `is_private` if a token is supplied |
| Self-hosted / Other | resolved via gate | per user declaration | per user declaration | per platform | per user-declared tooling, else git-only | git-only unless user supplies tooling |
| No remote | n/a | n/a | n/a | n/a | unavailable (git-only) | git-only (namespace unknown) |

*(Commands are ILLUSTRATIVE, not authoritative — exact flags vary by CLI version. Verify availability at runtime; never assert output you did not actually obtain.)*

Work-Item Authoring Adapter Map (write-side — for consumers that CREATE or amend tracked work, e.g. `create-epic`, `create-user-story`, `seed-backlog`):
| Platform | Work Item Term | Epic Representation | Illustrative Create / Amend / Close | Illustrative Parent Link |
| :--- | :--- | :--- | :--- | :--- |
| GitHub | Issue | Issue (conventionally `epic`-labelled; no first-class Epic type) | `gh issue create`, `gh issue edit <n>`, `gh issue close <n>` | native sub-issues via GraphQL `addSubIssue` (else a task-list / tracking issue) |
| GitLab | Issue | Native Epic (group-level; tier-gated) | `glab issue create`, `glab issue update <n>`, `glab issue close <n>` (Epics via API where licensed) | epic↔issue association, or `/epic` quick action in the description |
| Bitbucket | Issue | none native | REST API if a token is supplied, else unavailable | REST API only; no native epic hierarchy |
| Self-hosted / Other | per user declaration | per user declaration | per user-declared tooling, else unavailable | per user-declared tooling |
| No remote | n/a | n/a | unavailable — render the work item as portable Markdown instead of pushing | n/a |

*(Write tooling and flags vary more than read tooling — confirm at runtime. Where a platform lacks first-class Epics or sub-issues, degrade to the conventional substitute named above rather than asserting a capability that is not present.)*

## Operational Directives
- Resolve-Before-Invoke: A consuming skill MUST run this protocol and obtain a resolved platform (or an explicit git-only fallback) BEFORE issuing any platform-specific command. No skill may assume GitHub.
- Single-Gate Discipline: The confirmation gate asks at most ONE question via `interview-me` (platform identity, plus base host for self-managed). Re-use the answer for the remainder of the run; never re-prompt.
- Terminology Neutrality: In all shared and client-facing prose use the Neutral Vocabulary (Change Proposal, Review Discussion). Substitute the platform-specific term only inside platform-specific tooling steps.
- Honest Unavailability: When platform tooling is unavailable, state it plainly and degrade — never fabricate Change Proposal content or assert Repository Visibility without confirmation.
- Write-Side Safety: A consumer performing creates/amends/closes via the Work-Item Authoring map MUST obtain explicit user confirmation of the planned mutation set before issuing any write command, and degrade to emitting portable Markdown when no authenticated CLI is available — never silently mutate a tracker.

Resolution Record (consumers embed this in their own Scan Context / output header):
* **Platform:** [GitHub | GitLab | Bitbucket | Self-hosted:<host> | None] (`[Confidence: Level]`)
* **Evidence:** [git remote host | user-confirmed via gate]
* **Tooling:** [<CLI> authenticated | git-only fallback — platform-dependent findings marked requires verification]
