---
name: temporal-versioning-determinism
description: Keep Temporal Python Workflows deterministic across deployments using patching/worker versioning and replay tests.
---

# Temporal Workflow Versioning & Determinism

## Triggers

Use this skill when:
- Changing Workflow code (control flow, branching, activity calls)
- Upgrading Temporal SDK versions
- Rolling out new Workflow behavior while old executions are still running

## Goal

Ship Workflow changes without nondeterminism (replay) failures and without breaking in-flight executions.

## Checklist

1. Classify the change
   - If it changes Workflow control flow, activity sequencing, timers, or Signal/Query handling, treat it as a versioning change.

2. Keep side effects out of Workflows
   - Ensure all nondeterministic work (network, DB writes, LLM calls, random UUIDs/time, etc.) runs in Activities.
   - Workflows should orchestrate deterministic decisions only.

3. Pick a rollout strategy (major vs localized changes)
   - Localized change: use Temporal patching helpers (e.g., `patched()` / `deprecate_patch()`) to run new and old code paths safely.
   - Major change: use Worker Versioning / separate worker builds so old executions can complete on old code while new executions start on new code.
   - Note: Temporal's current docs state that support for the experimental pre-2025 Worker Versioning method will be removed from Temporal Server in March 2026. Use the latest Worker Versioning docs; consult the legacy docs only when maintaining older deployments.

4. Capture representative Event Histories for replay
   - Fetch event histories for real, running executions that cover important code paths.
   - Store these histories in the repo (or a dedicated artifacts store) so CI can replay them against new code.

5. Add replay tests to CI
   - Replay stored histories against the updated Workflow code using the SDK replayer APIs.
   - Fail CI on any replay errors (nondeterminism must be fixed before deploy).

6. Deploy safely
   - Roll out new workers gradually.
   - Keep old workers available until you are confident old executions have migrated past patched code paths (or have completed).

7. Clean up patches
   - After all executions that might take the old path are outside retention, deprecate patches and eventually remove them.

## Commands / tests (examples)

- Temporal Python SDK supports replaying event histories via the Replayer APIs. Add a CI step that replays saved histories and fails on any exception.
- Use the Temporal testing suite's time-skipping environment for fast Workflow tests.

## Common footguns

- "Small" Workflow refactors that accidentally change command ordering
- Any nondeterministic operation inside Workflow code (HTTP, DB, random, system time)
- Forgetting to deprecate and later remove patches after cutover

## Acceptance criteria

- All replay tests pass for stored, representative histories
- Existing executions continue without nondeterminism errors
- New executions run the intended new behavior

## Sources

- https://docs.temporal.io/develop/python/versioning
- https://docs.temporal.io/worker-versioning
- https://docs.temporal.io/develop/python/testing-suite
- https://docs.temporal.io/encyclopedia/event-history/event-history-python
- https://docs.temporal.io/develop/safe-deployments
