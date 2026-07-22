## Context

The project starts with no runtime implementation. Its purpose is to define a Windows-first, enterprise-grade regression and reporting system for OpenAI-compatible LLM endpoints. The system must distinguish model regressions from endpoint outages, preserve reproducible evidence, support streaming measurements, and integrate with CI without storing credentials or assuming full OpenAI API compatibility.

The primary stakeholders are platform operators, model owners, application teams, security reviewers, and release approvers. The design assumes endpoints may expose different subsets of Chat Completions, Responses, Models, Embeddings, streaming, structured output, and tool-calling behavior.

The target runtime is .NET 10 LTS. Windows is the first-class operator environment, while core libraries and the CLI remain portable to supported Linux runners. The local machine currently has .NET 9, so implementation must treat a .NET 10 SDK installation as an explicit prerequisite rather than silently retargeting to a shorter support release.

## Goals / Non-Goals

**Goals:**

- Provide deterministic, repeatable compatibility, quality, performance, and reliability regressions.
- Use explicit endpoint capability profiles instead of assuming that every OpenAI-compatible route exists.
- Produce canonical machine-readable evidence and concise human-readable reports.
- Make pass/fail policy versioned, reviewable, and independent from test execution.
- Keep secrets out of configuration, logs, reports, baselines, and source control.
- Support Windows local execution and CI with the same suite and policy contracts.
- Prefer supported platform libraries and focused dependencies over custom frameworks.

**Non-Goals:**

- Deploying or configuring LLM servers, model weights, MCP servers, browsers, or Stagehand.
- Replacing dedicated server telemetry, GPU profilers, or load-testing infrastructure.
- Claiming that client-side measurements alone explain GPU or scheduler bottlenecks.
- Building a web dashboard, distributed scheduler, multi-tenant service, or arbitrary plugin host in the first implementation.
- Making non-deterministic LLM-as-judge scores a blocking release gate without a separately approved calibration policy.
- Storing secret values or production prompts in the repository.

## Decisions

### 1. Use a layered .NET 10 LTS solution with explicit dependency direction

The future solution will separate domain contracts, orchestration, provider adapters, artifacts, reporting, observability, and host concerns:

```text
CLI / CI host
    -> Application orchestration
        -> Domain contracts and policy engine
        -> Provider abstractions
            -> Official OpenAI .NET SDK adapter
            -> Protocol-compatible fallback adapter
        -> Artifact store
        -> Report renderers
        -> OpenTelemetry instrumentation
```

Domain and policy projects will not reference HTTP, OpenAI SDK, console, filesystem, or report-rendering packages. This keeps comparison semantics testable and prevents provider-specific response shapes from becoming the public model.

Alternative considered: a single console project. It is faster to scaffold but creates direct coupling among secrets, transport, statistics, and reports, making audit and regression testing harder.

### 2. Use the official OpenAI .NET SDK behind an internal provider contract

The typed adapter will use the official OpenAI .NET SDK with an explicit custom endpoint, model name, API key credential, timeout, and cancellation token. A narrow protocol adapter will handle compatible-server deviations or raw JSON fields without forking or recreating the complete OpenAI client.

Each endpoint profile declares supported capabilities. The runner may verify declarations through safe discovery calls, but it will not infer full support from a successful health check.

Alternative considered: hand-written HttpClient code for every route. Rejected because it duplicates authentication, serialization, streaming, and error handling already maintained by reliable libraries. Protocol fallback remains intentionally narrow and test-driven.

### 3. Separate the regression runner from unit-test infrastructure

The product CLI will execute data-driven LLM scenarios and emit stable exit codes plus artifacts. xUnit v3 will test the runner, adapters, parsers, policy engine, and renderers. Microsoft Testing Platform is the preferred future test platform once the selected .NET 10 toolchain and CI integrations are pinned.

Dynamic remote LLM scenarios will not be modeled as ordinary unit tests because warm-up, scheduling, retries, baselines, and infrastructure-failure classification need coordinated run-level control. The runner will emit JUnit XML for CI systems that expect test-case results.

Alternative considered: one xUnit test per remote prompt. Rejected because independent test scheduling makes performance measurements and run-level evidence less deterministic.

### 4. Use versioned JSON contracts and content-addressed artifact bundles

Endpoint profiles, suite manifests, policy files, baselines, and canonical results will be versioned JSON documents validated against committed JSON Schemas. Prompt bodies may be referenced as UTF-8 files so sensitive or large inputs can be supplied outside the repository.

Each run creates an immutable artifact bundle containing a manifest, normalized results, environment metadata, report files, and SHA-256 digests. Baselines point to an approved bundle rather than copying mutable values into configuration. The initial implementation uses the filesystem; no database is required.

Alternative considered: YAML. Rejected for the initial contract because JSON has one built-in serializer, fewer ambiguous scalar rules, and easier canonical hashing in .NET. A future authoring layer may compile YAML into canonical JSON.

### 5. Treat performance runs as controlled experiments

