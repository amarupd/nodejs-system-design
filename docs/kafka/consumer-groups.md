# Kafka Consumer Groups

![Kafka](../../assets/kafka.png)

---

# Overview

Consumer Groups are one of Kafka's most powerful scalability features.

They allow multiple consumers to work together to process events from a topic while ensuring that each event is processed by only one consumer within the group.

Consumer Groups enable:

* Horizontal scaling
* Fault tolerance
* High throughput
* Load balancing
* Reliable event processing

Without Consumer Groups, Kafka would not be able to support large-scale event-driven architectures.

---

# What Is A Consumer Group?

A Consumer Group is a collection of consumers working together.

Architecture:

```text id="m8q2r5"
Kafka Topic
     ↓
Consumer Group
     ↓
Consumers
```

All consumers share the workload.

---

# Why Consumer Groups Exist

Single Consumer:

```text id="u3m7q8"
Topic
 ↓
Consumer
```

Problems:

* Limited throughput
* Single point of failure
* Poor scalability

---

Consumer Group:

```text id="k6q1m4"
Topic
 ↓
Consumer A

Consumer B

Consumer C
```

Benefits:

* Parallel processing
* Higher throughput
* Fault tolerance

---

# Core Rule

Within a Consumer Group:

```text id="r7m3q1"
One Partition
↓
One Consumer
```

at a time.

This is the foundation of Kafka scaling.

---

# Example

Topic:

```text id="t4q8m2"
orders
```

Partitions:

```text id="p9m1q6"
P0

P1

P2
```

Consumers:

```text id="v5m7q3"
A

B

C
```

Assignments:

```text id="n2q8m5"
A → P0

B → P1

C → P2
```

Each consumer processes a unique partition.

---

# More Consumers Than Partitions

Partitions:

```text id="u8m4q1"
3
```

Consumers:

```text id="k1q7m9"
5
```

Result:

```text id="r4m2q8"
2 Consumers Idle
```

Because:

```text id="w7q3m1"
Maximum Parallelism
=
Partition Count
```

---

# More Partitions Than Consumers

Partitions:

```text id="t2m8q4"
10
```

Consumers:

```text id="p6q1m7"
3
```

Assignments:

```text id="x9m3q2"
Consumer A → P0,P1,P2

Consumer B → P3,P4,P5

Consumer C → P6,P7,P8,P9
```

Consumers handle multiple partitions.

---

# Consumer Group Architecture

```text id="m5q8r1"
Topic
 ↓
Partitions
 ↓
Consumer Group
```

Benefits:

* Automatic workload distribution
* Fault recovery

---

# Independent Consumer Groups

Multiple groups can consume the same topic.

Example:

Topic:

```text id="u7m2q5"
orders
```

Groups:

```text id="k3q9m1"
Notification Group

Analytics Group

Reporting Group
```

Each group receives all events independently.

---

# Architecture

```text id="r8m4q2"
Kafka Topic
      ↓
Notification Group

Analytics Group

Reporting Group
```

Benefits:

* Service independence
* Loose coupling

---

# Consumer Offsets

Kafka tracks:

```text id="v2q7m8"
Offset
```

Example:

```text id="n6m1q4"
Event #105
```

Consumer remembers:

```text id="t9m3q7"
Last Processed Offset
```

Benefits:

* Recovery
* Replay

---

# Offset Commit

After successful processing:

```text id="p4q8m1"
Process Event
 ↓
Commit Offset
```

Kafka records progress.

---

# Auto Commit

Kafka automatically commits offsets.

Benefits:

* Simplicity

Risks:

```text id="u1m7q5"
Message Loss
```

during failures.

---

# Manual Commit

Preferred in production.

Flow:

```text id="r5q2m8"
Process Event
 ↓
Success
 ↓
Commit Offset
```

Benefits:

* Better reliability

---

# Consumer Failure

Example:

```text id="w8m4q1"
Consumer B Crashes
```

Kafka detects failure.

---

Flow:

```text id="k2q7m3"
Rebalance
 ↓
Partition Reassignment
```

Other consumers take over.

---

# Rebalancing

Occurs when:

