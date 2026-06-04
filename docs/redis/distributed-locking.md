# Redis Distributed Locking

![Redis](../../assets/redis.png)

---

# Overview

Distributed locking is a technique used to ensure that only one process, service, or server can perform a critical operation at a time.

In distributed systems, multiple instances of an application may try to update the same resource simultaneously.

Without proper coordination, this can cause:

* Race conditions
* Duplicate processing
* Data corruption
* Inventory overselling
* Double payments

Redis is one of the most common technologies used for implementing distributed locks because of its speed, simplicity, and atomic operations.

---

# Why Distributed Locking Exists

Single Server:

```text id="j9w3m8"
One Process
```

Only one execution path.

Locking is simple.

---

Distributed System:

```text id="k2v7p1"
Server A

Server B

Server C
```

All servers may attempt the same operation.

Example:

```text id="a8r4q6"
Inventory = 1
```

Two users place orders simultaneously.

Without locking:

```text id="t5n2k9"
Order #1 Success

Order #2 Success
```

Result:

```text id="m7q8v3"
Overselling
```

---

# Critical Section

A critical section is code that must be executed by only one process at a time.

Example:

```text id="p1r5m7"
Read Stock
 ↓
Update Stock
 ↓
Create Order
```

If multiple servers execute simultaneously:

```text id="h6t8k4"
Data Inconsistency
```

---

# High-Level Architecture

```text id="z4q7m2"
Application Servers
        ↓
Redis Lock
        ↓
Shared Resource
```

Only one server owns the lock.

---

# Lock Acquisition Flow

```text id="r8m3k1"
Acquire Lock
      ↓
Execute Logic
      ↓
Release Lock
```

Simple but extremely powerful.

---

# Naive Lock Example

Bad approach:

```text id="u5n7p4"
GET lock
```

Then:

```text id="v9q2m8"
SET lock
```

Problem:

Two clients can execute simultaneously.

Race condition remains.

---

# Correct Redis Lock

Use:

```redis id="c3r6p1"
SET lock:key value NX EX 30
```

Components:

```text id="x2m8k7"
NX
```

Only set if key does not exist.

---

```text id="b7p4n9"
EX 30
```

Auto-expire after 30 seconds.

---

Benefits:

* Atomic operation
* Safe acquisition

---

# Node.js Example

```js id="f8m1q5"
const acquired =
  await redis.set(
    "lock:inventory:101",
    lockId,
    {
      NX: true,
      EX: 30
    }
  )
```

Success:

```text id="q6v2r8"
Lock Acquired
```

Failure:

```text id="y1m7k3"
Lock Already Exists
```

---

# Lock Release

Important:

Only the owner should release the lock.

Bad:

```js id="w4t9p2"
await redis.del(lockKey)
```

Problem:

Another process may have acquired a new lock.

---

# Safe Release

Verify ownership.

```text id="p7m5k9"
Lock Owner Check
      ↓
Delete
```

Use Lua scripts for atomicity.

---

# Lock Expiration

Critical feature.

Problem:

```text id="n8q4r1"
Server Crash
```

Without expiration:

```text id="k3m7v2"
Permanent Lock
```

System becomes blocked.

---

Solution:

```text id="r5p8m4"
TTL
```

Locks automatically expire.

---

# Lock Renewal

Long-running jobs may exceed lock duration.

Flow:

```text id="t9v2k7"
Acquire Lock
      ↓
Renew TTL
      ↓
Continue Processing
```

Sometimes called:

```text id="m4q8r5"
Heartbeat
```

---

# Inventory Reservation Example

Ecommerce:

Stock:

```text id="u2m6k9"
1 Unit
```

Users:

```text id="y8q3r4"
User A

User B
```

---

Flow:

```text id="c6m1p7"
Acquire Lock
 ↓
Check Stock
 ↓
Reserve Stock
 ↓
Release Lock
```

Result:

Only one purchase succeeds.

---

# Contest Joining Example

Fantasy Sports:

Contest Capacity:

```text id="x7m4q2"
10,000 Users
```

Traffic:

```text id="n5p9r6"
100,000 Requests
```

during match launch.

---

Solution:

```text id="b3q7m1"
Redis Lock
```

Prevents:

```text id="v6r8k4"
Overbooking
```

---

# Payment Processing Example

Problem:

```text id="k1m5q8"
Duplicate Callback
```

