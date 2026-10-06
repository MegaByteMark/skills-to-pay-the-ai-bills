---
name: browser-verification
description: 'Shared protocol for browser-driven verification of rendered UI, cross-platform (macOS, Windows, Linux). Defines the capability (drive an installed Chrome/Chromium to open local prototypes, interact, and capture screenshots), mechanism selection (Chrome DevTools Protocol for scripted interaction; OS-native tooling for open-and-capture), per-platform permission caveats, and the screenshot evidence convention (capture into a project-scoped temp dir, read with the image reader, delete before commit). Consumed by design-facing skills (designer, prototype-ui) and the adversarial-review UI path.'
license: MIT
metadata:
  author: MegaByteMark
  version: 1.1.0
dependencies:
  - agent-markup
user-invocable: false
---

Static markup inspection cannot prove rendered behaviour. When a task requires verifying rendered UI — layout, interaction states, in-browser accessibility — drive a real browser and verify from screenshot evidence. Resolve the mechanism for the current OS before first use.

Capability:
- Drive an installed Chrome/Chromium: open `file://` prototypes or localhost URLs, navigate, interact, capture screenshots.
- Read the screenshot with the image reader to verify rendered output.

Mechanism selection:
1. Scripted interaction (click, type, JS evaluation) → Chrome DevTools Protocol (CDP) on any platform; macOS may alternatively use AppleScript (`osascript`).
2. Open-and-capture only → OS-native tooling per the map.

Platform Adapter Map (illustrative — verify at runtime):
| Platform | Browser launch | Screenshot capture | Permission caveats |
| :--- | :--- | :--- | :--- |
| macOS | `open -a "Google Chrome" <url>` | `screencapture` | AppleScript control needs Automation permission (System Settings → Privacy & Security → Automation); denied → error `-1743`; JS execution needs Chrome View → Developer → Allow JavaScript from Apple Events |
| Windows | `Start-Process chrome <url>` | PowerShell .NET screen capture, or CDP | Screen capture may require an unlocked session |
| Linux | `google-chrome <url>` / `chromium <url>` | `gnome-screenshot`, `scrot`, `import`, or CDP | Wayland compositors may restrict screen capture |
| Any OS (CDP) | `<chrome-binary> --remote-debugging-port=<port> --user-data-dir=<temp-profile> <url>` | CDP `Page.captureScreenshot` | None beyond browser install; dedicated `--user-data-dir` required when Chrome is already running |

Prerequisites (each missing item costs a round-trip — check before first use):
1. Chrome/Chromium installed. Absent → static inspection only; request install approval — never install silently.
2. Permission caveats for the selected mechanism (see map). Denied → re-grant in OS settings, then retry.
3. Host-app restart caveat: a running editor/terminal may not inherit a newly granted OS permission — restart the host app, then retry.

Screenshot evidence convention:
1. Capture or copy screenshots into a project-scoped temp dir (e.g. `.tmp/browser-verification/`) — the image reader is project-scoped and cannot read outside the project root.
2. Read and verify the screenshot; cite it as evidence in findings.
3. Delete the temp dir before commit; never commit screenshots unless the user asks.

Directives:
- Evidence over assertion: a rendered-UI claim is `[Confidence: Confirmed]` only with a screenshot read; otherwise `Possible — requires verification`.
- Graceful degradation: no browser or no permission → fall back to static inspection and state the limitation; never fabricate rendered evidence.
- Cleanup: screenshots are transient evidence, never repo artefacts.
