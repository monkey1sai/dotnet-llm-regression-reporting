## ADDED Requirements

### Requirement: Canonical result document
The system SHALL write a schema-versioned canonical JSON result containing run identity, timestamps, configuration digests, environment fingerprint, scenario observations, policy decisions, and reason codes.

#### Scenario: Run completes normally
- **WHEN** all scheduled scenarios reach terminal states
- **THEN** the canonical result validates against its committed schema and becomes the source for derived reports

#### Scenario: Run is interrupted
- **WHEN** execution terminates after evidence has been collected
- **THEN** the system writes a valid partial result that identifies incomplete work and termination reason

### Requirement: Human-readable summary
The system SHALL derive Markdown and self-contained HTML summaries from canonical results and SHALL show aggregate outcome, regressions, warnings, skipped capabilities, evidence sufficiency, and baseline identity.

#### Scenario: Reviewer opens an HTML report offline
- **WHEN** a completed report bundle is copied to a disconnected review machine
- **THEN** the HTML summary renders without external scripts, fonts, or network requests

### Requirement: CI-compatible result output
The system SHALL provide JUnit XML that maps scenario outcomes and diagnostics without changing the canonical status taxonomy.

#### Scenario: CI imports JUnit XML
- **WHEN** a CI system reads the generated JUnit file
- **THEN** failed, skipped, and infrastructure-error scenarios remain distinguishable through documented mappings and properties

### Requirement: Secret and sensitive-data redaction
The system MUST redact credentials and configured sensitive prompt, output, header, query, endpoint, and environment fields before any log, telemetry export, report, or persisted raw evidence is written.

#### Scenario: Credential appears in an error body
- **WHEN** a server or proxy echoes credential material in an error response
- **THEN** persisted diagnostics contain a redaction marker and never the credential value

#### Scenario: Raw payload retention is disabled
- **WHEN** policy forbids storing prompt or output bodies
- **THEN** reports retain only allowed hashes, lengths, classifications, and derived assertions

### Requirement: Artifact integrity and provenance
Every report bundle SHALL include SHA-256 digests, generator version, source revision, suite and policy versions, endpoint/model fingerprints, runtime version, operating system, and timestamps.

#### Scenario: Report is verified after transfer
- **WHEN** a reviewer validates a copied bundle
- **THEN** the verifier can detect any changed artifact and identify the generating run

### Requirement: Retention manifest
The system SHALL identify each artifact's classification, retention category, and safe deletion eligibility without embedding storage credentials.

#### Scenario: Retention process evaluates a bundle
- **WHEN** an external retention job reads the bundle manifest
- **THEN** it can determine which artifacts are eligible for deletion and which require preservation under policy

### Requirement: Telemetry is non-authoritative
OpenTelemetry export failures MUST NOT alter scenario or policy outcomes, and canonical local evidence SHALL remain sufficient to review the run.

#### Scenario: OTLP collector is unavailable
- **WHEN** telemetry export fails during a regression run
- **THEN** the run records an observability warning while continuing to produce canonical local artifacts
