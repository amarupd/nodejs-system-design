# Event-Driven Architecture (EDA)

![Kafka Architecture](../../assets/kafka.png)

---

# Overview

Event-Driven Architecture (EDA) is one of the most important architectural patterns in modern distributed systems.

Large-scale platforms such as:

* Amazon
* Netflix
* Uber
* Airbnb
* Shopify
* Dream11

heavily rely on event-driven communication.

Instead of services calling each other directly, services communicate by publishing and consuming events.

This approach enables:

* Scalability
* Reliability
* Loose coupling
* Independent deployments
* Better fault tolerance

EDA is a foundational concept for senior backend engineering and system design interviews.

---

# What Is An Event?

An event represents something that happened in the system.

Examples:

```text
USER_REGISTERED

ORDER_CREATED

PAYMENT_SUCCESS

CONTEST_JOINED

MATCH_STARTED

ORDER_SHIPPED
```

Events describe facts.

They should be immutable.

---

# Traditional Architecture

Synchronous communication:

```text
Order Service
      ↓
Payment Service
      ↓
Notification Service
```

Problems:

* Tight coupling
* Cascading failures
* Hard scaling
* Difficult deployments

---

# Event-Driven Architecture

```text
Order Service
      ↓
Kafka
      ↓
Payment Service

Notification Service

Analytics Service
```

Benefits:

* Independent services
* Better scalability
* Easier evolution

---

# Core Components

## Event Producer

Creates events.

Example:

```text
Order Service
```

Publishes:

```text
ORDER_CREATED
```

---

## Event Broker

Responsible for:

* Delivery
* Persistence
* Distribution

Examples:

* Kafka
* RabbitMQ
* AWS SNS
* AWS EventBridge

---

## Event Consumer

Consumes events.

Examples:

```text
Notification Service

Analytics Service

Recommendation Service
```

---

# High-Level Architecture

```text
Producer
   ↓
Broker
   ↓
Consumers
```

One producer may have many consumers.

---

# Why EDA Matters

Without EDA:

```text
Service A
 ↓
Service B
 ↓
Service C
 ↓
Service D
```

If Service C fails:

```text
Everything Breaks
```

---

With EDA:

```text
Producer
 ↓
Kafka
 ↓
Consumers
```

Consumers fail independently.

System remains operational.

---

# Example: Ecommerce Order Flow

Traditional:

```text
Order Service
 ↓
Payment Service
 ↓
Inventory Service
 ↓
Notification Service
```

---

Event Driven:

```text
Order Created
      ↓
Kafka
      ↓
Payment Consumer

Inventory Consumer

Notification Consumer

Analytics Consumer
```

Benefits:

* Better scalability
* Easier maintenance

---

# Event Structure

Example:

```json
{
  "event": "ORDER_CREATED",
  "order_id": 1001,
  "user_id": 500,
  "timestamp": "2026-01-01T10:00:00Z"
}
```

Important:

Events should contain enough context for consumers.

---

# Event Naming

Good:

```text
ORDER_CREATED
```

Bad:

```text
PROCESS_ORDER
```

Events describe facts.

Not commands.

---

# Event Lifecycle

```text
Business Action
      ↓
Event Published
      ↓
Broker
      ↓
Consumers
      ↓
Processing
```

---

# Kafka-Based Architecture

```text
Services
    ↓
Kafka Topics
    ↓
Consumers
```

Examples:

```text
orders

payments

notifications

matches
```

---

# RabbitMQ-Based Architecture

Typically used for:

* Background jobs
* Work queues
* Task processing

Example:

```text
Notification Queue
      ↓
Workers
```

---

# Event Ordering

Critical for many systems.

Example:

```text
ORDER_CREATED

PAYMENT_SUCCESS

ORDER_SHIPPED
```

Incorrect ordering:

```text
ORDER_SHIPPED

ORDER_CREATED
```

Creates inconsistencies.

---

# Ordering Solutions

Kafka:

```text
Partition Ordering
```

Events for a single entity should use the same partition key.

Example:

```text
order_id
```

---

# Idempotency

Consumers may receive duplicate events.

Example:

```text
PAYMENT_SUCCESS
```

Processed twice.

Bad result:

```text
Double Wallet Credit
```

---

Solution:

```text
Idempotent Consumers
```

Store:

```text
event_id
```

Ignore duplicates.

---

# Event Versioning

Events evolve over time.

