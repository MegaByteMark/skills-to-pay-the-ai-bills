---
name: agent-markup
description: Defines the strict token syntax, bracket-enclosed schema fields, and machine-readable value enumerations used for cross-agent parsing and automation.
license: MIT
metadata:
  author: MegaByteMark
  version: 1.4.0
---
All machine-readable tokens MUST be in square brackets `[...]`.

[Auth: Scope]: [Read, Write, Admin, None]

[Risk: Level]: [Low, Medium, High, Critical]

[Policy]: [Enforced, Advisory, Audit-Only]. Written `[Policy: Enforced]` etc. when tagging a concrete rule.

[Data: Classification]: [Public, Internal, Confidential, PII, Special-Category]. Special-Category = GDPR Art. 9 (health, biometrics, race, religion, sexual orientation, etc.) — strictest handling obligations.

[Confidence: Level]: [Confirmed, Probable, Possible]. Confirmed = directly observed; Probable = strong indicator missing one link; Possible = heuristic needing verification — phrase as "requires verification", never assert.

[Remediation: Effort]: [Low, Medium, High]

[Remediation: Action]: [Fix, Accept, Defer, Log, None]. Fix = resolve before proceeding; Accept = explicitly waive with recorded justification; Defer = park behind a tracked work item; Log = capture for opportunistic action; None = no action indicated. Paired with Effort when action is Fix.

[Competency: Level]: [Not-Ready, Paired, Guided, Solo]. Ordered weakest-to-strongest. Solo = wrote comparable work unaided; Guided = succeeds with full spec/acceptance criteria; Paired = succeeds only with scaffold + step-by-step prompts; Not-Ready = cannot do or explain. Self-report is provisional `[Confidence: Possible]` until corroborated by observed work. Owned by competency-profile.

[Inferred: Unverified]: [true]. Marks output from an ephemeral in-context baseline. Never applied to blueprint-drift findings.

[Priority: MoSCoW]: [Must, Should, Could, Wont]. Written `[Priority: Must]` etc. Must = release fails without it; Should = viable workaround; Could = desirable if capacity; Wont = explicitly out-of-scope (records decision). Owned by gather-requirements PRD stream.

[Review: Priority]: [Must, Should, Could, Nitpick]. Written `[Review: Must]` etc. Must = merge-blocking (breaks functionality, introduces vulnerability/violation, or regresses behaviour); Should = non-conformance with repo-resident standards; Could = opportunistic improvement within blast radius; Nitpick = trivial non-conformance (typos, inconsistency). Review-context only — never used in PRD or planning. Owned by adversarial-review.

[Doc: Archetype]: [QuickStart, Technical, Troubleshooting, Installation, Commentary]. Path bindings owned by document-a-codebase.

[Scope: Artefact]: [Release, Security-Governance, Health, Digest, Deployment, Change-Proposal]. Digest = client-facing weekly progress email from client-email-digest. Deployment = CI/CD or release coordination report from the devops persona and its leaves. Change-Proposal = PR/MR body from create-pr. Extend this enumeration here when adding report-producing skills.

[Section: Name]: [Motivation, Summary, Key-Decisions, Review-Focus, Verification]. Written `### [Section: Motivation]` etc. as stable machine-parseable headers. Enables AI consumers (e.g. adversarial-review) to regex-extract sections and lift `file:line` signposts into their own scope. Owned by create-pr.

[Scope: Origin]: [Pre-existing, Introduced]. Pre-existing = issue surfaced but not caused by the diff under review; Introduced = issue created by the diff. Used by adversarial-review to distinguish fix-in-PR from park-behind-bug-report.

Output Portability: All client-facing artefacts (FDS, blueprint, audits, docs) as export-clean Markdown — standard heading hierarchy, no renderer-fragile constructs, tables degrade gracefully. PDF/branding mechanism is separate.