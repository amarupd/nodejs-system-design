# Fantasy Sports System Architecture

![Fantasy Sports](../../assets/fantasy-sports.png)

---

# Overview

Fantasy sports platforms are among the most complex real-time consumer applications.

Examples:

* Dream11
* My11Circle
* ESPN Fantasy
* DraftKings
* FanDuel

These systems combine:

* Real-time scoring
* Wallet systems
* Payments
* Contest management
* Leaderboards
* Massive concurrency
* Event-driven processing

A production-grade fantasy sports platform must support:

* Millions of users
* Hundreds of thousands of concurrent requests
* Real-time score updates
* Financial transactions
* Fair and accurate point calculations

This document covers the architecture of a large-scale fantasy sports platform.

---

# Business Requirements

A fantasy platform must support:

### User Management

Registration and authentication.

---

### Team Creation

Users create fantasy teams.

---

### Contest Participation

Join paid or free contests.

---

### Real-Time Scoring

Live points based on match events.

---

### Wallet Management

Deposits, winnings, refunds.

---

### Leaderboards

Real-time rankings.

---

### Payments

Deposits and withdrawals.

---

# High-Level Architecture

```text id="b3v7w2"
Users
   ↓
API Gateway
   ↓
Core Services
   ↓
Kafka
   ↓
Workers
   ↓
Redis + Databases
```

Core Services:

* User Service
* Match Service
* Contest Service
* Team Service
* Wallet Service
* Payment Service
* Notification Service
* Leaderboard Service

---

# Core Microservices

## User Service

Responsibilities:

* Registration
* Login
* KYC
* Profile Management

Database:

```text id="w7m2q8"
MySQL
```

---

## Match Service

Stores:

* Fixtures
* Match Details
* Teams
* Venues

Data Sources:

```text id="x1k4p6"
Sports Data Provider
```

---

## Team Service

Handles:

* Team creation
* Captain selection
* Vice-captain selection
* Validation rules

Example:

```text id="r9d5s2"
11 Players
```

Constraints enforced here.

---

## Contest Service

Manages:

* Contest creation
* Entry fees
* Prize pools
* Participation

States:

```text id="f6v1m9"
Upcoming

Live

Completed

Cancelled
```

---

## Wallet Service

Tracks:

* Deposits
* Winnings
* Bonus cash
* Withdrawals

Requires:

* Ledger system
* Auditability

---

## Payment Service

Integrates with:

* Razorpay
* Stripe
* Cashfree

Responsibilities:

* Deposits
* Refunds
* Withdrawals

---

# Match Data Pipeline

Fantasy platforms depend on live sports feeds.

Architecture:

```text id="m8z4w7"
Provider
   ↓
Ingestion Service
   ↓
Kafka
   ↓
Scoring Service
```

Events:

* Runs
* Wickets
* Boundaries
* Goals
* Assists

---

# Real-Time Scoring Architecture

Flow:

```text id="q2j7k5"
Match Event
      ↓
Kafka
      ↓
Scoring Service
      ↓
Player Points
      ↓
Redis
      ↓
Leaderboard Update
```

Latency target:

```text id="s9c4n8"
< 2 Seconds
```

---

# Player Point Calculation

Example:

```text id="h4p7x3"
Run = +1

Four = +4

Six = +6

Wicket = +25
```

Flow:

```text id="v8m2d1"
Event
 ↓
Rule Engine
 ↓
Point Calculation
```

Rules configurable through admin systems.

---

# Leaderboard Architecture

Critical component.

Requirements:

* Real-time updates
* Fast ranking
* Millions of entries

---

# Redis Sorted Sets

Industry-standard approach.

Key:

```text id="y3k6q1"
contest:1001
```

Value:

```text id="e7j4r8"
user_score
```

Benefits:

* Fast ranking
* O(log n) updates

---

# Leaderboard Flow

```text id="p5d8v2"
Score Event
      ↓
Redis Sorted Set
      ↓
Leaderboard API
      ↓
Users
```

---

# Contest Joining Flow

```text id="a2m9q5"
User
 ↓
Contest Selection
 ↓
Wallet Check
 ↓
Team Validation
 ↓
Contest Entry
 ↓
Confirmation
```

---

# Concurrency Challenge

Popular contests may receive:

```text id="g6n3x9"
100,000+
Join Requests
```

Within minutes.

---

# Solution

Use:

```text id="u7v2c4"
Redis Locking
```

Flow:

```text id="r1k8p7"
Acquire Lock
 ↓
Validate Capacity
 ↓
Reserve Slot
 ↓
Release Lock
```

Prevents overbooking.

---

# Contest Capacity Control

Example:

