# Feature Specification: Minimal Record-Replay Testing Framework

**Feature Branch**: `1-record-replay-framework`
**Created**: 2025-11-04
**Status**: Draft
**Input**: User description: "Minimal open testing/contract framework that records API request/response pairs and replays them for regression testing. Include purpose and high‑level architecture (library + CLI), primary use cases (recording, replaying, validating, exporting fixtures), the data model for recorded events (timestamp, origin, request, response, metadata like env and version), default normalization rules for ambiguous/non‑deterministic fields and how to configure them, security/privacy considerations (sanitization, secret masking, opt‑in/opt‑out), performance/scale constraints for local demo (target: up to hundreds of recordings), configurability (JSON/YAML formats, storage layout, stubbing external calls), and error handling/failure modes (mismatched responses should produce a diff and optional auto‑update workflow). Use reasonable defaults when unspecified and explicitly list any assumptions. Acceptance: clear architecture and data model, normalization rules, security guidelines, file formats, and basic error handling; assumptions flagged."

## Summary

A minimal, open record-replay testing and contract framework that captures HTTP API
request/response pairs and replays them deterministically for regression testing and
contract validation. The tool is delivered as a lightweight library (importable in
languages that can call an HTTP client hook) plus a cross-platform CLI that records
traffic, replays fixtures, validates responses, and exports/imports fixtures.

Primary goals:

- Make it easy to record real interactions (manual or CI-driven) and produce
  portable fixtures for tests.
- Provide deterministic replay with configurable normalization to handle
  non-deterministic fields (timestamps, ids, auth tokens).
- Keep defaults simple: JSON fixtures, file-based storage, no external server
  required for local demos (scale: hundreds of recordings).
- Provide secure defaults (mask secrets, enable opt-in recording for sensitive
  endpoints) and clear guidance for sanitization.

## Technical Context

**Language/Version**: Language-agnostic library interface; CLI implementation
is left as an implementation choice for the PoC and SHOULD be documented in the
plan (see Assumptions).

**Primary Dependencies**: Minimal runtime deps; prefer standard libraries and a
small JSON/YAML library. New dependencies MUST be justified in plan.

**Storage**: File-based fixtures by default (directory layout below). Optional
pluggable storage backends (S3, artifact stores) MAY be added later.

**Testing**: Fixtures are consumable by unit/integration test runners. CLI must
provide `record`, `replay`, `validate`, and `export` commands.

**Target Platform**: Local dev machines and CI agents.

**Performance Goals**: Local demo: up to hundreds of recorded events with quick
lookup/replay (<100ms per lookup on average for typical fixtures). Not intended
for high-throughput production capture.

## User Scenarios & Testing (mandatory)

### User Story 1 - Record interactions (Priority: P1)

A developer wants to capture the request/response pairs produced when their
service calls an external API and save them as fixtures for tests.

**Why this priority**: Recording provides ground-truth data and is the basis for
replay and validation.

**Independent Test**: Use CLI `record` while exercising the code path; confirm
fixtures are written to the configured directory and include request, response,
and metadata.

**Acceptance Scenarios**:

1. **Given** a running application and CLI invoked in record mode,
   **When** an outgoing HTTP request is made to a target endpoint,
   **Then** the framework writes a fixture file containing timestamp, origin,
   request (method, URL, headers, body), response (status, headers, body), and
   metadata (env, recorded-by, tool-version).

2. **Given** recording is enabled with sensitive-field masking,
   **When** the response contains auth tokens,
   **Then** tokens are masked in the stored fixture per configuration.

---

### User Story 2 - Replay interactions (Priority: P1)

A developer wants to run tests that replay recorded responses without hitting
real external services.

**Independent Test**: Run CLI `replay` or library replay hook; code under test
should receive recorded responses matching the saved fixtures.

**Acceptance Scenarios**:

1. **Given** recorded fixtures exist for endpoint X,
   **When** tests are run with replay enabled,
   **Then** HTTP calls to X return the recorded responses deterministically.

2. **Given** a recorded fixture is missing for a call,
   **When** replay mode is active,
   **Then** the framework fails the test with a clear error describing the
   missing fixture and the originating call.

---

### User Story 3 - Validate & compare (Priority: P2)

A CI job wants to assert that the current behavior of an API client still matches
previously recorded responses (contract testing).

**Independent Test**: Run `validate` command that replays requests against
fixtures and compares live responses (or compares current implementation output
against expected fixtures).

**Acceptance Scenarios**:

