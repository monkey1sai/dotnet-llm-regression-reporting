# Repository Agent Guide

## Purpose and State

This repository designs a Windows-first C#/.NET regression and reporting system for OpenAI-compatible LLM APIs. The current repository is planning-only: do not add runtime code unless the user explicitly requests implementation.

## Source of Truth

1. Executable tests and runtime code, once they exist
2. Accepted specifications under `openspec/specs/`
3. The active change under `openspec/changes/design-enterprise-dotnet-regression-reporting/`
4. `README.md` and `docs/`

Before changing scope, read the proposal, design, affected capability specs, and tasks. Use OpenSpec commands for proposal, apply, validation, and archive workflows. Do not archive or mark tasks complete without implementation evidence.

## Architecture Boundaries

- Target .NET 10 LTS for implementation; do not silently retarget to the locally installed .NET 9 SDK.
- Keep Domain and policy code independent from HTTP, OpenAI SDK, console, filesystem, telemetry, and report-rendering packages.
- Put the official OpenAI .NET SDK behind a provider-neutral internal contract.
- Limit protocol-level fallback code to tested OpenAI-compatible deviations.
- Use a coordinated runner for remote LLM scenarios; use xUnit for product and contract tests.
- Treat canonical JSON as report evidence; derive Markdown, HTML, and JUnit from it.

## Safety

- Never commit API keys, authorization headers, real secret values, private endpoint addresses, production prompts, or unredacted model output.
- Configuration files contain secret references only. Examples use reserved hosts and placeholder model names.
- Measured inference requests have automatic retries disabled.
- Regression runs may call approved inference and safe discovery routes only; they never deploy models or change server, MCP, browser, or credential configuration.
- Generated reports, test results, binaries, caches, and local endpoint profiles remain untracked.

## Validation

For design-only changes, run:

```powershell
openspec status --change design-enterprise-dotnet-regression-reporting
openspec validate design-enterprise-dotnet-regression-reporting --strict
git diff --check
```

For future implementation, validate in order: restore-lock verification, build, analyzers/format, affected unit tests, fixture-backed contract tests, then explicitly authorized live integration tests. A skipped live check is a known gap, not a pass.
