---
name: temporal-testing-ci
description: Test Temporal Python Workflows with time skipping and replay in CI to catch nondeterminism and logic errors early.
---

# Temporal Testing & CI Discipline (Python)

## Triggers

Use this skill when:
- Adding or refactoring Workflows/Activities
- Preparing a release that changes Workflow behavior
- You need confidence that changes are replay-safe

## Goal

Catch nondeterminism and correctness bugs before deploy using time-skipping tests and replay tests in CI.

## Checklist

1. Use the Temporal testing suite
   - Run Workflows against the Temporal test environment (not just unit tests of pure functions).

2. Enable time skipping
   - Use the testing suite's time-skipping environment so timers and sleeps do not make tests slow.

3. Write integration-style tests for Workflows
   - Execute Workflows via the client and assert results.
   - Mock external calls by moving them into Activities and mocking Activities in tests.

4. Add replay tests
   - Fetch representative Event Histories from real executions.
   - Store them so CI can replay them against new code.
   - Run replay in CI and fail on any error.

5. Keep tests hermetic
   - Ensure no real network calls happen inside Workflow code (replay will fail, and tests become flaky).
   - Reset/cleanup test environment between tests to avoid leaked state.

## Commands / tests (examples)

- Add a CI job that runs:
  - standard tests (e.g., `pytest`)
  - replay tests using the SDK replayer APIs against stored event histories

## Common footguns

- Only writing unit tests and skipping end-to-end Workflow tests
- Not using time skipping, leading to slow tests that get skipped
- Forgetting to include replay tests for backward compatibility

## Acceptance criteria

- Workflow tests run quickly and reliably in CI
- Replay tests pass for all stored histories
- CI fails fast on nondeterministic changes

## Sources

- https://docs.temporal.io/develop/python/testing-suite
- https://docs.temporal.io/develop/safe-deployments
- https://docs.temporal.io/encyclopedia/event-history/event-history-python
