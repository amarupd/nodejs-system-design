# Kafka Partitions Architecture

![Kafka](../../assets/kafka.png)

---

# Overview

Partitions are the foundation of Kafka's scalability, parallelism, and fault tolerance.

Without partitions, Kafka would be unable to handle:

* Millions of events per second
* Large consumer groups
* Horizontal scaling
* High-throughput workloads

Understanding partitions is one of the most important Kafka concepts and a common system design interview topic.

---

# What Is A Partition?

A partition is an ordered, append-only log within a Kafka topic.

Example Topic:

```text id="k7m2q4"
orders
```

With three partitions:

```text id="u4q8m1"
orders

├─ Partition 0
├─ Partition 1
└─ Partition 2
```

Each partition stores events independently.

---

# Why Partitions Exist

Without partitions:

```text id="r9m3q5"
One Topic
 ↓
One Log
```

Problems:

* Limited throughput
* No parallelism
* Scaling bottlenecks

---

With partitions:

```text id="p2q7m8"
Partition 0

Partition 1

Partition 2
```

Benefits:

* Parallel writes
* Parallel reads
* Horizontal scaling

---

# Partition Architecture

```text id="n5m1q7"
Producer
 ↓
Topic
 ↓
Partitions
 ↓
Consumers
```

Each partition acts as an independent event stream.

---

# Ordered Event Log

Partition example:

```text id="x8q4m2"
Offset 0

Offset 1

Offset 2

Offset 3
```

Events are appended sequentially.

Benefits:

* Predictable ordering
* Efficient storage

---

# Ordering Guarantee

Kafka guarantees ordering:

```text id="t3m7q1"
Within A Partition
```

Example:

```text id="w6q2m8"
ORDER_CREATED

PAYMENT_SUCCESS

ORDER_SHIPPED
```

If all events are in the same partition:

Order remains preserved.

---

# Global Ordering Myth

Many engineers misunderstand this.

Kafka does NOT guarantee:

```text id="v1m8q4"
Global Ordering
```

Across all partitions.

Example:

```text id="r4q9m2"
Partition A

Partition B
```

Events can be processed independently.

---

# Partition Keys

Partition selection often depends on a key.

Example:

```text id="k2m7q5"
order_id
```

Kafka hashes the key and selects a partition.

Benefits:

* Consistent routing
* Ordered processing

---

# Example Partition Routing

Events:

```text id="u7q1m3"
Order 101

Order 101

Order 101
```

All routed to:

```text id="p8m4q6"
Partition 2
```

Ordering preserved.

---

Another order:

```text id="x5q2m8"
Order 102
```

May go to:

```text id="t9m3q1"
Partition 0
```

Independent processing.

---

# Partition Strategy

One of the most important design decisions.

---

# Random Partitioning

No key provided.

Kafka distributes events automatically.

Benefits:

* Balanced load

Drawbacks:

* No ordering guarantees per entity

---

# Key-Based Partitioning

Use:

```text id="n6q1m7"
user_id

order_id

match_id
```

Benefits:

* Entity ordering
* Predictable processing

---

# Partition Count

Topic Example:

```text id="r3m8q4"
orders
```

Partitions:

```text id="w8q2m5"
10
```

Consumers:

```text id="k1m7q9"
10
```

Maximum parallelism:

```text id="v7q4m1"
10 Consumers
```

---

# Consumer Assignment

Consumer Group:

```text id="p4m9q2"
Consumer A

Consumer B

Consumer C
```

Partitions:

```text id="u2q8m6"
P0

P1

P2
```

Assignments:

```text id="t5m3q8"
A → P0

B → P1

C → P2
```

Benefits:

* Parallel consumption

---

# Too Few Partitions

Example:

```text id="x3q7m1"
1 Partition
```

Problems:

* Throughput bottleneck
* Limited scaling

---

# Too Many Partitions

Example:

```text id="n9m2q5"
10,000 Partitions
```

Problems:

* Increased metadata
* Broker overhead
* Operational complexity

---

# Partition Rebalancing

Occurs when:

```text id="r7q1m4"
Consumer Added

Consumer Removed
```

Kafka redistributes partitions.

Benefits:

* Automatic scaling

---

Drawbacks:

```text id="w4m8q2"
Temporary Pause
```

during rebalance.

---

# Replication And Partitions

Partition Example:

```text id="k8q3m7"
Partition 0
```

Replicated to:

```text id="u1m7q5"
Broker A

Broker B

Broker C
```

Benefits:

* High availability
* Fault tolerance

---

# Leader Partition

Each partition has:

```text id="p6q2m8"
Leader
```

Producer writes here.

Consumers read here.

---

# Follower Partitions

Replicate data.

Architecture:

```text id="x7m4q1"
Leader
 ↓
Followers
```

Benefits:

* Recovery
* Availability

---

# Ecommerce Example

Partition Key:

```text id="n2q8m5"
order_id
```

Benefits:

* Order lifecycle remains ordered

Events:

```text id="t8m3q4"
ORDER_CREATED

ORDER_PAID

ORDER_SHIPPED
```

Remain sequential.

---

# Payment Example

Partition Key:

```text id="r1m7q8"
payment_id
```

Benefits:

* Transaction consistency

---

# Fantasy Sports Example

Partition Key:

```text id="u5q2m9"
match_id
```

Benefits:

* Ordered score processing

Events:

```text id="w9m4q1"
Runs

Wickets

Boundaries
```

Remain consistent.

---

# Live Score Example

Architecture:

```text id="k4q8m2"
Provider
 ↓
Kafka
 ↓
Partitions By Match
```

Benefits:

* Parallel match processing

---

# Monitoring Partitions

Track:

## Partition Count

Scalability indicator.

---

## Consumer Lag

Processing health.

---

## Rebalance Frequency

Stability indicator.

---

## Leader Availability

Cluster health.

---

## Replication Status

Fault tolerance.

---

# Security Considerations

## Access Control

Protect topics.

---

## Encryption

Secure data.

---

## Audit Logging

Track activity.

---

## Quotas

Prevent abuse.

---

# Common Mistakes

## Too Few Partitions

Scaling limitations.

---

## Too Many Partitions

Operational complexity.

---

## Poor Key Selection

Uneven load distribution.

---

## Ignoring Rebalances

Performance impact.

---

## Assuming Global Ordering

Incorrect assumption.

---

# Common Interview Questions

### What is a Kafka partition?

An ordered, append-only event log within a topic.

---

### Why are partitions important?

They enable scalability and parallelism.

---

### Does Kafka guarantee ordering?

Yes, within a partition.

---

### What is a partition key?

A value used to determine partition placement.

---

### Why use order_id as a partition key?

To preserve event ordering for a specific order.

---

### What happens during rebalancing?

Partitions are reassigned among consumers.

---

# Production Lessons

* Partitions are the core scaling mechanism in Kafka.
* Ordering exists only within partitions.
* Key selection is critical.
* Rebalancing impacts performance.
* Replication improves availability.
* Partition count directly affects scalability.
* Proper partition strategy is fundamental to successful Kafka deployments.

---

# Key Takeaways

* Partitions enable Kafka scalability and parallelism.
* Events remain ordered within a partition.
* Partition keys determine routing behavior.
* Consumer groups process partitions independently.
* Replication improves resilience.
* Partition design is one of the most important Kafka architecture decisions.
* Understanding partitions is essential for backend engineering and system design interviews.

---

# Related Documents

* docs/kafka/kafka-patterns.md
* docs/kafka/consumer-groups.md
* docs/kafka/exactly-once.md
* docs/architecture/event-driven-architecture.md

Related Diagram:

* diagrams/kafka-architecture.mmd