Versioning example:

```text
ORDER_CREATED_V1

ORDER_CREATED_V2
```

or

```json
{
  "version": 2
}
```

Benefits:

* Backward compatibility

---

# Event Replay

One major Kafka advantage.

Example:

```text
Analytics Service Failed
```

Replay:

```text
Historical Events
```

Benefits:

* Recovery
* Reprocessing
* Debugging

---

# Event Sourcing

Advanced architecture pattern.

Instead of storing current state:

Store:

```text
All Events
```

Example:

```text
ACCOUNT_CREATED

MONEY_DEPOSITED

MONEY_WITHDRAWN
```

Current state derived from events.

---

# CQRS Integration

Often combined with EDA.

Command Side:

```text
Writes
```

Query Side:

```text
Reads
```

Benefits:

* Independent scaling
* Better performance

---

# Ecommerce Example

Event:

```text
ORDER_CREATED
```

Consumers:

```text
Inventory Service

Notification Service

Analytics Service
```

---

Event:

```text
PAYMENT_SUCCESS
```

Consumers:

```text
Shipment Service

Rewards Service

Reporting Service
```

---

# Fantasy Sports Example

Event:

```text
PLAYER_SCORED_RUN
```

Consumers:

```text
Scoring Service

Leaderboard Service

Analytics Service
```

---

Event:

```text
MATCH_COMPLETED
```

Consumers:

```text
Settlement Service

Notification Service
```

---

# Live Score Example

Architecture:

```text
Provider
 ↓
Kafka
 ↓
Score Service
 ↓
WebSockets
```

Benefits:

* Real-time delivery
* Massive scalability

---

# Notification Example

Event:

```text
ORDER_SHIPPED
```

Consumers:

```text
Email Service

SMS Service

Push Service
```

No direct coupling.

---

# Saga Pattern

Used for distributed transactions.

Example:

```text
Order Service
 ↓
Payment Service
 ↓
Inventory Service
```

If one step fails:

```text
Compensation Events
```

Rollback workflow.

---

# Dead Letter Queues

Failed events:

```text
Consumer Failure
```

Move to:

```text
DLQ
```

Benefits:

* Reliability
* Easier troubleshooting

---

# Monitoring Metrics

Track:

## Event Throughput

Messages per second.

---

## Consumer Lag

Processing delay.

---

## Failed Events

Reliability indicator.

---

## Retry Count

System health metric.

---

## DLQ Size

Operational risk indicator.

---

# Security Considerations

## Event Validation

Reject malformed events.

---

## Access Control

Restrict topic access.

---

## Encryption

Protect sensitive data.

---

## Audit Logging

Track event flows.

---

# Common Mistakes

## Events As Commands

Events should represent facts.

---

## No Idempotency

Causes duplicate processing.

---

## Large Event Payloads

Increase infrastructure costs.

---

## No Versioning

Creates upgrade challenges.

---

## Tight Coupling Through Events

Avoid consumer assumptions.

---

# Common Interview Questions

### What is Event-Driven Architecture?

A system where services communicate through events rather than direct calls.

---

### Why use Kafka?

Reliable, scalable event streaming.

---

### Why is idempotency important?

To safely handle duplicate event delivery.

---

### What is event replay?

Reprocessing historical events from the broker.

---

### What is a Saga?

A distributed transaction pattern using compensating actions.

---

### What is a Dead Letter Queue?

A queue for events that repeatedly fail processing.

---

# Production Lessons

* Event-driven systems scale better than tightly coupled architectures.
* Kafka is the most common event backbone.
* Consumers must be idempotent.
* Monitoring consumer lag is critical.
* DLQs improve reliability.
* Event versioning prevents breaking changes.
* EDA is foundational for modern microservices.

---

# Key Takeaways

* Event-Driven Architecture enables loosely coupled systems.
* Events represent facts, not commands.
* Kafka and RabbitMQ are common event brokers.
* Idempotency is mandatory.
* Ordering, replay, and versioning are critical considerations.
* EDA improves scalability and reliability.
* Modern distributed systems rely heavily on event-driven communication.

---

# Related Documents

* docs/kafka/kafka-patterns.md
* docs/rabbitmq/rabbitmq-patterns.md
* docs/architecture/distributed-systems.md
* docs/architecture/notification-system.md
* docs/architecture/payment-system.md

Related Diagram:

* diagrams/event-driven-system.mmd
