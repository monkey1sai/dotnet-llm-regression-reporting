## ADDED Requirements

### Requirement: Versioned regression suite
The system SHALL execute scenarios from a versioned suite manifest that identifies fixtures, required capabilities, generation parameters, assertions, repetitions, and resource limits.

#### Scenario: Suite is reproducibly loaded
- **WHEN** a valid suite and referenced fixtures are selected
- **THEN** the run manifest records their normalized content hashes and schema versions

#### Scenario: Fixture hash does not match
- **WHEN** a referenced fixture differs from its declared digest
- **THEN** the system stops that suite before inference and records a configuration failure

### Requirement: Scenario isolation
The system SHALL isolate scenario state, cancellation, timeout, evidence, and output so that one scenario failure does not corrupt another scenario result.

#### Scenario: One scenario fails parsing
- **WHEN** a response cannot be parsed for one scenario
- **THEN** that scenario receives a classified result and remaining independent scenarios may continue

### Requirement: Deterministic execution controls
The system SHALL record and honor a run seed, scenario order, concurrency limit, warm-up count, measured repetition count, and output-token limit.

#### Scenario: Repeated run uses the same schedule
- **WHEN** an operator reruns the same suite with the same seed and configuration
- **THEN** the system produces the same planned scenario order and concurrency schedule

### Requirement: Result status taxonomy
Every scenario SHALL end in exactly one status from `Passed`, `Warning`, `Failed`, `Inconclusive`, `InfrastructureError`, or `Skipped`, with a machine-readable reason code.

#### Scenario: Endpoint is unreachable
- **WHEN** transport cannot establish a connection within the configured policy
- **THEN** affected scenarios are classified as infrastructure errors rather than model-quality failures

#### Scenario: Required evidence is insufficient
- **WHEN** a scenario completes but lacks the minimum evidence required by policy
- **THEN** it is classified as inconclusive rather than passed

### Requirement: Read-only endpoint operation
Regression execution MUST NOT modify server configuration, deploy models, rotate credentials, register MCP servers, or perform administrative endpoint operations.

#### Scenario: Suite requests an administrative action
- **WHEN** a suite includes an operation outside the approved inference and discovery allowlist
- **THEN** the system rejects the operation before sending a request

### Requirement: Run cancellation
The system SHALL support operator and CI cancellation and SHALL finalize available evidence without starting new scenarios after cancellation.

#### Scenario: Run is cancelled during streaming
- **WHEN** cancellation is requested while a stream is active
- **THEN** the system closes the request, marks unfinished work with a cancellation reason, and writes a partial run manifest
