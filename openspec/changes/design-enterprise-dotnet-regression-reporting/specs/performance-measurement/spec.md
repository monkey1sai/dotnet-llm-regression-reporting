## ADDED Requirements

### Requirement: Monotonic latency measurement
The system SHALL use a monotonic clock to measure end-to-end latency, time to first byte, time to first token, stream duration, and inter-event timing where applicable.

#### Scenario: Streaming response completes
- **WHEN** a valid streaming response emits text events and terminates normally
- **THEN** the system records first-byte, first-token, final-event, and total-duration measurements with their clock resolution

### Requirement: Warm-up separation
The system SHALL label warm-up requests separately and MUST exclude them from measured percentile and throughput summaries.

#### Scenario: Warm-up succeeds before measured samples
- **WHEN** a workload defines warm-up requests
- **THEN** warm-up evidence is retained but only measured samples contribute to policy metrics

### Requirement: Bounded workload schedule
The system SHALL enforce declared concurrency, request-rate, duration, timeout, input-size, and output-token limits.

#### Scenario: Concurrency limit is reached
- **WHEN** the number of active requests equals the configured limit
- **THEN** the system waits or rejects additional scheduled work according to the suite policy without exceeding the limit

### Requirement: Retry-free measured samples
Each measured sample MUST correspond to one transport attempt unless the suite explicitly defines attempts as separate samples.

#### Scenario: Server returns a retryable status
- **WHEN** a measured request receives a retryable HTTP status
- **THEN** the system records the status for that sample and does not automatically retry it

### Requirement: Statistically qualified summaries
The system SHALL report sample count, success count, error count, minimum, maximum, median, and configured percentiles and SHALL mark percentile comparisons inconclusive below the policy minimum sample count.

#### Scenario: Sample count is too small
- **WHEN** only three valid samples exist and policy requires twenty for percentile gating
- **THEN** percentile values may be shown as observations but cannot produce a passing gate

### Requirement: Token and throughput provenance
The system SHALL distinguish server-reported token counts from locally estimated counts and SHALL identify the source used for output-throughput calculations.

#### Scenario: Server returns usage metadata
- **WHEN** a completed response contains supported usage fields
- **THEN** the system stores the reported values and labels them as server-reported

#### Scenario: Usage metadata is absent
- **WHEN** a response completes without token usage
- **THEN** the system reports token-derived throughput as unavailable or estimated according to explicit policy

### Requirement: Context-boundary safety
Context tests SHALL use declared maximum input growth, output limits, deadlines, and stop conditions and MUST NOT probe indefinitely.

#### Scenario: Context rejection occurs
- **WHEN** the endpoint rejects an input at a tested context size
- **THEN** the system records the tested size, classified server response, and last successful bound without automatically increasing the input
