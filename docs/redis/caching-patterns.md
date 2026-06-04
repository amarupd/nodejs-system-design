# Redis Caching Patterns

![Redis](../../assets/redis.png)

---

# Overview

Caching is one of the most effective ways to improve application performance and scalability.

In modern distributed systems, Redis is the most widely used caching solution because it provides:

* Extremely low latency
* High throughput
* Distributed access
* Flexible data structures
* Simple operational model

A well-designed cache can reduce database load by more than 90% and dramatically improve user experience.

This document covers the most important caching patterns used in production-grade systems.

---

# Why Caching Exists

Without cache:

```text
Client
  ↓
API
  ↓
Database
  ↓
Response
```

Every request reaches the database.

Problems:

* High latency
* Expensive queries
* Database bottlenecks
* Poor scalability

---

With cache:

```text
Client
  ↓
API
  ↓
Redis
  ↓
Database
```

Benefits:

* Faster responses
* Lower database load
* Better scalability

---

# Cache Fundamentals

## Cache Hit

Data found in cache.

```text
Request
  ↓
Redis
  ↓
Response
```

Fastest path.

---

## Cache Miss

Data not found.

```text
Request
  ↓
Redis
  ↓
Database
  ↓
Redis
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

Target:

```text
90%+
```

for heavily cached workloads.

---

# Cache Pattern 1: Cache Aside

Most popular caching strategy.

Also called:

```text
Lazy Loading
```

---

# Flow

```text
Request
  ↓
Cache Check
  ↓
Miss
  ↓
Database
  ↓
Store In Cache
  ↓
Response
```

---

# Example

```js
let user = await redis.get(key)

if (!user) {
  user = await db.getUser(id)

  await redis.set(
    key,
    JSON.stringify(user),
    "EX",
    300
  )
}

return user
```

---

# Advantages

* Simple implementation
* Flexible
* Widely adopted

---

# Disadvantages

* First request is slower
* Cache misses hit database

---

# Best Use Cases

* Product details
* User profiles
* Categories
* Configuration data

---

# Cache Pattern 2: Read Through

Application reads from cache layer.

Architecture:

```text
Application
     ↓
Cache Layer
     ↓
Database
```

Cache automatically loads missing data.

---

# Flow

```text
Request
  ↓
Cache
  ↓
Database
  ↓
Cache
  ↓
Response
```

---

# Advantages

* Simpler application code
* Consistent behavior

---

# Disadvantages

* More complex cache layer

---

# Best Use Cases

* Shared platform infrastructure
* Enterprise systems

---

# Cache Pattern 3: Write Through

Writes occur to cache and database simultaneously.

Architecture:

```text
Application
     ↓
Cache
     ↓
Database
```

---

# Flow

```text
Update
  ↓
Cache
  ↓
Database
```

---

# Advantages

* Cache always up to date

---

# Disadvantages

* Increased write latency

---

# Best Use Cases

* Frequently accessed data
* Configuration systems

---

# Cache Pattern 4: Write Back

Also called:

```text
Write Behind
```

Writes occur to cache first.

Database updated asynchronously.

---

# Flow

```text
Write
 ↓
Cache
 ↓
Queue
 ↓
Database
```

---

# Advantages

* Extremely fast writes

---

# Risks

* Potential data loss
* More complexity

---

# Best Use Cases

* Analytics
* Event collection
* Non-critical workloads

---

# Cache Pattern 5: Refresh Ahead

Refresh data before expiration.

---

# Flow

```text
Cache Entry
      ↓
Approaching Expiry
      ↓
Background Refresh
```

Benefits:

* Avoids cache misses
* Improves latency

---

# Best Use Cases

* Product catalogs
* Configuration data
* Frequently accessed records

---

# Cache Pattern 6: Stale While Revalidate

Return stale data immediately.

Refresh asynchronously.

---

# Flow

```text
Request
 ↓
Return Stale Value
 ↓
Background Refresh
```

Benefits:

* Low latency
* Better availability

---

# Best Use Cases

* Product listings
* News feeds
* Public content

---

# Multi-Level Caching

Large systems use multiple cache layers.

Architecture:

```text
Browser Cache
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

* Lower latency
* Reduced backend load

---

# Browser Caching

Useful for:

