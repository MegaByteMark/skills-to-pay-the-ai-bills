---
name: detect-test-harness
description: Shared protocol for resolving a repository's test runner/framework and existing test layout from evidence before any test is read, scaffolded, or written. Provides a signal-file -> framework inference map, a per-framework idiom adapter (run command, file naming, test-double convention), a single confirmation question when inference is inconclusive, and a hard rule against silently introducing a new framework. Consumed by any skill that discovers or authors tests (audit-test-coverage, remediate-test-coverage).
dependencies:
  - interview-me
  - agent-markup
  - design-vocab
---
No skill may read/scaffold/write tests without first resolving the project's existing test harness through this protocol. Cardinal rule: match existing framework and style — NEVER silently introduce a new test framework, runner, or assertion library.

Resolution Protocol:
1. PHASE 1 (Signal Inference): Scan repo root + manifests for signal files in map below. Record each matching framework. Polyglot repos may resolve >1 harness. Confidence: runner config/dependency present = `Confirmed`; language manifest present, no test dependency = `Probable`; bare source-file extension, no manifest corroboration = `Possible`.
2. PHASE 2 (Layout Discovery): For each resolved framework, locate physical test layout (e.g. `tests/`, `test/`, `__tests__/`, `*_test.go`, `integration_test/`, `*.spec.ts`). Capture naming convention, fixture/helper locations, declared run command from manifest.
3. PHASE 3 (Confirmation Gate): No signal, or highest confidence < `Confirmed`, or two same-tier frameworks compete with no existing tests → `interview-me` ONE question: declare runner (+ test type for new harness). If tests already exist, prefer their framework over any question.
4. PHASE 4 (No-Silent-Introduction Guard): Consumer authoring tests MUST verify resolved framework is already a project dependency. If not (greenfield test suite), framework choice = mutation requiring Phase 3 question + explicit user approval.

Signal → Framework Inference Map:
| Signal files / markers | Stack | Runner(s) | Illustrative Run Command | Test File Convention |
| :--- | :--- | :--- | :--- | :--- |
| `pubspec.yaml` + `flutter_test`, `*_test.dart`, `integration_test/` | Flutter/Dart | `flutter test` / `dart test` | `flutter test`, `flutter test integration_test` | `*_test.dart` |
| `*.Tests.csproj`, `Microsoft.NET.Test.Sdk`, xunit/nunit/mstest | .NET | xUnit/NUnit/MSTest | `dotnet test` | `*Tests.cs` / `*.Tests` project |
| `metro.config.*`, `react-native` in `package.json` | React Native | Jest (+ Detox/Maestro e2e) | `npm test`, `detox test` | `*.test.tsx` / `__tests__/` |
| `package.json` + `jest.config.*`/`vitest.config.*`/`playwright.config.*` (non-RN) | TypeScript/Node.js | Jest/Vitest/Playwright | `npm test`, `npx vitest`, `npx playwright test` | `*.test.ts` / `*.spec.ts` / `__tests__/` |
| `pytest.ini`, `conftest.py`, `pyproject.toml [tool.pytest]`, `test_*.py` | Python | pytest/unittest | `pytest`, `python -m pytest` | `test_*.py` / `*_test.py` |
| `go.mod`, `*_test.go` | Go | `go test` | `go test ./...` | `*_test.go` |
| `Cargo.toml`, `#[cfg(test)]`, `tests/` | Rust | `cargo test` | `cargo test` | `tests/*.rs` + inline `#[test]` |
| `pom.xml`/`build.gradle` + junit/testng | JVM (Java/Kotlin) | JUnit/TestNG | `mvn test`, `gradle test` | `*Test.java` / `*Spec.kt` |

Per-Framework Test-Double Idiom (Seam Adapter Map):
| Stack | Idiomatic Fake/Mock at Seam | Note |
| :--- | :--- | :--- |
| Flutter/Dart | `mocktail` / `mockito` (codegen) | Prefer hand-written fakes for stable Seams |
| .NET | `Moq` / `NSubstitute`; hand-rolled fakes | Inject via constructor; fake at Interface, not concrete type |
| TypeScript/Node.js | `vi.fn` / `jest.fn`, manual fakes | Avoid mocking module under test — fake its Seam collaborators only |
| Python | `unittest.mock` / pytest fixtures | Wrap foreign Interfaces behind Seam, fake wrapper — do not patch what you don't own |
| Go | hand-written fakes satisfying Interface | Idiomatic Go fakes small Interface at Seam; mocking libraries exception |
| Rust | trait objects / generic test doubles | Inject trait impl; double at trait Seam |
| JVM | Mockito / fakes | Constructor injection; mock collaborator Interface, never unit under test |

Directives:
- Evidence-Before-Assumption: signal files + existing tests first; ONE `interview-me` question only when inconclusive. Re-use answer rest of run.
- Never-Introduce-Silently: matching existing framework mandatory. New framework/runner/mocking library = mutation requiring user approval.
- Copy-the-Conventions: new tests inherit discovered file location, naming, fixtures, helpers. Never impose different layout.
- Strict `design-vocab`: fake/mock Adapters at Seams verifying Module's Interface. Use "unit/integration/e2e" only for physical folder paths.
- Honest Confidence: tag every resolution `[Confidence: Level]`. Never assert run command not read from manifest; `Possible — requires verification`.

Resolution Record:
* **Harness:** [framework(s)] (`[Confidence: Level]`)
* **Evidence:** [signal files | existing tests | user-confirmed]
* **Run command:** [from manifest | `requires verification`]
* **Layout & conventions:** [paths + naming observed]