# Technology Evidence

Verified on 2026-07-22 for the design change `design-enterprise-dotnet-regression-reporting`. These sources guide architecture; package versions must still be pinned and revalidated during implementation.

## Platform and SDK

- [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy): .NET 10 is the active LTS release, supported through November 2028. The design therefore targets .NET 10 rather than the locally installed .NET 9 STS SDK.
- [Official OpenAI .NET SDK](https://github.com/openai/openai-dotnet): provides typed Chat and Responses clients, streaming, custom endpoint configuration for self-hosted OpenAI-compatible services, and protocol methods for narrower raw access.

## Test Execution

- [Microsoft Testing Platform overview](https://learn.microsoft.com/en-us/dotnet/core/testing/test-platforms-overview): distinguishes test framework from test platform and documents MTP's executable-first, deterministic model as well as current ecosystem trade-offs.
- [xUnit.net v3 with Microsoft Testing Platform](https://xunit.net/docs/getting-started/v3/microsoft-testing-platform): documents native MTP support and the compatibility path for environments that still rely on VSTest integrations.

The product's remote LLM scenarios remain outside unit-test scheduling. xUnit validates the engine and adapters; the coordinated CLI emits JUnit XML for CI ingestion.

## Observability and Reporting

- [OpenTelemetry .NET](https://opentelemetry.io/docs/languages/dotnet/): traces, metrics, and logs are stable signals. Export remains optional and non-authoritative.
- [GitHub SARIF guidance](https://docs.github.com/en/code-security/concepts/code-scanning/sarif-files): SARIF models static-analysis findings. The design deliberately uses canonical JSON and JUnit XML for LLM regressions instead of misclassifying them as code-scanning alerts.

## Selection Principles

- Prefer official or actively maintained libraries over custom infrastructure.
- Keep provider and renderer dependencies behind narrow internal contracts.
- Pin SDK and package versions with lock files and record them in every run bundle.
- Disable client retries for measured inference requests.
- Add a dependency only when it replaces meaningful custom code or satisfies a verified interoperability requirement.
