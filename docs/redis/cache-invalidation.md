# Cache Invalidation Architecture

![Redis](../../assets/redis.png)

---

# Overview

Cache invalidation is the process of ensuring cached data remains consistent with the source of truth.

Many engineers consider cache invalidation one of the hardest problems in software engineering because it introduces a tradeoff between:

```text
Performance
vs
Consistency
```

Caching improves speed dramatically, but stale data can create:

* Incorrect product prices
* Outdated inventory
* Wrong user information
* Inaccurate leaderboards
* Inconsistent application behavior

A production-grade caching strategy must include a well-designed invalidation mechanism.

---

# Why Cache Invalidation Matters

Example:

Database:

```text
Product Price = ₹999
```

Cached Value:

```text
Product Price = ₹799
```

User sees:

```text
Incorrect Information
```

Result:

* Customer complaints
* Revenue loss
* Trust issues

---

# Cache Consistency Problem

Architecture:

```text
Database
   ↓
Redis Cache
   ↓
Users
```

When database updates:

```text
Database Updated
```

Cache may still contain:

```text
Old Data
```

This creates inconsistency.

---

# The Fundamental Challenge

Cache exists because:

```text
Database Access
=
Slow
```

However:

```text
Cache Data
=
Potentially Stale
```

The challenge is balancing:

* Freshness
* Performance
* Complexity

---

# Common Invalidation Strategies

Production systems generally use one or more of the following:

1. TTL Expiration
2. Manual Deletion
3. Write Through
4. Event-Based Invalidation
5. Version-Based Invalidation
6. Refresh Ahead

---

# Strategy 1: TTL Expiration

The simplest approach.

Example:

```text
Product Cache
TTL = 300 Seconds
```

Redis:

```redis
SET product:101 value EX 300
```

After expiration:

```text
Cache Removed
```

Automatically.

---

# Advantages

* Simple implementation
* Low operational complexity
* Automatic cleanup

---

# Disadvantages

* Data may remain stale until expiration
* Difficult to choose ideal TTL

---

# Best Use Cases

* Product catalogs
* User profiles
* Configuration data

---

# Strategy 2: Manual Cache Deletion

Whenever data changes:

```text
Update Database
       ↓
Delete Cache
```

Example:

```js
await db.updateProduct(id)

await redis.del(
  `product:${id}`
)
```

---

# Advantages

* Immediate consistency
* Simple concept

---

# Disadvantages

* Requires discipline
* Easy to forget invalidation

---

# Best Use Cases

* Ecommerce products
* User settings
* Administrative updates

---

# Strategy 3: Write Through

Updates both:

```text
Database
```

and

```text
Cache
```

simultaneously.

---

Flow:

```text
Update Request
       ↓
Database
       ↓
Cache Update
```

---

# Advantages

* Cache always fresh

---

# Disadvantages

* Increased write latency

---

# Best Use Cases

* Frequently accessed records

---

# Strategy 4: Event-Based Invalidation

Common in microservices.

Architecture:

```text
Product Service
       ↓
Kafka Event
       ↓
Cache Service
       ↓
Cache Delete
```

---

Example Event

```json
{
  "event": "PRODUCT_UPDATED",
  "product_id": 101
}
```

---

Benefits

* Decoupled architecture
* Scalable invalidation

---

# Ecommerce Example

Product updated:

```text
Admin Changes Price
```

Flow:

```text
MySQL Update
      ↓
Kafka Event
      ↓
Cache Removal
```

Result:

Next request rebuilds cache.

---

# Strategy 5: Version-Based Invalidation

Instead of deleting:

Use version numbers.

---

Example:

```text
product:101:v1
```

After update:

```text
product:101:v2
```

Old cache becomes irrelevant.

---

# Advantages

* No explicit deletion
* Safe rollouts

---

# Disadvantages

* Increased memory usage

---

# Common Usage

* CDN assets
* Static content
* Large-scale caching

---

# Strategy 6: Refresh Ahead

Refresh data before expiration.

Architecture:

```text
Cache Entry
      ↓
Near Expiration
      ↓
Background Refresh
```

Benefits:

* Lower miss rate
* Better performance

---

# Cache Invalidation Flow

Standard approach:

```text
Read Request
      ↓
Redis
      ↓
Database (If Miss)
      ↓
Update Cache
```

Update Flow:

```text
Write Request
      ↓
Database
      ↓
Invalidate Cache
```

---

# Cache Rebuilding

After invalidation:

```text
First Request
      ↓
Database Query
      ↓
Cache Rebuild
```

Known as:

