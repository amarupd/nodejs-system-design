# Kafka Exactly-Once Processing

![Kafka](../../assets/kafka.png)

---

# Overview

Exactly-Once Processing (EOS) is one of the most advanced and frequently discussed topics in distributed systems and Kafka architecture.

In real-world systems, failures happen constantly:

* Network interruptions
* Consumer crashes
* Broker failures
* Timeout errors
* Service restarts

When failures occur, the same event may be delivered more than once.

For many business domains, duplicate processing is unacceptable.

Examples:

* Payments
* Banking transactions
* Wallet credits
* Inventory reservations
* Order settlements

Kafka provides mechanisms that help build exactly-once processing pipelines.

---

# Why Exactly-Once Matters

Imagine:

```text id="q4m8r2"
PAYMENT_SUCCESS
```

event arrives.

Consumer processes:

```text id="u7q1m5"
Credit Wallet
```

---

Consumer crashes before committing offset.

Kafka redelivers event.

Consumer executes again.

Result:

```text id="r2m7q9"
Double Credit
```

Business impact:

* Financial loss
* Incorrect balances
* Data inconsistency

---

# Delivery Guarantees

Distributed messaging systems generally provide three delivery models.

---

# At-Most-Once

Message delivered:

```text id="k6q3m1"
0 or 1 Time
```

---

Flow:

```text id="p8m4q7"
Commit Offset
 ↓
Process Message
```

If processing fails:

```text id="v5q2m8"
Message Lost
```

---

Benefits:

* Fast

Drawbacks:

* Data loss

---

# At-Least-Once

Most common model.

Flow:

```text id="u1m7q4"
Process
 ↓
Commit Offset
```

If failure occurs:

```text id="t9q3m6"
Message Replayed
```

---

Benefits:

* No message loss

Drawbacks:

```text id="r4m8q2"
Duplicates Possible
```

---

# Exactly-Once

Goal:

```text id="x7m2q5"
Process Once
Only Once
```

No loss.

No duplicates.

Most difficult guarantee.

---

# Why Exactly-Once Is Hard

Distributed systems involve:

```text id="m5q8r1"
Producer

Broker

Consumer

Database
```

Failures may occur at any point.

Examples:

* Consumer crash
* Network timeout
* Offset commit failure
* Database write failure

---

# Duplicate Processing Example

Flow:

```text id="k2m7q4"
Consume Event
 ↓
Update Database
 ↓
Crash
```

Offset not committed.

Kafka retries.

Database updated twice.

---

# Kafka Idempotent Producer

Kafka supports:

```text id="p7q1m8"
Idempotent Producer
```

Benefits:

* Prevent duplicate writes
* Safer retries

---

Architecture:

```text id="u4m8q3"
Producer
 ↓
Kafka
```

Duplicate producer retries are automatically handled.

---

# Producer Failure Example

Producer sends:

```text id="r8m3q6"
ORDER_CREATED
```

Network timeout occurs.

Producer retries.

Without idempotency:

```text id="t5q2m7"
Duplicate Event
```

---

With idempotency:

```text id="w1m9q4"
Single Event Stored
```

---

# Kafka Transactions

Kafka supports transactions across:

```text id="n6q3m8"
Multiple Partitions
```

and

```text id="v9m2q1"
Multiple Topics
```

---

Benefits:

* Atomic writes
* Consistent event streams

---

# Transaction Flow

```text id="p3q8m5"
Begin Transaction
 ↓
Write Events
 ↓
Commit Transaction
```

Consumers see either:

```text id="r7m1q4"
All Events
```

or

```text id="u5q9m2"
None
```

---

# Consumer Side Challenges

Even with Kafka transactions:

```text id="t2m8q6"
Database
```

is external.

Problem remains:

```text id="x4q7m1"
Database Updated
 ↓
Consumer Crash
```

---

Kafka cannot automatically roll back external systems.

---

# Idempotent Consumers

Most practical solution.

Store:

```text id="m8q2r5"
event_id
```

or

```text id="p6m4q8"
message_id
```

---

Before processing:

```text id="u3q7m1"
Already Processed?
```

If yes:

```text id="r9m2q4"
Ignore
```

---

Benefits:

* Duplicate-safe processing

---

# Example Event

```json id="k7q1m9"
{
  "event_id": "EVT_1001",
  "order_id": 501
}
```

