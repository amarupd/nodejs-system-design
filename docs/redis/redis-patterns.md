# Redis Architecture Patterns

![Redis](../../assets/redis.png)

---

# Overview

Redis is one of the most widely used technologies in modern backend systems.

Used by:

* Amazon
* Netflix
* Uber
* Airbnb
* Shopify
* Discord
* Dream11

Redis is far more than a cache.

It is commonly used for:

* Caching
* Session Storage
* Rate Limiting
* Distributed Locking
* Pub/Sub
* Leaderboards
* Queues
* Real-Time Systems

Because Redis operates primarily in memory, it provides extremely low latency and very high throughput.

---

# Why Redis Matters

Without Redis:

```text
Application
    ↓
Database
```

Every request reaches the database.

Problems:

* Increased latency
* High database load
* Poor scalability

---

With Redis:

```text
Application
    ↓
Redis
    ↓
Database
```

Benefits:

* Faster responses
* Reduced database load
* Higher throughput

---

# Redis Characteristics

### In-Memory Storage

Data stored in RAM.

---

### High Performance

Typical latency:

```text
< 1 ms
```

---

### Rich Data Structures

Supports:

* Strings
* Hashes
* Lists
* Sets
* Sorted Sets
* Streams
* Bitmaps

---

### Persistence Options

Supports:

* RDB Snapshots
* AOF Logs

---

### Replication

Supports read scaling.

---

# Redis Architecture

```text
Clients
   ↓
Redis
   ↓
Persistence
```

Large deployments:

```text
Clients
   ↓
Redis Cluster
   ↓
Replication
```

---

# Pattern 1: Cache-Aside

Most common Redis pattern.

Flow:

```text
Request
 ↓
Redis
 ↓
Database
```

Example:

```js
const cache = await redis.get(key)

if (cache) {
  return JSON.parse(cache)
}

const user = await db.getUser(id)

await redis.set(
  key,
  JSON.stringify(user)
)

return user
```

Benefits:

* Simplicity
* Flexibility

---

# Cache Key Design

Bad:

```text
user
```

Good:

```text
user:1001
```

Better:

```text
prod:user:1001
```

Benefits:

* Easier invalidation
* Better organization

---

# Pattern 2: Session Storage

Store user sessions centrally.

Architecture:

```text
User
 ↓
API
 ↓
Redis Session
```

Example:

```text
session:abc123
```

Contains:

```json
{
  "user_id": 101
}
```

Benefits:

* Shared sessions
* Horizontal scaling

---

# Pattern 3: Rate Limiting

Store counters in Redis.

Example:

```text
rate:user:101
```

Flow:

```text
Request
 ↓
INCR
 ↓
Check Limit
```

Benefits:

* Distributed enforcement
* High performance

---

# Pattern 4: Distributed Locking

Problem:

```text
Inventory = 1
```

Two users purchase simultaneously.

---

Solution:

```text
SET lock:item:1
NX
EX 10
```

Flow:

```text
Acquire Lock
 ↓
Critical Section
 ↓
Release Lock
```

Benefits:

* Prevent race conditions

---

# Pattern 5: Leaderboards

Redis Sorted Sets are ideal.

Example:

```text
contest:1001
```

Data:

```text
UserA → 250

UserB → 300

UserC → 100
```

---

Operations:

```text
ZADD

ZRANK

ZREVRANGE
```

Benefits:

* Real-time ranking
* Excellent performance

---

# Fantasy Sports Example

Architecture:

```text
Scoring Service
       ↓
Redis Sorted Set
       ↓
Leaderboard API
```

Millions of updates supported.

---

# Pattern 6: Pub/Sub

Real-time messaging pattern.

Architecture:

```text
Publisher
    ↓
Redis Pub/Sub
    ↓
Subscribers
```

Example:

```text
live-score
```

channel.

Benefits:

* Low latency
* Simple implementation

---

# Live Score Example

```text
Score Service
      ↓
Redis Pub/Sub
      ↓
Socket Servers
```

Broadcasts:

* Runs
* Wickets
* Goals
* Match Events

---

# Pattern 7: Queues

Redis Lists commonly used.

Producer:

```text
LPUSH
```

Consumer:

```text
BRPOP
```

Architecture:

```text
Producer
   ↓
Redis Queue
   ↓
Worker
```

---

Use Cases

* Emails
* Notifications
* Background jobs

---