```text
Lazy Rebuilding
```

---

# Cache Stampede

One of the most important cache topics.

Problem:

Popular cache expires.

Example:

```text
Product Page
```

Traffic:

```text
1000 Requests/sec
```

Cache expires.

All requests hit database simultaneously.

---

Result:

```text
Database Overload
```

---

# Stampede Architecture

```text
Expired Cache
      ↓
Thousands Of Requests
      ↓
Database Spike
```

---

# Stampede Solution 1

Distributed Locking

Flow:

```text
First Request
      ↓
Acquire Lock
      ↓
Database Query
      ↓
Rebuild Cache
```

Other requests wait.

---

# Stampede Solution 2

Stale While Revalidate

Flow:

```text
Serve Old Cache
       ↓
Refresh In Background
```

Benefits:

* Low latency
* Better availability

---

# Stampede Solution 3

Refresh Ahead

Refresh before expiration occurs.

---

# Hot Key Problem

Certain keys receive enormous traffic.

Example:

```text
india-vs-pakistan
```

or

```text
iphone-product-page
```

---

Architecture:

```text
Millions Of Requests
         ↓
Single Redis Key
```

Can overload Redis.

---

# Hot Key Solutions

## Redis Replication

Distribute reads.

---

## Local Cache

Store frequently accessed values locally.

---

## CDN

Offload requests.

---

## Key Sharding

Split load across keys.

---

# Distributed Invalidation

Microservice Example:

```text
Catalog Service
      ↓
Kafka
      ↓
Inventory Service

Search Service

Recommendation Service
```

All caches updated consistently.

---

# Multi-Level Cache Invalidation

Architecture:

```text
Browser Cache
      ↓
CDN
      ↓
Redis
      ↓
Database
```

Each layer may require invalidation.

Challenges:

* Coordination
* Propagation delays

---

# Live Score Example

Score Update:

```text
145/4
```

New Score:

```text
151/4
```

Flow:

```text
Provider
 ↓
Score Service
 ↓
Redis Update
 ↓
WebSocket Broadcast
```

Freshness more important than TTL.

---

# Fantasy Sports Example

Leaderboard:

```text
User Rank Updated
```

Flow:

```text
Kafka Event
 ↓
Redis Sorted Set
 ↓
Leaderboard API
```

Immediate cache updates required.

---

# Ecommerce Example

Product Inventory:

```text
Stock = 10
```

Purchase:

```text
Stock = 9
```

Invalidate:

```text
product:inventory:101
```

Immediately.

---

# Monitoring Cache Invalidation

Track:

## Hit Ratio

Cache effectiveness.

---

## Miss Ratio

Potential invalidation issues.

---

## Rebuild Frequency

Cache churn.

---

## Invalidation Events

Operational visibility.

---

## Database Load

Detect cache failures.

---

# Common Mistakes

## No Invalidation Strategy

Creates stale data.

---

## Excessive TTLs

Data becomes outdated.

---

## Invalidating Too Frequently

Reduces cache effectiveness.

---

## Ignoring Stampedes

Can overload databases.

---

## No Monitoring

Problems remain hidden.

---

# Common Interview Questions

### What is cache invalidation?

Removing or updating stale cached data.

---

### Why is cache invalidation difficult?

Because performance and consistency often conflict.

---

### What is a cache stampede?

Many requests rebuilding an expired cache simultaneously.

---

### What is stale-while-revalidate?

Serving stale data while refreshing asynchronously.

---

### What is event-based invalidation?

Using events to invalidate caches across services.

---

### What are hot keys?

Keys receiving extremely high traffic.

---

# Production Lessons

* Cache invalidation is a critical architecture concern.
* TTL alone is rarely enough.
* Event-based invalidation scales well in microservices.
* Stampede protection is essential.
* Hot keys require special handling.
* Monitoring cache health is mandatory.
* Consistency requirements vary by business domain.

---

# Key Takeaways

* Caching without invalidation eventually creates stale data.
* TTL, manual deletion, and event-driven invalidation are the most common approaches.
* Cache stampedes can severely impact databases.
* Distributed systems often use Kafka-based invalidation.
* Hot keys require careful optimization.
* Effective invalidation strategies improve both performance and correctness.
* Cache invalidation remains one of the most important topics in backend architecture and system design interviews.

---

# Related Documents

* docs/redis/caching-patterns.md
* docs/redis/redis-patterns.md
* docs/architecture/caching-strategy.md
* docs/architecture/event-driven-architecture.md

Related Diagram:

* diagrams/redis-architecture.mmd
