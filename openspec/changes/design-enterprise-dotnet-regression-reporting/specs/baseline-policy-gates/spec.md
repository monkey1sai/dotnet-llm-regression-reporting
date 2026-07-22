## ADDED Requirements

### Requirement: Immutable baseline reference
The system SHALL compare a candidate only against an explicitly selected, immutable baseline bundle whose manifest and artifact digests validate successfully.

#### Scenario: Baseline digest is valid
- **WHEN** an approved baseline is selected and all declared digests match
- **THEN** the system may use its observations for comparison

#### Scenario: Baseline artifact was modified
- **WHEN** a baseline artifact digest differs from its manifest
- **THEN** the system rejects the baseline and marks comparison inconclusive

### Requirement: Compatibility before comparison
The system SHALL evaluate suite, scenario, model, endpoint capability, streaming, concurrency, and generation-parameter compatibility before calculating a regression delta.

#### Scenario: Concurrency differs materially
- **WHEN** a candidate throughput run uses a different concurrency from its baseline
- **THEN** the system does not issue a passing relative-throughput decision unless policy explicitly allows normalization

### Requirement: Explicit policy thresholds
Policy files SHALL define absolute requirements, relative warning and failure thresholds, minimum evidence, and the metric direction for every blocking gate.

#### Scenario: Latency exceeds failure threshold
- **WHEN** a comparable candidate latency regression exceeds the configured failure threshold with sufficient evidence
- **THEN** the policy result is failed with the observed delta and threshold

#### Scenario: Quality improves
- **WHEN** a higher-is-better quality metric exceeds its baseline without violating an absolute requirement
- **THEN** the policy records an improvement and does not invert the metric direction

### Requirement: Missing baseline behavior
The system MUST NOT treat a missing or incomparable baseline as a passing relative gate.

#### Scenario: First execution has no baseline
- **WHEN** a relative policy is evaluated without an approved baseline
- **THEN** the result is inconclusive and identifies baseline approval as the required action

### Requirement: Baseline promotion separation
The system SHALL require a distinct, auditable approval operation to promote a candidate artifact bundle to baseline and MUST NOT auto-promote during evaluation.

#### Scenario: Candidate passes all gates
- **WHEN** a run passes every configured gate
- **THEN** it remains a candidate until an authorized promotion records approver, time, source digest, and reason

### Requirement: Stable process exit contract
The CLI SHALL map aggregate policy outcomes to documented, stable exit codes without losing detailed scenario statuses in artifacts.

#### Scenario: One blocking gate fails
- **WHEN** at least one blocking policy result is failed
- **THEN** the CLI returns the documented regression-failure exit code and preserves the complete report bundle