Measured requests use no automatic retry. Warm-up samples are labeled and excluded. The runner records a deterministic seed, request order, concurrency schedule, timeout, cancellation, input size, output limit, streaming mode, and client/runtime metadata.

Core metrics include success and error counts, end-to-end latency, time to first byte, time to first token, stream duration, inter-event gaps, input/output token counts when reported, and observed output throughput. Percentiles require a declared minimum sample count; otherwise the result is inconclusive.

The runner will use monotonic time and bounded concurrency. It will not use BenchmarkDotNet for live network inference because BenchmarkDotNet is designed for repeatable code microbenchmarks rather than variable remote workloads.

### 6. Make policy evaluation a pure, versioned step

Execution produces observations; a separate policy engine compares them with a compatible baseline. Compatibility includes suite version, scenario identity, model identity, endpoint capability fingerprint, streaming mode, concurrency, and relevant generation parameters.

Outcomes are `Passed`, `Warning`, `Failed`, `Inconclusive`, `InfrastructureError`, or `Skipped`. Absolute requirements and relative regression thresholds are explicit. Missing or incomparable baselines never become an automatic pass.

Candidate baseline promotion is a separate reviewed operation and must not occur during the run that evaluates the candidate.

### 7. Emit canonical JSON, Markdown/HTML, and JUnit XML

Canonical JSON is the evidence source of truth. Markdown and self-contained HTML are human views derived from it. JUnit XML provides broad CI compatibility. SARIF is not a default output because LLM test regressions are not static-analysis findings; CI annotations may be added through platform adapters without misrepresenting the result type.

Reports show status, deltas, thresholds, sample counts, environment and model fingerprints, omitted capabilities, and links to evidence. They never include secret values and apply configured prompt/output redaction before persistence.

### 8. Use standard observability and secret boundaries

Microsoft.Extensions.Logging provides structured local logs. OpenTelemetry provides optional traces, metrics, and logs through an OTLP exporter; telemetry failure never changes test results. Correlation IDs link events to the artifact manifest.

Profiles contain only secret references. The host resolves them from environment variables or an approved secret provider. Raw authorization headers and API keys are redacted at source. Debug HTTP-body logging is disabled by default.

### 9. Keep the first release closed to arbitrary runtime extensions

Provider and renderer interfaces are internal extension points, but the first release will not load arbitrary assemblies or scripts. This reduces supply-chain and code-execution risk. New adapters are compiled, reviewed, and version-pinned.

## Risks / Trade-offs

- [OpenAI-compatible servers return non-standard fields or SSE events] -> Preserve raw evidence under redaction, isolate typed parsing behind adapters, and add contract fixtures before expanding fallback behavior.
- [Client retries hide real latency and failure rates] -> Disable retries for measured calls and record every attempt for non-measured discovery operations.
- [Remote workload variance creates false regressions] -> Require warm-up, minimum samples, bounded concurrency, compatible baselines, and an inconclusive state.
- [Quality checks become nondeterministic] -> Start with deterministic assertions and require separately calibrated judge policies for probabilistic evaluation.
- [Reports leak prompts, model output, endpoints, or credentials] -> Apply deny-by-default persistence, field classification, source redaction, and secret scanning of fixtures and generated artifacts.
- [Windows-only assumptions block CI portability] -> Keep domain and orchestration projects platform-neutral and isolate Windows integrations behind host adapters.
- [.NET or dependency drift changes results] -> Pin SDK and package versions, record runtime metadata, use lock files, and review automated dependency updates.
- [Flat-file artifact volume grows] -> Define retention and external archival policies; add an index service only after measured need.
- [MTP ecosystem integrations lag] -> Preserve JUnit output and allow a documented VSTest-compatible test-project path during transition.

## Migration Plan

1. Approve this OpenSpec change and resolve the open questions that affect contracts.
2. Bootstrap the pinned .NET 10 solution, schemas, domain models, and unit tests without remote inference.
3. Implement fixture-backed protocol and streaming contract tests before any live endpoint adapter.
4. Add one non-production endpoint profile and run in observation-only mode.
5. Establish an approved baseline bundle and validate report redaction.
6. Enable CI in warning-only mode, then promote selected deterministic policies to blocking gates.
7. Add performance and context scenarios only after workload ownership and resource limits are approved.

Rollback consists of disabling blocking policy evaluation, restoring the previous baseline reference and package lock, and retaining the failed run bundle for diagnosis. No destructive data migration is required because run bundles are immutable.

## Open Questions

- Which CI targets are required first: GitHub Actions, Azure DevOps, or both?
- Who may approve or revoke a baseline, and is cryptographic signing required in addition to SHA-256 manifests?
- What prompt/output data classifications and retention periods apply?
- Which deterministic quality fixtures can be committed, and which must remain external?
- What minimum sample counts and default regression thresholds are acceptable for TTFT, throughput, and error rate?
- Is optional LLM-as-judge evaluation needed, and which calibration dataset and human-review process governs it?
- Which endpoint capability extensions, such as provider-specific reasoning content, must be first-class rather than retained only as raw evidence?
