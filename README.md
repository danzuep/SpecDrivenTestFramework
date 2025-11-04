# Spec Driven Test Framework

A minimal, open record-replay testing and contract framework (Kotlin MVP).

This repository contains the specification, tasks, and initial planning for a
lightweight tool that records HTTP request/response pairs, normalizes and
stores them as fixtures, and replays them for deterministic regression and
contract testing.

Contents
- `.specify/memory/constitution.md` — project constitution and guiding
  principles (code quality, modularity, simple UX, minimal dependencies).
- `specs/1-record-replay-framework/` — feature specification, tasks, and
  checklists for the Kotlin PoC.
- `specs/1-record-replay-framework/spec.md` — concise spec describing
  architecture, data model, normalization rules, and security guidance.
- `specs/1-record-replay-framework/tasks.md` — prioritized tasks and a
  2-week sprint backlog for the Kotlin PoC.

Quickstart (Kotlin PoC)

Prerequisites
- JDK 17+ or compatible Java SDK
- Git
- Gradle (recommended via the Gradle wrapper)

Build

Windows PowerShell:

```powershell
.
# from repo root (if Gradle wrapper exists):
.
# build
.\gradlew.bat build
```

Unix/macOS:

```bash
# build
./gradlew build
```

Run tests

```bash
./gradlew test
```

Project layout (planned)
- `core/` — Kotlin library implementing RecordedEvent, Normalizer, Storage
- `cli/` — Kotlin CLI wrapping library features (record, replay, validate, export)
- `examples/` — demo server and client used for E2E tests
- `fixtures/` — default fixtures storage layout used by the PoC
- `specs/` — specs, tasks and checklists (this folder)
- `docs/` — user docs and quickstart guides

Developer DX
- See `specs/1-record-replay-framework/` for the current plan, spec, and task
  list. The plan targets a Kotlin Gradle PoC. If you want to run the PoC, build
  the project (Gradle) and follow the quickstart in `docs/quickstart.md` once
  the CLI artifacts are available.

Contribution
- Follow the constitution in `.specify/memory/constitution.md` — tests, code
  quality, and minimal dependencies are required. Open a PR against `main` and
  include links to the relevant spec or task.