```text id="p7m1q6"
Consumer Added

Consumer Removed

Consumer Failed
```

Kafka redistributes partitions.

---

Benefits:

* Automatic recovery

---

Drawbacks:

```text id="u4q8m2"
Temporary Pause
```

during reassignment.

---

# Consumer Lag

One of Kafka's most important metrics.

Definition:

```text id="r9m3q5"
Latest Offset
-
Consumer Offset
```

Represents:

```text id="x3q7m1"
Unprocessed Events
```

---

Example:

Latest:

```text id="t8m2q4"
1000
```

Consumer:

```text id="v6q1m8"
900
```

Lag:

```text id="n5m7q2"
100
```

---

High Lag Indicates

* Slow consumers
* Traffic spikes
* Resource bottlenecks

---

# Scaling Consumer Groups

Option 1:

```text id="p2q8m5"
Add Consumers
```

---

Option 2:

```text id="u7m3q1"
Add Partitions
```

---

Often both are required.

---

# Ecommerce Example

Topic:

```text id="k8q4m2"
orders
```

Group:

```text id="r1m7q9"
Notification Workers
```

Processes:

```text id="t5q2m6"
ORDER_CREATED
```

events.

---

# Payment Example

Topic:

```text id="v9m3q4"
payments
```

Group:

```text id="n2q8m1"
Ledger Workers
```

Processes:

```text id="u6m4q7"
PAYMENT_SUCCESS
```

events.

---

# Fantasy Sports Example

Topic:

```text id="k4q7m2"
match-events
```

Group:

```text id="r8m1q5"
Scoring Workers
```

Processes:

```text id="w3q9m4"
Runs

Wickets

Boundaries
```

---

# Live Score Example

Architecture:

```text id="p5m2q8"
Kafka
 ↓
Score Processing Group
 ↓
Redis
 ↓
WebSockets
```

Benefits:

* Massive scalability

---

# Monitoring Consumer Groups

Track:

## Consumer Lag

Processing health.

---

## Rebalance Frequency

Stability indicator.

---

## Consumer Availability

Reliability.

---

## Throughput

Performance.

---

## Offset Commit Rate

Consumer progress.

---

# Security Considerations

## Authentication

Restrict access.

---

## Authorization

Topic permissions.

---

## Encryption

Secure event traffic.

---

## Audit Logs

Track consumption activity.

---

# Common Mistakes

## Too Few Partitions

Limits scaling.

---

## Auto Commit Everywhere

Can cause data loss.

---

## Ignoring Lag

Leads to delayed processing.

---

## Excessive Rebalancing

Reduces throughput.

---

## Weak Monitoring

Hidden problems accumulate.

---

# Common Interview Questions

### What is a Consumer Group?

A collection of consumers working together to process a topic.

---

### Can two consumers in the same group read the same partition?

No.

Only one consumer per partition.

---

### What is consumer lag?

The difference between produced and consumed offsets.

---

### Why use manual offset commits?

To improve reliability and prevent data loss.

---

### What happens when a consumer crashes?

Kafka rebalances partitions among remaining consumers.

---

### How do Consumer Groups enable scaling?

By distributing partitions across multiple consumers.

---

# Production Lessons

* Consumer Groups are the primary Kafka scaling mechanism.
* Partition count determines maximum parallelism.
* Consumer lag must be monitored continuously.
* Manual commits improve reliability.
* Rebalancing affects performance.
* Multiple groups allow service independence.
* Proper Consumer Group design is essential for large-scale event processing.

---

# Key Takeaways

* Consumer Groups enable Kafka scalability and fault tolerance.
* Each partition is processed by one consumer within a group.
* Offsets track consumption progress.
* Consumer lag is a critical operational metric.
* Rebalancing provides automatic recovery.
* Multiple Consumer Groups can independently consume the same topic.
* Understanding Consumer Groups is essential for modern backend engineering and system design interviews.

---

# Related Documents

* docs/kafka/partitions.md
* docs/kafka/kafka-patterns.md
* docs/kafka/exactly-once.md
* docs/architecture/event-driven-architecture.md

Related Diagram:

* diagrams/kafka-architecture.mmd
