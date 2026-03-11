---
name: kafka-reliability-idempotency
description: Build reliable classic Kafka consumers/producers with explicit commit strategy, idempotent processing, and DLQ handling.
---

# Kafka Reliability & Idempotent Processing

This skill targets classic Kafka consumer groups that track progress with committed offsets
(`KafkaConsumer`, `AIOKafkaConsumer`, similar clients).

It does **not** cover Kafka share consumers (`KafkaShareConsumer`), which use per-record
acknowledge/release/reject/renew semantics instead of plain offset commits.

## Triggers

Use this skill when:
- Implementing a classic Kafka consumer or producer
- Handling message retries, duplicates, or delivery guarantees
- Designing a dead-letter queue (DLQ) or poison-message strategy

## Goal

Prevent lost progress and uncontrolled duplication by making commit timing, retries, idempotency,
and failure isolation explicit.

## Checklist

1. Choose the commit strategy intentionally
   - Auto-commit is acceptable for simple, one-record-at-a-time consumers.
   - If processing includes batching or non-Kafka side effects (DB writes, API calls, downstream publishes), prefer manual commits so the committed offset does not move ahead of durable processing.

2. When using manual commits, commit only after successful processing
   - Commit after the side effects are done.
   - Commit the next offset after the last successfully processed record (for example, `last_processed.offset + 1`).
   - Handle rebalance/commit failures explicitly; do not assume a manual commit will always succeed.

3. Make side effects idempotent
   - Kafka's default delivery model is at-least-once, so duplicate deliveries are normal.
   - Use unique keys, upserts, dedup tables, or other application-level idempotency so retries do not create duplicate effects.

4. Keep long processing compatible with consumer-group liveness
   - In classic Kafka consumers, long delays between `poll()` calls can trigger `max.poll.interval.ms` and cause a rebalance.
   - In aiokafka, heartbeats and rebalances happen in the background, but long CPU-bound work can still starve the event loop.
   - Use smaller batches, worker tasks, partition pause/resume, and rebalance listeners where needed.

5. For Kafka-to-Kafka stronger guarantees, keep producer idempotence enabled
   - In Kafka 4.2, `enable.idempotence=true` is the default unless conflicting producer configs disable it.
   - Idempotence requires `acks=all`, `retries>0`, and `max.in.flight.requests.per.connection<=5`.
   - In aiokafka, enable it explicitly with `enable_idempotence=True`.

6. Use transactions only for atomic consume-process-produce on Kafka topics
   - `transactional.id` belongs to the producer only and should be unique per producer instance; reusing it concurrently will fence the previous producer.
   - For exactly-once Kafka-to-Kafka flows with classic consumers, use `enable.auto.commit=false`, send the consumed offsets as part of the producer transaction (`sendOffsetsToTransaction`, `send_offsets_to_transaction`, or the client equivalent), and have downstream readers use `read_committed`.
   - Without transactional offset commit, a crash after producing but before the offset update will replay input and can duplicate output.
   - Transactions can atomically publish output records and committed offsets in Kafka; they do not by themselves give exactly-once behavior for external databases or APIs.

7. Implement retries and an explicit poison-message path
   - Retry transient failures with bounded attempts/backoff.
   - For permanently bad messages, prefer a DLQ or other quarantine path carrying the original message plus error metadata.
   - Treat DLQ as an application pattern, not as a built-in Kafka primitive.

8. Monitor lag and failure isolation
   - Alert on consumer lag growth, retry spikes, and messages accumulating in DLQ/quarantine topics.

## Commands / tests (examples)

- Integration test duplication: send the same message twice and verify idempotent behavior.
- Integration test failure isolation: force a poison message and confirm it lands in DLQ after retries.
- Integration test rebalance safety: trigger a rebalance during in-flight processing and confirm offsets are not advanced past successful work.
- If using transactions, integration test consume-process-produce with `read_committed` consumers.

## Common footguns

- Assuming auto-commit is always wrong or always safe
- Committing offsets before side effects complete
- Forgetting that committed offsets store the next record to read
- Reusing the same `transactional.id` across concurrent producer instances
- Treating Kafka transactions as enough to make external DB/API side effects exactly once
- Retrying forever and silently hiding permanent failures
- Applying offset-commit guidance to share consumers

## Acceptance criteria

- The commit strategy is documented and matches the processing model
- Offsets are only advanced after successful processing in manual-commit flows
- Duplicate deliveries do not produce duplicate side effects
- Long-running processing does not cause silent rebalance/data-loss bugs
- If transactions are used for Kafka-to-Kafka consume-process-produce, producers use a unique `transactional.id`, consumed offsets are committed in the transaction, and readers use `read_committed`
- Poison messages are isolated via DLQ/quarantine after bounded retries
- Lag, retries, and DLQ/quarantine volume are monitored

## Sources

- https://kafka.apache.org/42/design/design/#message-delivery-semantics
- https://kafka.apache.org/42/design/design/#using-transactions
- https://kafka.apache.org/42/design/design/#the-share-consumer
- https://kafka.apache.org/42/generated/producer_config.html
- https://kafka.apache.org/42/generated/consumer_config.html
- https://kafka.apache.org/41/javadoc/org/apache/kafka/clients/producer/KafkaProducer.html#sendOffsetsToTransaction(java.util.Map,org.apache.kafka.clients.consumer.ConsumerGroupMetadata)
- https://aiokafka.readthedocs.io/en/stable/consumer.html#manual-vs-automatic-committing
- https://aiokafka.readthedocs.io/en/stable/consumer.html#reading-transactional-messages
- https://aiokafka.readthedocs.io/en/stable/producer.html#idempotent-produce
- https://aiokafka.readthedocs.io/en/stable/producer.html#transactional-producer
- https://aiokafka.readthedocs.io/en/stable/kafka-python_difference.html#rebalances-are-happening-in-the-background
