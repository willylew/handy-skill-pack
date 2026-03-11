---
name: temporal-retry-policies
description: Design Temporal activity timeouts and retry policies for outbound HTTP/service calls (Python SDK).
---

# Temporal Failure Handling & Retry Policies (Python)

## Triggers

Use this skill when:
- Adding or modifying Activities that call external services (HTTP, queues, third parties)
- Observing frequent Activity retries, timeouts, or intermittent failures
- Needing to respect server-provided retry hints (e.g., `Retry-After`)

## Goal

Make retries predictable: transient failures retry with bounded backoff; permanent failures stop quickly; the Workflow stays long-lived.

## Checklist

1. Prefer Activity timeouts over Workflow timeouts
   - Configure Activity timeouts (`schedule_to_close`, `start_to_close`, `schedule_to_start`) based on the external dependency.
   - Avoid using Workflow execution timeouts as a substitute for Activity SLAs.

2. Define a bounded retry policy
   - Set `maximum_attempts` (avoid infinite retries).
   - Set backoff parameters (`initial_interval`, `maximum_interval`, `backoff_coefficient`) to match the dependency.

3. Classify failures
   - Separate retryable vs non-retryable failures (e.g., HTTP 429/503 vs 400/404).
   - For non-retryable failures, raise a non-retryable application error so the Activity does not retry.

4. Respect `Retry-After` when present (HTTP)
   - If the server provides `Retry-After`, compute the next delay and propagate it to Temporal via the exception so retries follow the server hint.

5. Test retry behavior
   - Simulate transient failures and confirm retry count and backoff match expectations.
   - Simulate non-retryable failures and confirm the Activity stops retrying immediately.

## Commands / tests (examples)

- Unit test error classification logic (status code mapping, `Retry-After` parsing).
- In integration tests, run the Activity in a Temporal test environment and assert retry behavior.

## Common footguns

- Infinite retries that hide permanent failures
- Misclassifying HTTP status codes
- Ignoring `Retry-After` and hammering a rate-limited dependency
- Using Workflow-level timeouts instead of Activity timeouts

## Acceptance criteria

- Retryable failures back off and stop after a bounded number of attempts
- Non-retryable failures do not retry
- `Retry-After` is respected when provided

## Sources

- https://docs.temporal.io/develop/python/failure-detection
- https://docs.temporal.io/ai-cookbook
- https://docs.temporal.io/ai-cookbook/http-retry-enhancement-python
