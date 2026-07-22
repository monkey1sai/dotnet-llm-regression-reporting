## 1. Toolchain and Solution Bootstrap

- [ ] 1.1 Pin a supported .NET 10 LTS SDK in `global.json` and document Windows and Linux prerequisites
- [ ] 1.2 Create the solution and projects for Domain, Application, Providers.OpenAI, Artifacts, Reporting, Observability, CLI, and tests
- [ ] 1.3 Enable nullable reference types, analyzers, warnings-as-errors, deterministic builds, package lock files, and central package management
- [ ] 1.4 Add repository build, format, test, and package verification commands without configuring live endpoints

## 2. Versioned Contracts and Configuration

- [ ] 2.1 Define JSON Schemas and domain records for endpoint profiles, capability declarations, suites, scenarios, policies, baselines, canonical results, and retention manifests
- [ ] 2.2 Implement schema-version dispatch, canonical JSON serialization, and rejection of unknown versions
- [ ] 2.3 Implement secret-reference parsing and host resolution with tests proving plaintext credentials are rejected and redacted
- [ ] 2.4 Add safe example profiles and suites that contain no real endpoint, credential, or sensitive prompt data
- [ ] 2.5 Add content hashing and fixture-integrity validation for all referenced inputs

## 3. Provider and Protocol Boundary

- [ ] 3.1 Define provider-neutral request, response, streaming-event, usage, capability, and classified-error contracts
- [ ] 3.2 Implement the official OpenAI .NET SDK adapter with explicit custom endpoint, model, timeout, cancellation, and retry control
- [ ] 3.3 Implement the narrow protocol-compatible fallback for approved non-standard fields and SSE events
- [ ] 3.4 Add fixture-backed Chat Completions, Responses, Models, structured-output, tool-call, error, and streaming contract tests
- [ ] 3.5 Implement safe capability discovery and prove unsupported capabilities do not trigger requests
- [ ] 3.6 Add transport redaction tests covering headers, query values, bodies, exception messages, and echoed credentials

## 4. Regression Execution Engine

- [ ] 4.1 Implement suite discovery, normalization, deterministic ordering, and run-plan generation
- [ ] 4.2 Implement bounded concurrency, deadlines, cancellation, warm-up phases, measured phases, and partial-run finalization
- [ ] 4.3 Implement deterministic assertions for text, structured JSON, tool calls, error contracts, and capability expectations
- [ ] 4.4 Implement the terminal status and reason-code taxonomy with scenario isolation tests
- [ ] 4.5 Enforce the inference/discovery operation allowlist and reject administrative actions before transport
- [ ] 4.6 Add regression tests proving reruns with the same seed produce the same plan

## 5. Performance and Context Measurement

- [ ] 5.1 Implement monotonic measurements for end-to-end latency, first byte, first token, stream duration, and inter-event gaps
- [ ] 5.2 Implement bounded workload schedules for concurrency, request rate, duration, input size, and output tokens
- [ ] 5.3 Implement sample aggregation, configured percentiles, minimum-sample qualification, and error-rate summaries
- [ ] 5.4 Record token provenance and calculate throughput only when the declared source supports it
- [ ] 5.5 Implement bounded context probes with explicit stop conditions and last-successful-bound evidence
- [ ] 5.6 Add deterministic tests proving warm-ups and retried discovery attempts cannot enter measured statistics

## 6. Baselines and Policy Gates

- [ ] 6.1 Implement immutable artifact-manifest validation and baseline reference loading
- [ ] 6.2 Implement candidate-to-baseline compatibility checks for suite, scenario, model, capabilities, streaming, concurrency, and generation parameters
- [ ] 6.3 Implement absolute gates, directional relative thresholds, warning/failure bands, and evidence sufficiency
- [ ] 6.4 Implement aggregate outcomes and stable CLI exit codes with exhaustive unit tests
- [ ] 6.5 Implement a separate audited baseline-promotion command that cannot run implicitly during evaluation
- [ ] 6.6 Add policy tests for missing, corrupt, and incomparable baselines and for higher-is-better and lower-is-better metrics

## 7. Evidence, Reports, and Observability

- [ ] 7.1 Implement immutable run bundles, canonical and partial result JSON, SHA-256 manifests, and provenance capture
- [ ] 7.2 Implement Markdown and self-contained offline HTML renderers sourced only from canonical results
- [ ] 7.3 Implement JUnit XML output with documented mappings for all canonical statuses
- [ ] 7.4 Implement deny-by-default payload persistence and end-to-end redaction before logs, telemetry, raw evidence, and reports
- [ ] 7.5 Add Microsoft.Extensions.Logging and optional OpenTelemetry traces, metrics, and logs without making telemetry authoritative
- [ ] 7.6 Add golden-file tests for JSON, Markdown, HTML, JUnit, redaction, integrity verification, and retention metadata

## 8. CLI, CI, and Pilot Gates

- [ ] 8.1 Implement CLI commands for validate, plan, run, compare, report, verify, and baseline promotion
- [ ] 8.2 Add command help, stable exit-code documentation, cancellation handling, and non-interactive CI behavior
- [ ] 8.3 Add xUnit v3 test projects and pin the approved Microsoft Testing Platform or compatibility runner path
- [ ] 8.4 Add CI jobs for restore-lock verification, build, analyzers, unit tests, contract fixtures, report golden files, and secret scanning
- [ ] 8.5 Run a non-production observation-only pilot and retain its redacted evidence bundle
- [ ] 8.6 Approve an initial baseline, enable warning-only policy evaluation, and document the review required before any blocking gate
