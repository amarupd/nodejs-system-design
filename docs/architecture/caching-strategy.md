# Caching Strategy Architecture

![Redis](../../assets/redis.png)

---

# Overview

Caching is one of the highest-impact performance optimization techniques in modern backend systems.

A well-designed cache can:

* Reduce database load
* Lower API latency
* Improve throughput
* Reduce infrastructure costs
* Improve user experience

Most large-scale systems rely heavily on caching.

Examples:

* Amazon
* Netflix
* Uber
* Airbnb
* Stripe
* Fantasy Sports Platforms
* Ecommerce Systems

Caching is often the difference between a system that supports:

```text
1,000 Requests/sec
```

and one that supports:

```text
100,000 Requests/sec
```

---

# What Is Caching?

Caching stores frequently accessed data in a fast-access storage layer.

Instead of:

```text
Request
   ↓
Database
   ↓
Response
```

Use:

```text
Request
   ↓
Cache
   ↓
Database (if needed)
```

Benefits:

* Faster responses
* Reduced database pressure
* Better scalability

---

# Why Databases Need Caching

Database queries are expensive compared to memory access.

Typical Latencies:

```text
Memory (Redis)
≈ 0.1 ms

Database Query
≈ 5-50 ms

Cross Region Query
≈ 50-200 ms
```

Even small improvements create massive gains at scale.

---

# High-Level Caching Architecture

```text
Client
   ↓
API
   ↓
Redis
   ↓
Database
```

Flow:

1. Check cache
2. Return cached value if present
3. Query database if missing
4. Store result in cache
5. Return response

---

# Cache Terminology

## Cache Hit

Requested data exists in cache.

```text
Request
 ↓
Cache Hit
 ↓
Response
```

Fastest scenario.

---

## Cache Miss

Data not found in cache.

```text
Request
 ↓
Cache Miss
 ↓
Database
 ↓
Cache Update
 ↓
Response
```

Slower path.

---

## Hit Ratio

Formula:

```text
Cache Hits
-----------
Total Requests
```

Example:

```text
95%
```

High hit ratio indicates an effective cache strategy.

---

# Types of Caching

## Application Cache

Stored inside application memory.

Example:

```js
const cache = {}
```

Advantages:

* Extremely fast

Disadvantages:

* Not shared
* Lost on restart
* Poor scalability

Not recommended for distributed systems.

---

## Distributed Cache

Example:

```text
Redis
```

Architecture:

```text
API Servers
      ↓
Redis
      ↓
Database
```

Benefits:

* Shared cache
* Horizontal scaling
* High performance

---

# Cache Patterns

Several patterns are commonly used in production.

---

# Cache Aside Pattern

Most popular strategy.

Flow:

```text
Request
 ↓
Cache Lookup
 ↓
Miss
 ↓
Database
 ↓
Store Cache
 ↓
Response
```

Example:

```js
const cached = await redis.get(key)

if (cached) {
  return JSON.parse(cached)
}

const user = await db.getUser(id)

await redis.set(key, JSON.stringify(user))

return user
```

Advantages:

* Simple
* Flexible
* Widely adopted

---

# Read Through Cache

Architecture:

```text
Application
      ↓
Cache Layer
      ↓
Database
```

Cache automatically loads missing data.

Benefits:

* Simpler application logic

---

# Write Through Cache

Flow:

```text
Write Request
      ↓
Cache
      ↓
Database
```

Benefits:

* Cache always current

Disadvantages:

* Increased write latency

---

# Write Back Cache

Flow:

```text
Write
 ↓
Cache
 ↓
Async Database Write
```

Benefits:

* Fast writes

Risks:

* Data loss during failures

Used carefully.

---

# Redis As A Cache

Most popular caching technology.

Advantages:

* In-memory
* Extremely fast
* Rich data structures
* Distributed support

Common Data Types:

* Strings
* Hashes
* Lists
* Sets
* Sorted Sets

---

# Cache Key Design

Poor keys cause maintenance problems.

Bad:

```text
user
```

Good:

```text
user:101
```

Better:

```text
prod:user:101
```

Benefits:

* Namespacing
* Easier invalidation
* Better organization

---

# TTL (Time To Live)

Cached data should expire.

Example:

```text
5 Minutes
```

Redis:

```js
SET user:101 data EX 300
```

Benefits:

* Automatic cleanup
* Prevent stale data

---

# Cache Invalidation