Consumer stores:

```text id="t4m8q2"
EVT_1001
```

in processed-event table.

---

# Database Pattern

Table:

```text id="v1q7m5"
processed_events
```

Columns:

```text id="x8m3q4"
event_id

processed_at
```

---

Flow:

```text id="n2q9m1"
Check Event
 ↓
Process
 ↓
Store Event ID
```

---

# Outbox Pattern

One of the most important enterprise patterns.

Problem:

```text id="p5m7q2"
Database Updated
```

but:

```text id="u8q1m6"
Kafka Event Lost
```

---

Solution:

```text id="r4m9q3"
Outbox Table
```

Architecture:

```text id="w7m2q5"
Database
 ↓
Outbox
 ↓
Kafka
```

Benefits:

* Reliable event publication

---

# Inbox Pattern

Used on consumers.

Architecture:

```text id="k3q8m4"
Kafka
 ↓
Inbox Table
 ↓
Business Logic
```

Benefits:

* Deduplication
* Auditability

---

# Payment Example

Event:

```text id="v6m1q8"
PAYMENT_SUCCESS
```

Consumer:

```text id="t9q4m2"
Wallet Service
```

Without idempotency:

```text id="r2m7q5"
Double Credit
```

Possible.

---

With idempotency:

```text id="u5q3m9"
Safe Processing
```

Guaranteed.

---

# Ecommerce Example

Events:

```text id="p8m2q4"
ORDER_CREATED

ORDER_PAID

ORDER_SHIPPED
```

Duplicates could:

* Create duplicate shipments
* Send duplicate emails
* Reserve inventory twice

Idempotent consumers prevent this.

---

# Fantasy Sports Example

Event:

```text id="n4q7m1"
POINTS_UPDATED
```

Duplicate processing may:

```text id="x1m8q5"
Double Score
```

participants.

---

Idempotency protects leaderboard integrity.

---

# Event Sourcing Example

Architecture:

```text id="k9q3m2"
Events
 ↓
Kafka
 ↓
Consumers
```

Replay frequently occurs.

Exactly-once design becomes essential.

---

# Monitoring Exactly-Once Systems

Track:

## Duplicate Events

Reliability indicator.

---

## Transaction Failures

Producer health.

---

## Offset Commit Errors

Consumer stability.

---

## Event Replay Counts

Recovery metric.

---

## Consumer Lag

Processing performance.

---

# Security Considerations

## Audit Logging

Track event history.

---

## Access Control

Protect topics.

---

## Encryption

Secure event data.

---

## Compliance

Important for financial systems.

---

# Common Mistakes

## Assuming Kafka Alone Solves EOS

It does not.

---

## No Idempotency

Duplicate effects occur.

---

## Ignoring Outbox Pattern

Creates inconsistencies.

---

## Weak Monitoring

Problems remain hidden.

---

## No Replay Strategy

Recovery becomes difficult.

---

# Common Interview Questions

### What is Exactly-Once Processing?

Processing an event exactly one time without loss or duplication.

---

### Why is it difficult?

Distributed systems fail in many ways.

---

### What is an idempotent producer?

A producer that avoids duplicate event writes.

---

### What is an idempotent consumer?

A consumer that safely handles duplicate deliveries.

---

### What is the Outbox Pattern?

A reliable method for publishing events from databases to Kafka.

---

### Can Kafka guarantee exactly-once end-to-end?

Only when applications are designed correctly with idempotency and transactional patterns.

---

# Production Lessons

* Exactly-once is a system design challenge, not merely a Kafka feature.
* Idempotency is mandatory.
* Outbox patterns improve reliability.
* Duplicate events are normal.
* Financial systems require strong protections.
* Monitoring and auditing are essential.
* Well-designed event-driven systems embrace failure and recover safely.

---

# Key Takeaways

* Exactly-once processing is one of the hardest distributed systems problems.
* Kafka provides idempotent producers and transactions.
* Consumer idempotency remains critical.
* Outbox and Inbox patterns improve consistency.
* Payments and financial systems rely heavily on EOS strategies.
* Duplicate events should always be expected.
* Understanding exactly-once processing is a senior-level Kafka and system design skill.

---

# Related Documents

* docs/kafka/consumer-groups.md
* docs/kafka/event-streaming.md
* docs/kafka/kafka-patterns.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/kafka-architecture.mmd