* Images
* CSS
* JavaScript

Headers:

```http
Cache-Control
ETag
Expires
```

Benefits:

* Faster page loads

---

# CDN Caching

Architecture:

```text
User
 ↓
CDN Edge
 ↓
Origin
```

Benefits:

* Global performance
* Reduced traffic

---

# API Response Caching

Example:

```text
GET /products
```

Store response in Redis.

Benefits:

* Reduced compute
* Faster APIs

---

# Database Query Caching

Example:

```sql
SELECT *
FROM products
WHERE category_id = 1
```

Cache result.

Benefits:

* Lower query load

---

# Session Caching

Store:

```text
session:token
```

Benefits:

* Fast authentication
* Shared sessions

---

# Product Catalog Caching

Ecommerce Example:

Cache:

```text
products

categories

brands
```

TTL:

```text
5–30 Minutes
```

Benefits:

* Faster browsing

---

# Leaderboard Caching

Fantasy Sports Example:

Store:

```text
contest:leaderboard
```

Using:

```text
Redis Sorted Sets
```

Benefits:

* Real-time ranking

---

# Live Score Caching

Store:

```text
match:1001
```

Contains:

* Score
* Overs
* Commentary

Benefits:

* Low-latency access

---

# Cache Key Design

Bad:

```text
user
```

---

Good:

```text
user:101
```

---

Better:

```text
prod:user:101
```

Benefits:

* Namespacing
* Easier invalidation

---

# TTL Strategy

Different data requires different lifetimes.

Example:

Product Details:

```text
30 Minutes
```

Live Score:

```text
5 Seconds
```

Configuration:

```text
24 Hours
```

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

# Invalidation Strategies

## TTL Expiration

Automatic expiration.

---

## Manual Deletion

```js
await redis.del(key)
```

---

## Event-Based Invalidation

```text
Update
 ↓
Kafka Event
 ↓
Cache Delete
```

Best for microservices.

---

# Cache Stampede

Problem:

Popular key expires.

Thousands of requests hit database simultaneously.

---

Architecture:

```text
Expired Cache
      ↓
1000 Requests
      ↓
Database Spike
```

---

# Solutions

## Distributed Locking

Only one request rebuilds cache.

---

## Stale While Revalidate

Serve stale content.

---

## Refresh Ahead

Refresh before expiration.

---

# Hot Key Problem

Example:

```text
india-vs-pakistan
```

Millions of reads.

Can overload Redis.

---

# Solutions

* Replication
* Local cache
* CDN
* Key sharding

---

# Monitoring Metrics

Track:

## Hit Ratio

Cache efficiency.

---

## Miss Ratio

Potential problems.

---

## Memory Usage

Capacity planning.

---

## Evictions

Memory pressure.

---

## Latency

Cache performance.

---

# Common Mistakes

## No TTL

Leads to stale data.

---

## Caching Everything

Wastes memory.

---

## Large Payloads

Consumes resources.

---

## Ignoring Stampedes

Can overload databases.

---

## Weak Key Naming

Difficult maintenance.

---

# Common Interview Questions

### What is Cache Aside?

A lazy-loading cache strategy.

---

### What is Write Through?

Writes update cache and database simultaneously.

---

### What is Write Back?

Writes update cache first and database later.

---

### What is cache stampede?

Many requests rebuilding an expired cache entry.

---

### Why use Redis?

Low-latency distributed caching.

---

### Why use TTL?

To automatically remove stale data.

---

# Production Lessons

* Cache Aside is the most common strategy.
* TTL management is critical.
* Cache invalidation requires careful planning.
* Stampede protection is mandatory for popular data.
* Multi-level caching improves scalability.
* Monitoring cache performance is essential.
* Redis remains the industry standard for distributed caching.

---

# Key Takeaways

* Caching dramatically improves performance.
* Redis powers most production caching systems.
* Cache Aside is the most widely adopted pattern.
* Different workloads require different caching strategies.
* Stampedes and hot keys must be handled proactively.
* Effective caching reduces latency and infrastructure costs.
* Caching is a core skill for senior backend engineers.

---

# Related Documents

* docs/redis/cache-invalidation.md
* docs/redis/redis-patterns.md
* docs/architecture/caching-strategy.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/redis-architecture.mmd
