---
name: browser-verification
description: 'Shared protocol for browser-driven verification of rendered UI. Defines the capability (drive Google Chrome via osascript to open local prototypes and capture screenshots), the macOS prerequisites (Chrome installed, Automation permission, Zed restart caveat, JS-from-Apple-Events for scripted interaction), and the screenshot evidence convention (capture into a project-scoped temp dir, read with the image reader, delete before commit). Consumed by design-facing skills (designer, prototype-ui) and the adversarial-review UI path.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.0.0
dependencies:
  - agent-markup
user-invocable: false
---

Static markup inspection cannot prove rendered behaviour. When a task requires verifying rendered UI — layout, interaction states, in-browser accessibility — drive a real browser and verify from screenshot evidence.

Capability:
- Drive Google Chrome via `osascript` (macOS AppleScript): open `file://` prototypes or localhost URLs, navigate, capture screenshots via `screencapture`.
- Read the screenshot with the image reader to verify rendered output.
- Illustrative commands (verify at runtime):
  - Open: `osascript -e 'tell application "Google Chrome" to open location "<url>"'`
  - Capture: `screencapture -x <project-temp-dir>/shot.png`

Prerequisites (each missing item costs a round-trip — check before first use):
1. Chrome installed (`/Applications/Google Chrome.app`). Absent → static inspection only; request install approval — never install silently.
2. macOS Automation permission: the first `osascript` targeting Chrome triggers the system prompt. Denied or previously denied → error `-1743`; fix in System Settings → Privacy & Security → Automation → enable the controlling app (Zed/Terminal) for Google Chrome.
3. Zed restart caveat: a running Zed and its terminals do not inherit a newly granted Automation permission — restart Zed, then retry.
4. Scripted interaction (`execute javascript`) additionally requires Chrome View → Developer → Allow JavaScript from Apple Events.

Screenshot evidence convention:
1. Capture or copy screenshots into a project-scoped temp dir (e.g. `.tmp/browser-verification/`) — the image reader is project-scoped and cannot read outside the project root.
2. Read and verify the screenshot; cite it as evidence in findings.
3. Delete the temp dir before commit; never commit screenshots unless the user asks.

Directives:
- Evidence over assertion: a rendered-UI claim is `[Confidence: Confirmed]` only with a screenshot read; otherwise `Possible — requires verification`.
- Graceful degradation: no Chrome or no permission → fall back to static inspection and state the limitation; never fabricate rendered evidence.
- Cleanup: screenshots are transient evidence, never repo artefacts.
