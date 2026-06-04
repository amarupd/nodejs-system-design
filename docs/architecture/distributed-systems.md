# Distributed Systems Architecture

![Architecture](../../assets/architecture.png)

---

# Overview

A distributed system is a collection of independent services and machines that work together to appear as a single system.

Modern platforms such as:

* Amazon
* Netflix
* Google
* Uber
* Shopify
* Dream11

are all distributed systems.

As applications grow beyond the capacity of a single server, engineers must design architectures that can:

* Scale horizontally
* Remain highly available
* Recover from failures
* Handle network issues
* Process massive workloads

Distributed systems are one of the most important topics in senior-level backend engineering and system design interviews.

---

# Why Distributed Systems Exist

Single Server Architecture:

```text
Users
  ↓
Application
  ↓
Database
```

Eventually reaches limits:

* CPU limits
* Memory limits
* Network limits
* Storage limits

As traffic increases:

```text
10 Users
100 Users
1,000 Users
100,000 Users
10,000,000 Users
```

A single machine becomes insufficient.

---

# Distributed Architecture

```text
Users
  ↓
Load Balancer
  ↓
Service Cluster
  ↓
Databases
Caches
Queues
```

Benefits:

* Scalability
* Reliability
* Fault tolerance

---

# Core Characteristics

A distributed system should provide:

### Scalability

Handle increasing traffic.

---

### Availability

Remain operational during failures.

---

### Reliability

Produce correct results.

---

### Fault Tolerance

Survive component failures.

---

### Performance

Maintain low latency.

---

# Horizontal Scaling

Vertical Scaling:

```text
Bigger Server
```

Examples:

* More CPU
* More RAM

Limitations:

* Hardware ceiling
* Increased cost

---

Horizontal Scaling:

```text
Server 1
Server 2
Server 3
Server 4
```

Benefits:

* Nearly unlimited growth
* Better redundancy

Preferred approach in modern systems.

---

# Load Balancing

Purpose:

Distribute traffic.

Architecture:

```text
Users
  ↓
Load Balancer
  ↓
Servers
```

Popular Options:

* NGINX
* HAProxy
* AWS ALB
* Cloud Load Balancers

Benefits:

* Better utilization
* High availability

---

# CAP Theorem

One of the most important distributed systems concepts.

A distributed system can only guarantee two of:

```text
Consistency
Availability
Partition Tolerance
```

---

## Consistency

All nodes see the same data.

---

## Availability

System always responds.

---

## Partition Tolerance

System continues despite network failures.

---

# CAP Tradeoffs

### CP Systems

Consistency + Partition Tolerance

Example:

```text
Banking Systems
```

Prioritize correctness.

---

### AP Systems

Availability + Partition Tolerance

Example:

```text
Social Media Feeds
```

Prioritize availability.

---

# Consistency Models

Different systems require different consistency guarantees.

---

## Strong Consistency

All reads see latest writes.

Example:

```text
Bank Balance
```

---

## Eventual Consistency

Data converges over time.

Example:

```text
Like Counts
```

Benefits:

* Better scalability

---

## Read-Your-Writes

User sees their own updates immediately.

Common in modern applications.

---

# Replication

Purpose:

Create copies of data.

Architecture:

```text
Primary
  ↓
Replica 1
Replica 2
Replica 3
```

Benefits:

* High availability
* Read scaling

---

# Database Replication

Example:

```text
Writes → Primary

Reads → Replicas
```

Benefits:

* Reduced load
* Better throughput

---

# Failover

Problem:

```text
Primary Database Fails
```

Solution:

```text
Promote Replica
```

Benefits:

* Reduced downtime

---

# Partitioning (Sharding)

Single database eventually becomes too large.

Solution:

```text
Shard 1

Shard 2

Shard 3
```

Example:

```text
Users 1-1M

Users 1M-2M

Users 2M-3M
```

Benefits:

* Horizontal database scaling

---

# Service Discovery

Microservices need to locate each other.

Architecture:

```text
Service Registry
       ↓
Services
```

Examples:

* Consul
* Eureka
* Kubernetes DNS

---

# Distributed Caching

Architecture:

```text
Services
   ↓
Redis Cluster
```

Benefits:

* Shared state
* Faster reads

---

# Distributed Messaging

Examples:

* Kafka
* RabbitMQ

Architecture:

```text
Producer
   ↓
Broker
   ↓
Consumer
```

Benefits:

* Decoupling
* Scalability

---

# Distributed Locking

Problem:

Multiple services update same resource.

Example:

```text
Inventory = 1
```

Two purchases arrive simultaneously.

