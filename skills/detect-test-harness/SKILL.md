---
name: detect-test-harness
description: Shared protocol for resolving a repository's test runner/framework and existing test layout from evidence before any test is read, scaffolded, or written. Provides a signal-file -> framework inference map, a per-framework idiom adapter (run command, file naming, test-double convention), a single confirmation question when inference is inconclusive, and a hard rule against silently introducing a new framework. Consumed by any skill that discovers or authors tests (audit-test-coverage, remediate-test-coverage).
dependencies:
  - interview-me  # Drives the single-question confirmation gate when the harness cannot be inferred
  - agent-markup  # Pulls [Confidence: Level] for the resolution record
  - design-vocab  # Seam / Adapter / Interface terms used to describe the test-double idiom
---

## Purpose

No skill may read, scaffold, or write tests without first resolving the project's existing test harness through this protocol. Frameworks, run commands, file-naming conventions, and test-double idioms are NOT consistent across stacks, and guessing wrong either misses real tests during an audit or introduces a foreign framework during remediation. This skill is the single source of truth for that resolution so behaviour never drifts between consumers.

The cardinal rule: **match the project's existing framework and style — never introduce a new test framework, runner, or assertion library silently.** A missing or ambiguous harness is resolved by a single question, not an assumption.

## Resolution Protocol

1. **PHASE 1 (Signal Inference):** Scan the repository root and manifests for the signal files in the map below. Record every framework whose signals are present — a polyglot repo may resolve to more than one harness (e.g. a React Native app with a Python service). Assign each resolution a `[Confidence: Level]`: a runner config file or test dependency directly present is `Confirmed`; a language manifest present with no test dependency declared is `Probable`; a bare source-file extension with no manifest corroboration is `Possible`.
2. **PHASE 2 (Layout Discovery):** For each resolved framework, locate the physical test layout (e.g. `tests/`, `test/`, `__tests__/`, `*_test.go`, `integration_test/`, co-located `*.spec.ts`). Capture the existing naming convention, fixture/helper locations, and the declared run command from the manifest. Consumers copy these conventions rather than inventing new ones.
3. **PHASE 3 (Confirmation Gate):** IF no signal is found, OR the highest confidence is below `Confirmed`, OR two frameworks of the same tier compete with no existing tests to disambiguate — trigger `interview-me` for exactly ONE question: ask the user to declare the runner (and, where relevant, the test type the new harness should target). Do NOT scaffold a harness or pick a framework from a bare language manifest alone. If tests already exist, prefer their framework over any question.
4. **PHASE 4 (No-Silent-Introduction Guard):** A consumer that needs to author tests MUST verify the resolved framework is already a project dependency before writing. If it is not (greenfield test suite), the choice of framework is itself a mutation requiring the Phase 3 question and explicit user approval — never add a test dependency unannounced.

## Signal -> Framework Inference Map

*(Signals are ILLUSTRATIVE and non-exhaustive; presence of any row signal resolves that framework. Confirm the run command against the manifest at runtime — never assert a command you did not read.)*

| Signal files / markers | Stack | Typical Runner(s) | Illustrative Run Command | Test File Convention |
| :--- | :--- | :--- | :--- | :--- |
| `pubspec.yaml` + `flutter_test` in dev_dependencies, `*_test.dart`, `integration_test/` | Flutter / Dart | `flutter test` / `dart test` | `flutter test`, `flutter test integration_test` | `*_test.dart` |
| `*.Tests.csproj`, `Microsoft.NET.Test.Sdk`, `xunit` / `nunit` / `mstest` package refs | .NET | xUnit / NUnit / MSTest | `dotnet test` | `*Tests.cs` / `*.Tests` project |
| `metro.config.*`, `react-native` in `package.json` | React Native | Jest (+ Detox/Maestro e2e) | `npm test`, `detox test` | `*.test.tsx` / `__tests__/` |
| `package.json` + `jest.config.*` / `vitest.config.*` / `playwright.config.*` (non-RN) | TypeScript / Node.js | Jest / Vitest / Playwright | `npm test`, `npx vitest`, `npx playwright test` | `*.test.ts` / `*.spec.ts` / `__tests__/` |
| `pytest.ini`, `conftest.py`, `pyproject.toml [tool.pytest]`, `test_*.py` | Python | pytest (or `unittest`) | `pytest`, `python -m pytest` | `test_*.py` / `*_test.py` |
| `go.mod`, `*_test.go` | Go | `go test` | `go test ./...` | `*_test.go` |
| `Cargo.toml`, `#[cfg(test)]`, `tests/` | Rust | `cargo test` | `cargo test` | `tests/*.rs` + inline `#[test]` |
| `pom.xml` / `build.gradle` + `junit` / `testng` | JVM (Java/Kotlin) | JUnit / TestNG | `mvn test`, `gradle test` | `*Test.java` / `*Spec.kt` |

## Per-Framework Test-Double Idiom (Seam Adapter Map)

*(Consumed when authoring Interface-verifying tests that fake/mock Adapters at Seams. Names the idiomatic doubling mechanism so remediation fakes at the Seam in the project's native style rather than importing a foreign mocking library.)*

| Stack | Idiomatic Fake/Mock at the Seam | Note |
| :--- | :--- | :--- |
| Flutter / Dart | `mocktail` / `mockito` (codegen) | Prefer hand-written fakes for stable Seams; reserve generated mocks for interaction ordering. |
| .NET | `Moq` / `NSubstitute`; hand-rolled fakes | Inject via constructor; fake at the interface, not the concrete type. |
| TypeScript / Node.js | `vi.fn` / `jest.fn`, manual fakes | Avoid `jest.mock` of the module under test — fake its Seam collaborators only. |
| Python | `unittest.mock` / pytest fixtures | Wrap foreign Interfaces behind a Seam, then fake the wrapper — do not patch what you do not own. |
| Go | hand-written fakes satisfying the interface | Idiomatic Go fakes a small interface at the Seam; mocking libraries are the exception. |
| Rust | trait objects / generic test doubles | Inject a trait impl; double at the trait Seam. |
| JVM | Mockito / fakes | Constructor injection; mock the collaborator interface, never the unit under test. |

## Operational Directives

- **Evidence-Before-Assumption:** Resolve from signal files and existing tests first; ask exactly ONE `interview-me` question only when evidence is inconclusive. Re-use the answer for the rest of the run — never re-prompt.
- **Never-Introduce-Silently:** Matching the existing framework is mandatory. Adopting a new framework, runner, or assertion/mocking library is a mutation that requires explicit user approval — never add a test dependency unannounced.
- **Copy-the-Conventions:** New tests inherit the discovered file location, naming, fixtures, and helpers. Do not impose a different layout on a repo that already has one.
- **Vocabulary Compliance:** Describe doubling in `design-vocab` terms — fake/mock Adapters at Seams verifying a Module's Interface. Use "unit/integration/e2e" only when naming a physical folder path (e.g. `tests/unit`).
- **Honest Confidence:** Tag every resolution `[Confidence: Level]`. Never assert a run command you did not read from a manifest; mark inferred commands `Possible — requires verification`.

## Resolution Record (consumers embed this in their own Scan Context / output header)

* **Harness:** [framework(s) resolved] (`[Confidence: Level]`)
* **Evidence:** [signal files matched | existing tests observed | user-confirmed via gate]
* **Run command:** [command read from manifest | `requires verification`]
* **Layout & conventions:** [test paths + naming observed]