# Pattern 8: Delayed Jobs

Schedule future work.

Example:

```text
Send Reminder
After 30 Minutes
```

Implementation:

```text
Sorted Set
```

Score:

```text
Timestamp
```

Workers process ready jobs.

---

# Pattern 9: Presence Tracking

Used by:

* Chat Systems
* Gaming Platforms
* Collaboration Tools

Store:

```text
online:user:101
```

TTL:

```text
60 Seconds
```

Benefits:

* Real-time presence

---

# Pattern 10: Real-Time Analytics

Track:

```text
Page Views

Active Users

Clicks
```

Redis counters:

```text
INCR
```

Benefits:

* High throughput
* Near real-time reporting

---

# Pattern 11: Inventory Counters

Example:

```text
product:100:stock
```

Value:

```text
250
```

Operation:

```text
DECR
```

Benefits:

* Atomic updates

---

# Pattern 12: OTP Storage

Store:

```text
otp:user:101
```

TTL:

```text
300 Seconds
```

Benefits:

* Automatic expiration

---

# Redis Persistence

Redis is memory-first.

Persistence options:

---

## RDB

Snapshots.

Benefits:

* Compact
* Fast recovery

---

## AOF

Append-only logs.

Benefits:

* Better durability

---

Production often combines both.

---

# Redis Replication

Architecture:

```text
Primary
   ↓
Replica
```

Benefits:

* Read scaling
* Failover support

---

# Redis Sentinel

Provides:

* Monitoring
* Failover
* Leader election

Architecture:

```text
Sentinel
   ↓
Redis Nodes
```

---

# Redis Cluster

Used for large-scale deployments.

Architecture:

```text
Shard 1

Shard 2

Shard 3
```

Benefits:

* Horizontal scaling
* Higher capacity

---

# Memory Optimization

Strategies:

## TTL Usage

Avoid stale data.

---

## Compression

Reduce memory usage.

---

## Smaller Payloads

Improve efficiency.

---

## Key Expiration

Automatic cleanup.

---

# Monitoring Redis

Track:

## Memory Usage

Capacity planning.

---

## Hit Ratio

Cache efficiency.

---

## Latency

Performance.

---

## Connections

System load.

---

## Evictions

Memory pressure.

---

# Ecommerce Example

Redis Stores:

```text
Carts

Sessions

Inventory

Product Cache
```

Benefits:

* Faster checkout
* Reduced DB load

---

# Fantasy Sports Example

Redis Stores:

```text
Leaderboards

Scores

Sessions

Locks
```

Benefits:

* Real-time performance

---

# Live Score Example

Redis Stores:

```text
Current Scores

Commentary

Subscriptions
```

Benefits:

* Low-latency updates

---

# Common Mistakes

## No TTL

Leads to stale data.

---

## Large Objects

Consumes memory unnecessarily.

---

## No Monitoring

Causes outages.

---

## Using Redis As Primary Database

Not always appropriate.

---

## Poor Key Naming

Difficult maintenance.

---

# Common Interview Questions

### Why is Redis fast?

Because data is primarily stored in memory.

---

### What is Cache Aside?

A caching pattern where applications load data into cache on demand.

---

### Why use Sorted Sets?

Efficient ranking and leaderboard operations.

---

### What is Redis Pub/Sub?

A publish-subscribe messaging mechanism.

---

### What is Redis Sentinel?

A monitoring and failover solution.

---

### Why use Redis for rate limiting?

Fast counters and expiration support.

---

# Production Lessons

* Redis is far more than a cache.
* Sorted Sets are perfect for leaderboards.
* Pub/Sub enables low-latency messaging.
* Distributed locking prevents race conditions.
* Monitoring is critical.
* Memory management matters.
* Redis is a core building block of modern distributed systems.

---

# Key Takeaways

* Redis is a foundational infrastructure component.
* It supports caching, messaging, locking, analytics, and real-time systems.
* Proper key design and TTL management are essential.
* Replication and clustering improve scalability.
* Redis is heavily used in ecommerce, fantasy sports, fintech, and SaaS platforms.
* Understanding Redis patterns is essential for senior backend engineers and system design interviews.

---

# Related Documents

* docs/architecture/caching-strategy.md
* docs/redis/distributed-locking.md
* docs/redis/pubsub.md
* docs/architecture/live-score-system.md

Related Diagram:

* diagrams/redis-architecture.mmd