Solution:

```text
Redis Lock
```

Flow:

```text
Acquire Lock
 ↓
Update
 ↓
Release Lock
```

---

# Circuit Breaker Pattern

Problem:

Downstream service failure.

Without protection:

```text
Service A
 ↓
Service B
 ↓
Timeout
```

Repeated indefinitely.

---

Solution:

```text
Circuit Open
```

Requests fail fast.

Benefits:

* System stability

---

# Retry Pattern

Temporary failures are common.

Example:

```text
Network Timeout
```

Retry:

```text
Attempt 1

Attempt 2

Attempt 3
```

Use:

* Backoff
* Jitter

---

# Bulkhead Pattern

Purpose:

Failure isolation.

Architecture:

```text
Resource Pool A

Resource Pool B
```

Failure in one area doesn't affect others.

---

# Leader Election

Needed in distributed coordination.

Examples:

* Scheduled jobs
* Cluster coordination

Only one node becomes leader.

Common tools:

* ZooKeeper
* etcd

---

# Eventual Consistency Example

Ecommerce:

```text
Order Created
 ↓
Kafka
 ↓
Inventory Update
```

Inventory may update slightly later.

Tradeoff:

* Better scalability

---

# Distributed Transactions

Challenge:

```text
Order Service

Payment Service

Inventory Service
```

One operation spans multiple services.

---

# Saga Pattern

Solution:

```text
Step 1

Step 2

Step 3
```

If failure:

```text
Compensating Action
```

Rollback business process.

---

# Observability

Distributed systems require visibility.

Three pillars:

---

## Metrics

Examples:

* CPU
* Memory
* Throughput

---

## Logs

Examples:

* Errors
* Requests
* Audit Trails

---

## Traces

Track request flow across services.

Tools:

* OpenTelemetry
* Jaeger

---

# Ecommerce Example

Architecture:

```text
Users
 ↓
API Gateway
 ↓
Services
 ├─ Catalog
 ├─ Inventory
 ├─ Cart
 ├─ Orders
 └─ Payments
```

Supporting Infrastructure:

```text
Redis

Kafka

MySQL
```

---

# Fantasy Sports Example

Architecture:

```text
Users
 ↓
Gateway
 ↓
Contest Service

Wallet Service

Scoring Service

Leaderboard Service
```

Real-Time Components:

```text
Kafka

Redis

WebSockets
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
Redis
 ↓
Socket.IO
```

Millions of concurrent users supported.

---

# Multi-Region Deployment

Architecture:

```text
US

Europe

Asia
```

Benefits:

* Lower latency
* Disaster recovery

Challenges:

* Replication
* Consistency

---

# Monitoring Metrics

Track:

## Latency

User experience.

---

## Throughput

Capacity.

---

## Error Rate

Reliability.

---

## Availability

System health.

---

## Consumer Lag

Kafka performance.

---

# Common Mistakes

## Shared Database Everywhere

Creates bottlenecks.

---

## No Observability

Makes debugging difficult.

---

## Excessive Synchronous Calls

Creates cascading failures.

---

## Ignoring CAP Tradeoffs

Leads to poor architecture decisions.

---

## No Failure Planning

Failures are inevitable.

---

# Common Interview Questions

### What is a distributed system?

A collection of independent systems working together as a single platform.

---

### What is CAP Theorem?

Consistency, Availability, and Partition Tolerance tradeoff model.

---

### What is eventual consistency?

Data becomes consistent over time rather than immediately.

---

### Why use sharding?

To horizontally scale databases.

---

### What is a Saga?

A distributed transaction pattern using compensating actions.

---

### Why are circuit breakers important?

To prevent cascading failures.

---

# Production Lessons

* Distributed systems fail constantly.
* Design for failure from the beginning.
* Scalability requires horizontal thinking.
* Observability is mandatory.
* CAP tradeoffs are unavoidable.
* Event-driven communication improves resilience.
* Reliability is more important than elegance.

---

# Key Takeaways

* Distributed systems power modern internet-scale applications.
* Horizontal scaling enables massive growth.
* CAP Theorem influences architecture decisions.
* Replication and sharding improve scalability.
* Distributed locks and messaging solve coordination challenges.
* Observability is critical.
* Understanding distributed systems is essential for senior backend engineers and system design interviews.

---

# Related Documents

* docs/architecture/event-driven-architecture.md
* docs/architecture/caching-strategy.md
* docs/architecture/live-score-system.md
* docs/kafka/kafka-patterns.md
* docs/redis/distributed-locking.md

Related Diagram:

* diagrams/distributed-system.mmd
