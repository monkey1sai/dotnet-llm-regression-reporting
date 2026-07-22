## ADDED Requirements

### Requirement: Versioned endpoint profile
The system SHALL load an endpoint profile from a versioned schema containing a logical endpoint identifier, base URI, model identifier, secret reference, timeouts, retry policy, and declared capabilities.

#### Scenario: Valid profile is accepted
- **WHEN** an operator supplies a profile that conforms to the supported schema version
- **THEN** the system creates an immutable normalized profile for the run

#### Scenario: Unknown profile version is rejected
- **WHEN** a profile declares an unsupported schema version
- **THEN** the system stops before network access and reports a configuration error

### Requirement: Secret references only
The system MUST reject plaintext API keys or authorization tokens in committed endpoint-profile fields and SHALL resolve credentials only through the host secret boundary.

#### Scenario: Environment secret is resolved
- **WHEN** a profile references an allowed environment variable and that variable is present
- **THEN** the system supplies the credential to the transport without persisting its value

#### Scenario: Plaintext credential is detected
- **WHEN** a profile field contains a value classified as a credential rather than a secret reference
- **THEN** the system rejects the profile and redacts the detected value from diagnostics

### Requirement: Explicit capability contract
The system SHALL represent each endpoint capability as supported, unsupported, or unknown and MUST NOT infer unrelated route support from a health response.

#### Scenario: Unsupported embedding route
- **WHEN** a scenario requires embeddings and the selected profile declares embeddings unsupported
- **THEN** the scenario is recorded as skipped with the capability reason and no embedding request is sent

#### Scenario: Unknown capability requires discovery policy
- **WHEN** a scenario requires a capability marked unknown
- **THEN** the system follows the configured safe-discovery policy or records the scenario as inconclusive

### Requirement: Measured transport behavior
The system SHALL use explicit cancellation and timeout values and MUST disable automatic retries for measured inference requests.

#### Scenario: Request exceeds deadline
- **WHEN** a measured request exceeds its configured deadline
- **THEN** the system cancels the request and records one timed-out attempt without silently retrying

### Requirement: Redacted connection diagnostics
The system SHALL expose enough transport metadata to diagnose failures while removing credentials, authorization headers, sensitive query values, and configured sensitive payload fields.

#### Scenario: Authentication failure
- **WHEN** an endpoint returns an authentication error
- **THEN** the report contains status and classified error evidence but no credential value
