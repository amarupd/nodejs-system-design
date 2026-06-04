# Apache Kafka Architecture Patterns

![Kafka](../../assets/kafka.png)

---

# Overview

Apache Kafka is the industry standard platform for event streaming and large-scale distributed messaging.

It is used by companies such as:

* Netflix
* Uber
* LinkedIn
* Airbnb
* Amazon
* Shopify

Kafka is designed to handle:

* Millions of events per second
* Real-time data pipelines
* Event-driven architectures
* Stream processing
* Analytics workloads

Unlike traditional message queues, Kafka is built around durable event logs and replayable streams.

---

# Why Kafka Exists

Traditional architecture:

```text id="r4m7q2"
Service A
 ↓
Service B
```

Problems:

* Tight coupling
* Limited scalability
* Failure propagation

---

Kafka architecture:

```text id="m8q3r1"
Producer
 ↓
Kafka
 ↓
Consumers
```

Benefits:

* Decoupling
* Durability
* Scalability

---

# Core Kafka Concepts

Understanding these concepts is critical.

---

## Producer

Publishes events.

Example:

```text id="t2m6q8"
Order Service
```

Produces:

```text id="v7q1m4"
ORDER_CREATED
```

---

## Topic

Logical event stream.

Example:

```text id="p9m3q7"
orders
```

Topics contain events.

---

## Partition

Sub-division of a topic.

Example:

```text id="u3q8m2"
orders
 ├─ Partition 1
 ├─ Partition 2
 └─ Partition 3
```

Partitions enable scalability.

---

## Consumer

Reads events.

Example:

```text id="k5m1q9"
Notification Service
```

---

## Consumer Group

Multiple consumers working together.

Architecture:

```text id="n7q4m3"
Topic
 ↓
Consumer Group
```

Enables horizontal scaling.

---

# High-Level Architecture

```text id="x2m8q5"
Producer
 ↓
Topic
 ↓
Partitions
 ↓
Consumer Groups
```

This is the foundation of Kafka.

---

# Kafka vs RabbitMQ

RabbitMQ:

```text id="m4q7r2"
Message Queue
```

Kafka:

```text id="t8m1q6"
Event Log
```

RabbitMQ focuses on:

* Task processing
* Work queues

Kafka focuses on:

* Event streaming
* Replayability
* Massive throughput

---

# Pattern 1: Event Notification

Producer emits event.

Example:

```text id="v1q9m4"
ORDER_CREATED
```

Consumers:

```text id="u6m3q8"
Notification Service

Analytics Service
```

Benefits:

* Loose coupling

---

# Pattern 2: Event-Carried State Transfer

Event contains required data.

Example:

```json id="k2q7m1"
{
  "order_id": 101,
  "user_id": 501,
  "amount": 999
}
```

Benefits:

* Fewer service calls
* Better scalability

---

# Pattern 3: Event Sourcing

Store events instead of current state.

Events:

```text id="r5m8q2"
ACCOUNT_CREATED

MONEY_DEPOSITED

MONEY_WITHDRAWN
```

Current state reconstructed from events.

Benefits:

* Auditability
* Replay capability

---

# Pattern 4: CQRS

Command Query Responsibility Segregation.

Writes:

```text id="w8q3m5"
Commands
```

Reads:

```text id="p4m1q9"
Queries
```

Kafka synchronizes read models.

Benefits:

* Independent scaling

---

# Pattern 5: Data Replication

Architecture:

```text id="n2q6m8"
Database A
 ↓
Kafka
 ↓
Database B
```

Benefits:

* Synchronization
* Integration

---

# Pattern 6: Audit Logging

Every event stored permanently.

Example:

```text id="x7m4q1"
User Login

Password Change

Payment Success
```

Benefits:

* Compliance
* Forensics

---

# Pattern 7: Stream Processing

Events processed continuously.

Architecture:

```text id="v3q8m6"
Kafka
 ↓
Stream Processor
 ↓
Results
```

Examples:

* Analytics
* Fraud Detection
* Metrics

---

# Pattern 8: Fan-Out Processing

Single event.

Many consumers.

Architecture:

```text id="k9m2q4"
Producer
 ↓
Kafka
 ↓
Consumer A

Consumer B

Consumer C
```

Benefits:

* Independent services

---

