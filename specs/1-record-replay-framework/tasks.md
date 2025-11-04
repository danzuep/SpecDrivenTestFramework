# Tasks: Record-Replay Framework (Kotlin PoC)

**Feature**: Minimal Record-Replay Testing Framework
**Language/PoC**: Kotlin (Gradle)
**Feature Dir**: `specs/1-record-replay-framework`

## Phase 1: Setup (Project scaffolding)

- [ ] T001 Create repository skeleton and Kotlin Gradle project (S) - src/main/kotlin, src/test/kotlin, cli/, fixtures/, docs/ - `./`, `build.gradle.kts`, `settings.gradle.kts`, `README.md`, `.gitignore`
- [ ] T002 Add development tooling and local run docs (S) - add `ktlint` config, `gradle wrapper` and README run instructions - `README.md`, `gradle/wrapper/` 
- [ ] T003 Add CI pipeline skeleton: GitHub Actions workflow for build, lint, tests (M) - `.github/workflows/ci.yml`
- [ ] T004 Create fixtures layout + example manifest (S) - create `fixtures/.rrconfig.example.json` and `fixtures/fixtures.index.json` template - `fixtures/`, `fixtures/fixtures.index.json`
- [ ] T005 Add CONTRIBUTING and pointer to constitution (S) - add `CONTRIBUTING.md` referencing `.specify/memory/constitution.md` - `CONTRIBUTING.md`

## Phase 2: Foundational (Core library IO & normalization)

- [ ] T006 [P] Implement `RecordedEvent` data model + file IO (M) - `src/main/kotlin/com/example/rr/model/RecordedEvent.kt`, `src/main/kotlin/com/example/rr/io/FixtureStore.kt` - serialize/deserialize JSON, write `<timestamp>_<uuid>.json`, update `fixtures/fixtures.index.json`
- [ ] T007 [P] Implement normalization engine (core rules + config) (L) - `src/main/kotlin/com/example/rr/normalize/NormalizationEngine.kt`, `src/main/resources/default-normalization.json`, `.rrconfig.json` support
- [ ] T008 [P] Implement recording hook/adapter for HTTP clients (OkHttp interceptor example) (M) - `src/main/kotlin/com/example/rr/hook/OkHttpRecordingInterceptor.kt` and `src/main/kotlin/com/example/rr/api/RecordingApi.kt`
- [ ] T009 [P] Implement fixture lookup & matching strategies (exact, path-normalize) (M) - `src/main/kotlin/com/example/rr/match/Matcher.kt`, `src/main/kotlin/com/example/rr/index/FixtureIndex.kt`
- [ ] T010 [P] Implement secret masking & sanitizers utilities (S) - `src/main/kotlin/com/example/rr/security/Sanitizers.kt`

## Phase 3: CLI (record, replay, validate, export)

- [ ] T011 CLI skeleton & command dispatch (M) - `cli/src/main/kotlin/com/example/rr/cli/Main.kt`, args parsing and help
- [ ] T012 `record` command: file-based recording with include/exclude (M) - `cli/src/main/kotlin/com/example/rr/cli/RecordCommand.kt`
- [ ] T013 `replay` command & matching flags (M) - `cli/src/main/kotlin/com/example/rr/cli/ReplayCommand.kt`
- [ ] T014 `validate` command: diff output and auto-update flow (L) - `cli/src/main/kotlin/com/example/rr/cli/ValidateCommand.kt`
- [ ] T015 `export` command: gzipped archive and import round-trip (S) - `cli/src/main/kotlin/com/example/rr/cli/ExportCommand.kt`

## Phase 4: Testing & CI

- [ ] T016 Unit tests for data model & IO (M) - tests in `src/test/kotlin/com/example/rr/io/` - assert write/read integrity, index updates
- [ ] T017 Unit & integration tests for normalization engine (M) - `src/test/kotlin/com/example/rr/normalize/` - cover timestamp, uuid, header rules
- [ ] T018 End-to-end record→replay demo test (P1) (M) - `tests/e2e/record_replay_demo.kt` - start local server, record, replay, assert equality
- [ ] T019 CI enforcement: require lint + tests on PRs (S) - update `.github/workflows/ci.yml` to block merges
- [ ] T020 Add test fixtures for edge cases & failure modes (S) - `tests/fixtures/` with ambiguous and corrupted fixtures