One of the hardest problems in software engineering.

Problem:

```text
Database Updated
```

Cache:

```text
Old Value
```

Result:

```text
Stale Data
```

---

# Cache Invalidation Strategies

## TTL Expiration

Most common.

Simple.

---

## Explicit Deletion

Example:

```js
await redis.del("user:101")
```

Immediately removes stale data.

---

## Event-Based Invalidation

Flow:

```text
Database Update
      ↓
Kafka Event
      ↓
Cache Invalidation
```

Excellent for microservices.

---

# Cache Stampede

Problem:

Popular key expires.

Thousands of requests arrive simultaneously.

```text
1000 Requests
      ↓
Cache Miss
      ↓
Database Overload
```

---

# Cache Stampede Solutions

## Distributed Lock

Flow:

```text
First Request
      ↓
Acquire Lock
      ↓
Load Data
      ↓
Update Cache
```

Other requests wait.

Redis commonly used.

---

## Stale While Revalidate

Return stale value.

Refresh cache asynchronously.

Benefits:

* Low latency
* Better resilience

---

# Hot Keys

Problem:

One key receives massive traffic.

Example:

```text
match:live-score
```

Millions of requests.

Can overload Redis.

---

# Hot Key Solutions

* Replication
* Local cache
* Key sharding
* CDN

---

# Multi-Level Caching

Architecture:

```text
Client
 ↓
CDN
 ↓
Application Cache
 ↓
Redis
 ↓
Database
```

Benefits:

* Extremely low latency
* Reduced backend load

---

# CDN Caching

Useful for:

* Images
* Videos
* Static ../../assets

Architecture:

```text
User
 ↓
CDN
 ↓
Origin Server
```

Benefits:

* Global performance
* Reduced infrastructure cost

---

# Ecommerce Example

Product Catalog

Architecture:

```text
User
 ↓
API
 ↓
Redis
 ↓
MySQL
```

Cache:

```text
Product Details
Categories
Pricing
```

Benefits:

* Faster browsing
* Reduced DB load

---

# Fantasy Sports Example

Live Leaderboard

Architecture:

```text
Users
 ↓
Redis Sorted Sets
 ↓
Leaderboard API
```

Benefits:

* Real-time ranking
* High throughput

---

# Live Score System Example

Architecture:

```text
Score Provider
       ↓
Kafka
       ↓
Score Service
       ↓
Redis
       ↓
Socket.IO
```

Redis stores:

* Current score
* Match state
* Commentary

Benefits:

* Low latency updates

---

# Monitoring Cache Systems

Track:

## Hit Ratio

Target:

```text
90%+
```

---

## Miss Ratio

Indicates inefficiency.

---

## Memory Usage

Redis memory consumption.

---

## Evictions

Keys removed due to memory pressure.

---

## Latency

Response times.

---

# Common Mistakes

## No TTL

Leads to stale data.

---

## Caching Everything

Wasteful.

Cache valuable data.

---

## Large Objects

Increase memory usage.

---

## Poor Key Naming

Makes maintenance difficult.

---

## Ignoring Invalidation

Creates stale data issues.

---

# Common Interview Questions

### What is caching?

Storing frequently accessed data in a fast-access layer.

---

### What is Cache Aside?

Application checks cache first, then database if missing.

---

### Why use Redis?

Fast in-memory storage with excellent scalability.

---

### What is cache stampede?

Many requests simultaneously rebuilding an expired cache entry.

---

### What is cache invalidation?

Removing or updating stale cached data.

---

### Why use TTL?

To automatically expire outdated cache entries.

---

# Production Lessons

* Redis is the most common cache technology.
* Cache Aside is the most widely used strategy.
* Cache invalidation requires careful planning.
* High cache hit rates dramatically improve scalability.
* Hot keys must be monitored.
* Multi-level caching provides the best performance.
* Effective caching reduces infrastructure costs significantly.

---

# Key Takeaways

* Caching is a foundational scalability strategy.
* Redis is the industry-standard distributed cache.
* Cache Aside is the most common caching pattern.
* TTLs and invalidation are critical.
* Multi-level caching improves performance dramatically.
* Production systems depend heavily on cache efficiency.
* Understanding caching is essential for system design interviews and senior backend engineering roles.

---

# Related Documents

* docs/redis/redis-patterns.md
* docs/redis/cache-invalidation.md
* docs/architecture/rate-limiting.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/redis-architecture.mmd