```text id="n4p1q8"
Contest Size

10,000 Users
```

Must never exceed limit.

Redis counters commonly used.

---

# Wallet Architecture

Flow:

```text id="b8w4r6"
Deposit
 ↓
Ledger
 ↓
Wallet Balance
```

Contest Entry:

```text id="q7t3m1"
Wallet Debit
 ↓
Contest Join
```

Winnings:

```text id="x5v8k2"
Contest Settlement
 ↓
Wallet Credit
```

---

# Settlement Architecture

After match completion:

```text id="m3q9j6"
Match Ends
      ↓
Kafka Event
      ↓
Settlement Service
      ↓
Leaderboard Finalization
      ↓
Prize Distribution
```

---

# Prize Distribution

Example:

```text id="v1r7p5"
Rank 1

₹100,000
```

Flow:

```text id="j8n4c3"
Settlement
 ↓
Wallet Credits
 ↓
Notifications
```

Must be idempotent.

---

# Real-Time User Experience

Users expect:

* Instant score updates
* Live rankings
* Contest status changes

Architecture:

```text id="f2x8v4"
Redis
 ↓
Socket.IO
 ↓
Users
```

---

# Notification Architecture

Events:

```text id="w6p3m7"
Contest Joined

Match Started

Winnings Credited
```

Flow:

```text id="c9k5r2"
Kafka
 ↓
Notification Service
```

Channels:

* Push
* Email
* SMS
* In-App

---

# Scalability Architecture

```text id="z4t7q1"
Load Balancer
       ↓
Kubernetes
       ↓
Node.js Services
       ↓
Redis
       ↓
Kafka
       ↓
MySQL
```

Benefits:

* Horizontal scaling
* Fault tolerance

---

# Database Strategy

MySQL stores:

```text id="l8n2v5"
Users

Teams

Contests

Transactions
```

Redis stores:

```text id="d1q7k9"
Leaderboards

Live Scores

Sessions

Locks
```

Kafka handles:

```text id="h5m8p2"
Events
```

---

# Security Requirements

## KYC

Required for withdrawals.

---

## Fraud Detection

Monitor:

* Multiple accounts
* Device abuse
* Suspicious activity

---

## Rate Limiting

Protect APIs.

---

## Audit Logs

Track financial actions.

---

# High-Traffic Match Example

India vs Pakistan

Potential:

```text id="p8k4v1"
Millions Of Users
```

Traffic:

```text id="t7m3q9"
Login

Team Creation

Contest Join

Leaderboard Refresh
```

Architecture must scale automatically.

---

# Monitoring Metrics

Track:

## Active Users

Real-time engagement.

---

## Contest Join Rate

Traffic indicator.

---

## Kafka Lag

Processing health.

---

## Leaderboard Latency

User experience metric.

---

## Wallet Transactions

Financial accuracy.

---

# Common Mistakes

## Calculating Points Synchronously

Creates bottlenecks.

Use Kafka workers.

---

## Storing Leaderboards In MySQL

Poor performance.

Use Redis Sorted Sets.

---

## No Idempotency

Causes duplicate rewards.

---

## Weak Settlement Design

Financial inconsistencies.

---

## No Concurrency Controls

Contest overbooking risk.

---

# Common Interview Questions

### Why use Kafka?

For reliable event-driven score processing.

---

### Why use Redis Sorted Sets?

For efficient leaderboard ranking.

---

### How do you prevent contest overbooking?

Using Redis locks and atomic counters.

---

### How do you calculate fantasy points?

Through a configurable scoring engine consuming match events.

---

### Why is settlement difficult?

Because financial accuracy is mandatory.

---

### How do you scale live leaderboards?

Redis + WebSockets + horizontal scaling.

---

# Production Lessons

* Fantasy sports platforms are event-driven systems.
* Kafka enables scalable score processing.
* Redis is critical for leaderboards and live data.
* Settlement must be idempotent.
* Wallet systems require ledger-based accounting.
* Real-time updates drive user engagement.
* Scalability planning is essential for major sporting events.

---

# Key Takeaways

* Fantasy sports combines real-time systems and financial systems.
* Kafka, Redis, MySQL, and WebSockets form the core architecture.
* Leaderboards require specialized data structures.
* Contest participation introduces concurrency challenges.
* Settlement and wallet systems require financial-grade reliability.
* Event-driven architecture enables massive scalability.
* Fantasy sports platforms are excellent system design interview case studies.

---

# Related Documents

* docs/architecture/live-score-system.md
* docs/architecture/payment-system.md
* docs/architecture/notification-system.md
* docs/kafka/event-streaming.md
* docs/redis/redis-patterns.md

Related Diagram:

* diagrams/fantasy-sports.mmd