# Pattern 9: Event Replay

One of Kafka's strongest features.

Example:

```text id="r6m1q8"
Consumer Failed
```

Replay:

```text id="u2q7m5"
Historical Events
```

Benefits:

* Recovery
* Reprocessing

---

# Topic Design

Good topics:

```text id="m7q3r2"
orders

payments

notifications
```

Avoid:

```text id="t5m8q1"
everything
```

Benefits:

* Clear ownership
* Better scalability

---

# Partitioning Strategy

Critical design decision.

Example:

```text id="p8m4q6"
order_id
```

as key.

Benefits:

* Ordering per order
* Balanced distribution

---

# Ordering Guarantees

Kafka guarantees ordering:

```text id="n4q1m7"
Within A Partition
```

Example:

```text id="u8m3q2"
Event 1

Event 2

Event 3
```

Order preserved.

---

Across partitions:

```text id="x5q7m4"
No Global Ordering
```

---

# Consumer Groups

Example:

```text id="k1m8q5"
Consumer A

Consumer B

Consumer C
```

Each partition processed by one consumer.

Benefits:

* Horizontal scaling

---

# Offset Management

Kafka tracks:

```text id="p3q9m2"
Offset
```

Example:

```text id="r8m4q7"
Event #101
```

Consumers resume from stored offsets.

Benefits:

* Reliability
* Replay

---

# Ecommerce Example

Event:

```text id="u4m7q1"
ORDER_CREATED
```

Consumers:

```text id="v9q2m5"
Inventory

Notifications

Analytics
```

Benefits:

* Decoupling

---

# Payment Example

Events:

```text id="n7m3q8"
PAYMENT_SUCCESS

PAYMENT_FAILED
```

Consumers:

```text id="k6q1m4"
Ledger

Reporting

Notifications
```

---

# Fantasy Sports Example

Events:

```text id="t1m8q6"
PLAYER_SCORED
```

Consumers:

```text id="x3q5m2"
Scoring Service

Leaderboard Service
```

Benefits:

* Real-time updates

---

# Live Score Example

Architecture:

```text id="u7m2q9"
Data Provider
 ↓
Kafka
 ↓
Score Service
 ↓
WebSockets
```

Millions of users supported.

---

# Monitoring Kafka

Track:

## Consumer Lag

Processing health.

---

## Throughput

Events per second.

---

## Topic Size

Storage growth.

---

## Replication Status

Cluster health.

---

## Broker Availability

Infrastructure reliability.

---

# Security Considerations

## Authentication

Control access.

---

## Authorization

Topic permissions.

---

## Encryption

TLS protection.

---

## Audit Logs

Track activity.

---

# Common Mistakes

## Too Few Partitions

Scaling limitations.

---

## Too Many Partitions

Operational complexity.

---

## Large Events

Storage inefficiency.

---

## Ignoring Consumer Lag

Causes delays.

---

## Weak Topic Design

Ownership confusion.

---

# Common Interview Questions

### What is Kafka?

A distributed event-streaming platform.

---

### What is a partition?

A scalable sub-division of a topic.

---

### Why use consumer groups?

To scale consumers horizontally.

---

### What is an offset?

The position of an event within a partition.

---

### Why is Kafka different from RabbitMQ?

Kafka focuses on durable event logs and replay.

---

### What is event replay?

Reprocessing historical events.

---

# Production Lessons

* Kafka is the backbone of many event-driven systems.
* Topic design matters.
* Partition strategy affects scalability and ordering.
* Consumer lag must be monitored.
* Replayability is extremely valuable.
* Event-driven systems scale naturally with Kafka.
* Kafka is a core technology for modern backend engineering.

---

# Key Takeaways

* Kafka is designed for large-scale event streaming.
* Topics and partitions enable scalability.
* Consumer groups enable parallel processing.
* Kafka provides durable, replayable event storage.
* Ordering is guaranteed within partitions.
* Kafka powers ecommerce, fintech, analytics, and real-time platforms.
* Understanding Kafka is essential for senior backend engineers and system design interviews.

---

# Related Documents

* docs/kafka/partitions.md
* docs/kafka/consumer-groups.md
* docs/kafka/event-streaming.md
* docs/architecture/event-driven-architecture.md

Related Diagram:

* diagrams/kafka-architecture.mmd