## Phase 5: Demo & Docs

- [ ] T021 Quickstart guide & examples (S) - `docs/quickstart.md`, update `README.md` with Kotlin build/run examples
- [ ] T022 Create sample demo app & fixtures (M) - `examples/demo-server/` and `examples/demo-client/` with script `scripts/run-demo.ps1` / `scripts/run-demo.sh`
- [ ] T023 API reference & config examples (`.rrconfig`) (S) - `docs/config.md`, `docs/normalization.md`
- [ ] T024 Security & privacy guidance + checklist (S) - `docs/security.md`, `docs/privacy-checklist.md`
- [ ] T025 Release notes template & v1.0.0 process (S) - `docs/release.md`

---

## Dependencies (high-level)

- `T006` depends on: T001, T004
- `T007` depends on: T006
- `T008` depends on: T006, T007
- `T009` depends on: T006, T007
- `T010` depends on: T007
- `T011` depends on: T001, T003, T006
- `T012` depends on: T008, T011, T006
- `T013` depends on: T009, T011, T006
- `T014` depends on: T007, T009, T011
- `T015` depends on: T006, T011
- `T016` depends on: T006
- `T017` depends on: T007
- `T018` depends on: T006, T008, T011, T013
- `T019` depends on: T003, T016, T017
- `T020` depends on: T006, T007, T009
- `T021` depends on: T011, T012, T013
- `T022` depends on: T006, T008, T012
- `T023` depends on: T007, T011
- `T024` depends on: T010
- `T025` depends on: T003, T019

## Dependency Graph (story completion order)

1. Phase 1 tasks (T001..T005)
2. Foundational: T006 → T007 → {T008, T009, T010}
3. CLI: T011 → {T012, T013} → T014 → T015
4. Testing: T016, T017 → T018 → T019
5. Docs & Demo: T021..T025

## Parallel execution opportunities

- T006 (IO) and T004 (fixtures layout) can run in parallel with T001/T002
- T007 (normalization) can start once T006 has a stable data model; T006 and T007 overlap
- T008, T009, T010 (adapters, matcher, sanitizers) are parallelizable after T006
- CLI subcommands T012 and T013 development can proceed in parallel after T011
- Unit tests (T016,T017) can be written while library code is implemented (parallel)

## Sprint Backlog (2-week sprint - Kotlin PoC)

Selected sprint tickets (fit for 2-week PoC):
- T001 (0.5-1d) - eng
- T002 (0.5d) - eng
- T003 (1d) - devops
- T004 (0.5d) - eng
- T006 (2d) - eng
- T007 (2d) - eng
- T008 (1d) - eng
- T011 (1d) - eng
- T012 (1d) - eng
- T016 (1d) - qa/eng
- T018 (1d) - qa/eng
- T021 (0.5d) - doc

Sprint totals: 12–13 work-days of effort; parallelism assumed between devops and eng. Adjust priorities if team headcount is smaller.

## Task counts & validation

- Total tasks: 25
- Tasks per milestone:
  - Project scaffolding: 5
  - Core library: 5
  - CLI: 5
  - Testing & CI: 5
  - Demo & Docs: 5

Format validation: All tasks follow checklist format `- [ ] T### [P?] [US?] Description with file path` with explicit file paths where applicable.

## Acceptance criteria for this artifact

- Tasks are granular and actionable with file paths and acceptance tests.
- Prioritized by milestone and dependency order.
- Sprint backlog chosen for a Kotlin PoC and sized to 2 weeks with assumptions noted.

---

If you'd like I will now:
- A) Commit `specs/1-record-replay-framework/tasks.md` and create branch `1-record-replay-framework` via the `.specify` script (I can run the script and git commands), or
- B) Start scaffolding the Kotlin PoC (create Gradle project, basic RecordedEvent classes, and the OkHttp interceptor). 

Which should I do next?