1. **Given** fixtures and a validation run,
   **When** the live response differs from recorded fixture beyond tolerated
   normalization rules,
   **Then** the CLI outputs a human-friendly diff and returns a non-zero exit
   code.

2. **Given** auto-update workflow enabled,
   **When** the diff is acceptable and user confirms,
   **Then** fixtures are updated atomically and a changelog entry is recorded.

---

### User Story 4 - Export & share fixtures (Priority: P3)

A team wants to export fixtures to share with other teams or persist in an
artifact store.

**Independent Test**: Use `export` to produce a gzipped archive of fixtures in
JSON or YAML and confirm import on another machine reproduces the same fixtures.

## Edge Cases

- Request ordering differences: Replays that depend on sequential state must be
  recorded with explicit ordering metadata. If missing, replay can fail or use a
  best-effort matching rule (configurable).
- Non-deterministic headers (e.g., Date, Request-Id): Must be normalized or
  ignored by default.
- Binary payloads: Stored as base64 in JSON fixtures or natively in file-based
  formats.

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST record outgoing HTTP requests and corresponding
  responses to fixture files with the data model defined below.
- **FR-002**: The system MUST be able to replay fixtures so that code under test
  receives recorded responses without network calls.
- **FR-003**: The system MUST support normalization rules (default set + user
  config) to treat non-deterministic fields consistently.
- **FR-004**: The system MUST provide CLI commands: `record`, `replay`,
  `validate`, and `export`.
- **FR-005**: The system MUST provide configuration via a file (JSON/YAML) and
  environment variables.
- **FR-006**: The system MUST mask or sanitize sensitive fields by default when
  configured to do so and provide opt-in/opt-out controls per endpoint.
- **FR-007**: The system MUST produce machine-parseable diffs when validation
  fails and return a non-zero exit code for CI.
- **FR-008**: The system SHOULD offer an optional auto-update workflow that
  updates fixtures when authorized by a user/maintainer.

### Key Entities

- **RecordedEvent**: The primary fixture record capturing a single request/response.
  - timestamp (ISO 8601)
  - origin (hostname/service that initiated the request)
  - request:
    - method
    - url (path + query)
    - headers
    - body (string or base64 for binary)
  - response:
    - status
    - headers
    - body (string or base64)
  - metadata:
    - env (e.g., 'dev', 'ci')
    - recorded_by (CLI user/hostname)
    - tool_version
    - fixture_id (UUID)
    - order_index (optional integer for ordered flows)

- **NormalizationRule**: Named rule that transforms or masks fields before
  comparison (see Normalization section).

- **FixtureSet**: Directory of RecordedEvent files with an index file
  (`fixtures.index.json`) for quick lookup and manifest data.

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Recording a simple HTTP flow produces fixtures that replay in
  < 2 seconds end-to-end in local demo for up to 200 recorded events.
- **SC-002**: Validation diffs for mismatched responses are generated and the
  CLI returns non-zero exit in CI runs.
- **SC-003**: Default sanitization masks secrets in at least 90% of common auth
  header and cookie patterns when enabled.
- **SC-004**: Export/Import round-trip reproduces fixtures with byte-identical
  request/response bodies for non-normalized fields.

## Data Model for Recorded Events

A fixture file is a JSON object matching `RecordedEvent` above. Example:

```json
{
  "timestamp": "2025-11-04T12:00:00Z",
  "origin": "service-a",
  "request": {
    "method": "GET",
    "url": "/api/users?active=1",
    "headers": {"Accept": "application/json"},
    "body": ""
  },
  "response": {
    "status": 200,
    "headers": {"Content-Type": "application/json"},
    "body": "{\"users\":[]}"  
  },
  "metadata": {
    "env": "ci",
    "recorded_by": "dev-machine",
    "tool_version": "0.1.0",
    "fixture_id": "6f1a7c2e-...",
    "order_index": 1
  }
}
```

Fixtures are stored under `fixtures/<feature-name>/` with file names
`<timestamp>_<fixture_id>.json` and a `fixtures.index.json` manifest for fast
lookup.

## Default Normalization Rules

Normalization rules are applied before comparison. Defaults below can be
overridden in a config file (`.rrconfig.json` or `.rrconfig.yaml`). Rules are
applied in order.

1. Timestamp normalization: Remove or replace timestamps in headers and bodies
   that match ISO8601 or common timestamp patterns. Default: REPLACE with
   `<TIMESTAMP>` token.
2. UUID/ID normalization: Replace UUID-like tokens in bodies/paths with
   `<ID>` unless explicitly whitelisted.
