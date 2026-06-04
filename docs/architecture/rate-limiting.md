# Rate Limiting System Design

![Architecture](../../assets/architecture.png)

---

# Overview

Rate limiting is a critical scalability and security mechanism used to control how frequently users, services, or clients can access resources.

Without rate limiting, systems become vulnerable to:

* Abuse
* Brute-force attacks
* DDoS attacks
* API misuse
* Resource exhaustion
* Excessive infrastructure costs

Every production-grade platform uses rate limiting.

Examples:

* Google APIs
* Stripe APIs
* GitHub APIs
* AWS APIs
* Payment Gateways
* Ecommerce Platforms

---

# What Is Rate Limiting?

Rate limiting controls:

```text
How Many Requests
Can Be Made
Within A Time Period
```

Example:

```text
100 Requests
Per Minute
```

If the limit is exceeded:

```http
429 Too Many Requests
```

Response:

```json
{
  "success": false,
  "message": "Rate limit exceeded"
}
```

---

# Why Rate Limiting Matters

Without protection:

```text
Single User
      ↓
100,000 Requests
      ↓
Database Overload
      ↓
Service Failure
```

Consequences:

* Increased latency
* Server crashes
* Higher cloud costs
* Poor user experience

---

# Common Use Cases

## Login Protection

Example:

```text
5 Login Attempts
Per Minute
```

Prevents:

* Brute-force attacks
* Credential stuffing

---

## Public APIs

Example:

```text
100 Requests
Per Minute
```

Protects infrastructure.

---

## Payment APIs

Example:

```text
10 Requests
Per Minute
```

Prevents duplicate transactions.

---

## OTP Verification

Example:

```text
3 OTP Requests
Per Hour
```

Prevents abuse.

---

## Search APIs

Example:

```text
50 Searches
Per Minute
```

Reduces unnecessary load.

---

# Rate Limiting Architecture

```text
Client
   ↓
API Gateway
   ↓
Rate Limiter
   ↓
Application
```

The rate limiter evaluates requests before they reach business logic.

Benefits:

* Lower resource consumption
* Better protection

---

# Identification Strategies

Requests may be limited by:

## IP Address

Example:

```text
192.168.1.10
```

Good for:

* Anonymous traffic

Limitations:

* Shared networks
* NAT environments

---

## User ID

Example:

```text
User #101
```

Best for authenticated users.

---

## API Key

Example:

```text
sk_live_xxxxx
```

Common for developer APIs.

---

## Tenant ID

Used in SaaS systems.

Prevents:

```text
One Tenant
Consuming All Resources
```

---

# Rate Limiting Algorithms

The most common interview topic.

---

# Fixed Window

Simplest approach.

Example:

```text
100 Requests
Per Minute
```

Implementation:

```text
Counter
+
Expiration
```

Flow:

```text
Request
   ↓
Increment Counter
   ↓
Check Limit
```

---

Advantages

* Easy implementation
* Low memory usage

---

Problems

Boundary issue.

Example:

```text
100 Requests
At 12:00:59

+

100 Requests
At 12:01:01
```

User effectively sends:

```text
200 Requests
In 2 Seconds
```

---

# Sliding Window

More accurate.

Architecture:

```text
Track Requests
Within Rolling Window
```

Example:

```text
Last 60 Seconds
```

Benefits:

* Better fairness
* More predictable

---

Challenges:

* More memory
* More computation

---

# Sliding Log

Store every request timestamp.

Example:

```text
10:00:01
10:00:02
10:00:05
```

Evaluate recent requests.

Benefits:

* Highest accuracy

Limitations:

* Memory intensive

---

# Token Bucket

Most popular production algorithm.

Architecture:

```text
Bucket
Filled With Tokens
```

Example:

```text
100 Tokens
```

Each request:

```text
Consumes Token
```

Tokens replenish over time.

---

Advantages

* Allows bursts
* Smooth traffic control

Widely used by:

* AWS
* Stripe
* API Gateways

---

# Leaky Bucket

Architecture:

```text
Incoming Requests
        ↓
Bucket
        ↓
Constant Outflow
```

Benefits:

* Stable traffic
* Smooth request rate

Useful for:

* Messaging systems
* Payment systems

---

# Redis-Based Rate Limiting

Most common production implementation.

Architecture:

```text
Client
   ↓
API
   ↓
Redis
   ↓
Counter
```

Example Key:

```text
rate:user:101
```

Value:

```text
45
```

TTL:

```text
60 Seconds
```

---

# Fixed Window Redis Example

Flow:

```text
Request
   ↓
INCR
   ↓
EXPIRE
   ↓
Check Limit
```

Benefits:

* Fast
* Distributed
* Easy to scale

---

# Token Bucket With Redis

Store:

```text
Available Tokens
```

Per User.

Each request:

```text
Token -= 1
```

Background process:

```text
Refill Tokens
```

Benefits:

* Fair traffic management
* Distributed architecture

---

# Distributed Rate Limiting

Single server solution fails in multi-instance systems.

Problem:

```text
API Server 1
Counter = 50

API Server 2
Counter = 50
```

Actual:

```text
100 Requests
```

But neither server knows.

---

Solution:

Centralized Redis.

```text
Server 1
      ↓
Redis

Server 2
      ↓
Redis

Server 3
      ↓
Redis
```

Single source of truth.

---

# API Gateway Rate Limiting

Large systems implement limits at gateway level.

Architecture:

```text
Client
   ↓
NGINX
Kong
AWS API Gateway
   ↓
Services
```

Benefits:

* Centralized enforcement
* Reduced service complexity

---

# Ecommerce Example

Product APIs:

```text
200 Requests
Per Minute
```

Cart APIs:

```text
50 Requests
Per Minute
```

Checkout APIs:

```text
10 Requests
Per Minute
```

Benefits:

* Prevent abuse
* Reduce fraud

---

# Payment System Example

Payment Creation:

```text
5 Requests
Per Minute
```

Why?

Prevent:

* Duplicate charges
* Abuse
* Accidental retries

Combine with:

```text
Idempotency Keys
```

---

# Authentication Example

Login Endpoint:

```text
5 Attempts
Per Minute
```

After exceeding:

```text
Temporary Lockout
```

Benefits:

* Reduced brute-force attacks

---

# Fantasy Sports Example

Contest Joining:

```text
20 Requests
Per Minute
```

Prevents:

* Bot activity
* Automated abuse

---

# Monitoring Metrics

Track:

## Allowed Requests

Successful traffic.

---

## Rejected Requests

Rate limit violations.

---

## Redis Latency

Rate limiter performance.

---

## Top Consumers

Most active users.

---

## API Usage Patterns

Traffic analysis.

---

# Rate Limit Headers

Helpful API design practice.

Example:

```http
X-RateLimit-Limit: 100

X-RateLimit-Remaining: 20

X-RateLimit-Reset: 1710000000
```

Benefits:

* Better developer experience

---

# Common Mistakes

## In-Memory Counters

Bad:

```text
Server Memory
```

Problem:

Doesn't work across multiple instances.

Use Redis.

---

## Missing Expiration

Counter grows forever.

Always set TTL.

---

## Overly Strict Limits

Can block legitimate users.

---

## No Monitoring

Rate limit failures go unnoticed.

---

## No Differentiation

Different endpoints require different limits.

---

# Common Interview Questions

### What is rate limiting?

Controlling how many requests a client can make within a time period.

---

### Why use Redis for rate limiting?

Redis provides fast distributed counters and expiration support.

---

### Difference between Fixed Window and Sliding Window?

Fixed Window is simpler.

Sliding Window is more accurate.

---

### What is Token Bucket?

A rate-limiting algorithm that allows bursts while maintaining long-term limits.

---

### Why implement rate limiting at API Gateway level?

Centralized control and reduced service complexity.

---

### How do distributed systems implement rate limiting?

Using a shared store such as Redis.

---

# Production Lessons

* Every public API should have rate limits.
* Redis is the most common implementation.
* Token Bucket is often the preferred algorithm.
* Authentication endpoints require stricter limits.
* Gateway-level enforcement simplifies architecture.
* Monitoring is essential.
* Rate limiting protects both infrastructure and business operations.

---

# Key Takeaways

* Rate limiting is both a scalability and security feature.
* Redis enables distributed rate limiting.
* Fixed Window, Sliding Window, Token Bucket, and Leaky Bucket are the primary algorithms.
* Different endpoints require different policies.
* API Gateways commonly enforce limits.
* Proper rate limiting improves reliability and reduces abuse.
* Production-grade systems always include rate-limiting strategies.

---

# Related Documents

* docs/architecture/api-design.md
* docs/architecture/authentication-design.md
* docs/architecture/caching-strategy.md
* docs/redis/redis-patterns.md

Related Diagram:

* diagrams/rate-limiting.mmd