Gateway retries.

---

Flow:

```text id="y4p7r2"
Acquire Lock
 ↓
Process Payment
 ↓
Release Lock
```

Benefits:

* Prevent duplicate credits
* Prevent duplicate settlements

---

# Scheduled Job Example

Multiple instances:

```text id="p9m2q6"
Worker A

Worker B

Worker C
```

Need:

```text id="u7r4k1"
Single Execution
```

---

Flow:

```text id="m6q8p3"
Acquire Lock
 ↓
Run Job
 ↓
Release Lock
```

Used for:

* Cron jobs
* Settlement jobs
* Cleanup tasks

---

# Leader Election

A lock can be used to elect a leader.

Architecture:

```text id="r3m7k9"
Node A

Node B

Node C
```

First node obtaining lock becomes:

```text id="t5q8m1"
Leader
```

Benefits:

* Coordination
* Scheduling

---

# Lock Granularity

Important design decision.

---

## Coarse-Grained Lock

Example:

```text id="n2p7r4"
lock:inventory
```

Benefits:

* Simplicity

Problem:

* Low concurrency

---

## Fine-Grained Lock

Example:

```text id="v8m3q5"
lock:inventory:101
```

Benefits:

* Better scalability
* Higher concurrency

Preferred approach.

---

# Redlock Algorithm

Advanced distributed locking approach.

Architecture:

```text id="w7p2m6"
Redis A

Redis B

Redis C

Redis D

Redis E
```

Lock acquired on majority.

Benefits:

* Higher fault tolerance

---

Important Note

Redlock remains debated in distributed systems communities.

Use when stronger guarantees are required.

---

# Lock Timeout Strategy

Never wait forever.

Example:

```text id="m1q9r7"
Acquire Lock
 ↓
Retry
 ↓
Fail
```

Benefits:

* Better user experience
* Reduced deadlocks

---

# Retry Strategy

Use:

```text id="u4p8m2"
Exponential Backoff
```

Example:

```text id="q7r3m6"
100 ms

200 ms

400 ms
```

Avoids lock contention.

---

# Monitoring Locks

Track:

## Lock Acquisition Success

System health.

---

## Lock Wait Time

Contention indicator.

---

## Lock Expirations

Potential issue indicator.

---

## Failed Acquisitions

Traffic signal.

---

# Security Considerations

## Unique Lock Tokens

Prevent accidental releases.

---

## Ownership Validation

Only owner releases lock.

---

## Expiration

Avoid deadlocks.

---

## Monitoring

Detect abnormal behavior.

---

# Common Mistakes

## No Expiration

Leads to deadlocks.

---

## Releasing Someone Else's Lock

Causes inconsistencies.

---

## Coarse Locks Everywhere

Reduces scalability.

---

## Infinite Retries

Creates cascading issues.

---

## Ignoring Monitoring

Operational risk.

---

# Common Interview Questions

### What is distributed locking?

A mechanism ensuring only one process accesses a critical section at a time.

---

### Why use Redis for distributed locks?

Atomic operations and low latency.

---

### Why use SET NX EX?

It provides atomic lock acquisition with expiration.

---

### Why should locks expire?

To recover from crashes and prevent deadlocks.

---

### What is Redlock?

A distributed locking algorithm using multiple Redis nodes.

---

### Why validate lock ownership?

To prevent releasing another process's lock.

---

# Production Lessons

* Distributed locks solve race conditions.
* Redis is one of the most popular locking solutions.
* Lock expiration is mandatory.
* Fine-grained locks improve scalability.
* Ownership validation prevents corruption.
* Monitoring lock contention is important.
* Distributed locking is essential for ecommerce, payments, and fantasy sports systems.

---

# Key Takeaways

* Distributed locking coordinates access across multiple servers.
* Redis provides simple and powerful lock primitives.
* Inventory reservations, payment processing, and contest joins commonly use locks.
* Expiration and ownership validation are mandatory.
* Fine-grained locking improves throughput.
* Redlock provides additional fault tolerance.
* Distributed locking is a fundamental distributed systems concept for senior backend engineers.

---

# Related Documents

* docs/redis/redis-patterns.md
* docs/architecture/distributed-systems.md
* docs/architecture/ecommerce-architecture.md
* docs/architecture/fantasy-sports-architecture.md

Related Diagram:

* diagrams/redis-architecture.mmd