3. Dynamic header removal: Ignore headers commonly non-deterministic
   (e.g., `Date`, `X-Request-Id`, `X-Correlation-Id`, `Server`).
4. Authorization masking: Replace bearer tokens (Authorization header
   `Bearer ...`) with `<REDACTED_TOKEN>` unless opted-out for the endpoint.
5. Numeric tolerance (optional): For fields that vary within small numeric
   ranges, allow +/- N tolerance when configured.

Configuration example (JSON):

```json
{
  "normalization": {
    "timestamp": true,
    "uuid": true,
    "ignore_headers": ["Date","X-Request-Id","Server"],
    "auth_masking": true
  }
}
```

## How to Configure Normalization

- Global config file: `.rrconfig.json` or `.rrconfig.yaml` at repository root.
- Per-fixture overrides: `fixtures/<set>/.rrconfig.json` for local adjustments.
- CLI flags: `--no-normalize`, `--normalize-timestamps`, `--mask-auth`.
- Programmatic API: Pass a NormalizationPolicy object to the library.

## Security & Privacy Considerations

- Sanitization by default: The tool MUST offer built-in sanitizers for common
  secret carriers (Authorization headers, cookies, query params with `token` or
  `password` keys). When sanitization is enabled these fields are masked in
  stored fixtures.
- Opt-in recording: Users MUST be able to mark endpoints or request/response
  bodies as sensitive and opt them out of recording (or store in encrypted
  storage using a configured key).
- Secrets policy: The default behavior is to mask secrets; explicit opt-out is
  required to store raw secrets. The plan MUST include a review of any
  environments that allow storing raw secrets.
- Access control: Fixtures SHOULD be treated as sensitive artifacts and stored
  with appropriate repository access controls or artifact storage rules.
- Audit trail: Each fixture MUST include `recorded_by` and `tool_version` to
  aid audits.

## File Formats & Storage Layout

Defaults:

- File format: JSON fixtures (UTF-8), optional YAML export/import.
- Layout:
  - `fixtures/<feature-or-suite>/` - contains `<timestamp>_<fixture_id>.json`
  - `fixtures/<feature-or-suite>/fixtures.index.json` - manifest with mapping
    keys -> fixture files and quick metadata.
  - `exports/` - gzipped archives created by `export` command.

Naming and manifest are designed for simple file lookups and for portability.

## Configurability

- Formats supported: JSON (default) and YAML (optional). CLI flag `--format`.
- Storage backends: Local filesystem (default). Pluggable adapters for S3,
  GCS, or artifact stores via plugin interface.
- Stubbing rules: Provide match strategies:
  - exact match (method + URL + body)
  - path-match with parameter normalization
  - header-insensitive match
  - best-effort heuristics (for best-effort replays)
- Auto-update policy: `--auto-update=ask|always|never` controls how fixture
  updates are applied in validation mode.

## Error Handling & Failure Modes

- Missing fixture: Fail fast in replay mode with a non-zero exit code and an
  actionable error specifying request details and the expected fixture key.
- Mismatched response: Produce a structured diff (JSON & human readable). Exit
  non-zero for CI. Diff includes:
  - fields that differ after normalization
  - raw values (both sides) unless masked by security rules
  - suggestion: command to update fixture (if auto-update permitted)
- Corrupted fixture: Detect invalid JSON/YAML and mark a failure with a clear
  error and the filename.
- Partial matches: If multiple fixtures partially match a request, the framework
  should either fail with an ambiguity error or apply a deterministic tie-break
  policy (configurable). Default: fail and require explicit matching config.

## Acceptance Criteria

- Clear architecture (library + CLI) described above.
- Data model and fixture layout defined and example provided.
- Default normalization rules described and configurable.
- Security guidelines included (sanitization, opt-in/opt-out, masking).
- File formats (JSON/YAML) and storage layout specified.
- Basic error handling and auto-update workflow described.

## Assumptions

1. Implementation language is not specified; the spec is language-agnostic.
   (If you want an immediate PoC, indicate preferred language.)
2. Local demo scale target is modest (up to ~200-500 fixtures). Not intended
   for streaming capture at massive scale.
3. Authentication and secret patterns follow common header/cookie/query naming
   conventions; custom patterns can be configured.
4. The project will add a small lockfile or pinning mechanism when adding
   external dependencies.


## Next Steps

1. Confirm implementation language for PoC (Assumption #1).  
2. Decide whether to implement CLI in the same language as the library or as a
   separate binary.  
3. Create an initial PoC that supports JSON fixtures, `record` and `replay` for
   a single HTTP client integration, and the default normalization rules.


