# .NET LLM Regression Reporting

Design repository for a Windows-first, enterprise-grade C#/.NET regression and reporting system targeting OpenAI-compatible LLM APIs.

## Status

This repository is **design-only**. It contains an apply-ready OpenSpec change but no runtime implementation, live endpoint configuration, API key, or inference result.

The proposed implementation targets .NET 10 LTS. The current machine has .NET 9, so implementation must begin by installing and pinning a supported .NET 10 SDK rather than silently changing the target framework.

## Design Scope

- Provider-neutral endpoint and capability profiles
- Deterministic compatibility and quality regression suites
- Streaming, latency, throughput, reliability, and bounded context measurements
- Immutable baselines and explicit policy gates
- Redacted JSON, Markdown, HTML, and JUnit evidence
- Windows local execution with portable CI boundaries
- Optional OpenTelemetry instrumentation

Out of scope are model deployment, server configuration, Stagehand, MCP registration, credential rotation, web dashboards, and production inference during design work.

## Source of Truth

The active change is [`design-enterprise-dotnet-regression-reporting`](openspec/changes/design-enterprise-dotnet-regression-reporting/):

- [`proposal.md`](openspec/changes/design-enterprise-dotnet-regression-reporting/proposal.md) — motivation and capability boundaries
- [`design.md`](openspec/changes/design-enterprise-dotnet-regression-reporting/design.md) — architecture and decisions
- [`specs/`](openspec/changes/design-enterprise-dotnet-regression-reporting/specs/) — testable requirements
- [`tasks.md`](openspec/changes/design-enterprise-dotnet-regression-reporting/tasks.md) — future implementation sequence

Supporting technology research is recorded in [`docs/technology-evidence.md`](docs/technology-evidence.md).

## Review Commands

```powershell
openspec status --change design-enterprise-dotnet-regression-reporting
openspec validate design-enterprise-dotnet-regression-reporting --strict
git diff --check
```

Implementation is intentionally deferred. After design approval, use the OpenSpec apply workflow and complete the tasks in dependency order.
