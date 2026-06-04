# Live Score System Architecture

![Live Score](../../assets/live-score.png)

---

# Overview

Live score systems power real-time sports applications used by millions of fans worldwide.

Examples:

* Cricbuzz
* ESPN
* SofaScore
* Flashscore
* Fantasy Sports Platforms

Users expect:

* Real-time updates
* Low latency
* High availability
* Accurate score delivery

A delay of even a few seconds can significantly impact user experience, especially for fantasy sports, betting, and live match engagement.

This document covers the architecture of a production-grade live score platform.

---

# Business Requirements

A live score platform must provide:

### Real-Time Updates

Scores should appear within seconds.

---

### High Availability

Matches must remain available during traffic spikes.

---

### Scalability

Support millions of concurrent users.

---

### Accuracy

Score updates must be consistent and reliable.

---

### Low Latency

Fast propagation from score provider to users.

---

# High-Level Architecture

```text id="z8e4p7"
Data Provider
      ↓
Ingestion Service
      ↓
Kafka
      ↓
Score Processing Service
      ↓
Redis
      ↓
WebSocket Layer
      ↓
Users
```

Benefits:

* Real-time delivery
* Horizontal scalability
* Fault isolation

---

# Core Components

## Data Provider

External source of match data.

Examples:

* SportsRadar
* CricAPI
* Internal scoring systems

Provides:

* Ball-by-ball updates
* Player statistics
* Match status

---

## Ingestion Service

Responsibilities:

* Receive updates
* Validate payloads
* Normalize formats
* Publish events

Architecture:

```text id="ab6v1t"
Provider
   ↓
Ingestion
   ↓
Kafka
```

---

## Kafka

Acts as the event backbone.

Benefits:

* Durability
* Scalability
* Replay capability

Example Events:

```json id="0s3q6v"
{
  "match_id": 1001,
  "event": "FOUR",
  "runs": 4
}
```

---

## Score Processing Service

Responsibilities:

* Update match state
* Calculate statistics
* Generate derived events

Examples:

* Current score
* Run rate
* Partnerships
* Leaderboards

---

## Redis

Stores:

* Live scores
* Match state
* Player stats

Advantages:

* Low latency
* High throughput

Example:

```text id="p7m6qs"
match:1001
```

---

## WebSocket Layer

Responsible for:

* Real-time fan-out
* Match subscriptions
* Push delivery

Technologies:

* Socket.IO
* Native WebSockets

---

# Data Flow

Example:

```text id="6gn5ly"
Ball Delivered
      ↓
Provider Update
      ↓
Kafka Event
      ↓
Score Service
      ↓
Redis Update
      ↓
WebSocket Broadcast
      ↓
Users
```

Target latency:

```text id="5r0jtk"
< 1 Second
```

---

# Why Kafka?

Without Kafka:

```text id="8q9j1z"
Provider
   ↓
Score Service
```

Problems:

* Tight coupling
* Failure propagation

---

With Kafka:

```text id="mtrt6e"
Provider
   ↓
Kafka
   ↓
Many Consumers
```

Benefits:

* Scalability
* Reliability
* Event replay

---

# Redis Data Model

Example:

```json id="6v4fdk"
{
  "match_id": 1001,
  "score": "125/3",
  "overs": "15.2"
}
```

Key:

```text id="9xg0dw"
match:1001
```

TTL:

Optional depending on retention requirements.

---

# WebSocket Architecture

Traditional Polling:

```text id="q2s1y7"
Client
 ↓
API
 ↓
Database
```

Repeated every few seconds.

Problems:

* High load
* Increased latency

---

WebSockets:

```text id="hr1gr8"
Server
 ↓
Push Update
 ↓
Client
```

Benefits:

* Instant delivery
* Reduced traffic

---

# Match Subscription Model

Users subscribe to specific matches.

Example:

```text id="7t6l9r"
match:1001
```

Flow:

```text id="k0rx6v"
User
 ↓
Join Room
 ↓
Receive Match Updates
```

Benefits:

* Efficient broadcasts
* Lower bandwidth

---

# Horizontal Scaling

Challenge:

Millions of users.

Architecture:

```text id="t3s8jz"
Load Balancer
       ↓
WebSocket Servers
       ↓
Redis Pub/Sub
```

Redis synchronizes events across instances.

---

# Redis Pub/Sub

Flow:

```text id="x6k7mz"
Score Service
      ↓
Redis Pub/Sub
      ↓
Socket Servers
```

Benefits:

* Real-time synchronization
* Multi-server support

---

# Match State Management

Store:

```text id="b1d2wq"
Score

Overs

Wickets

Players

Commentary

Statistics
```

Redis provides:

* Fast reads
* Fast writes

---

# Commentary System

Architecture:

```text id="2w9p8m"
Provider
 ↓
Commentary Service
 ↓
Redis
 ↓
Users
```

Example:

```text id="v8y3fc"
FOUR!
Brilliant cover drive.
```

---

# Caching Strategy

Hot Matches:

```text id="y2w4e1"
India vs Australia
```

Millions of requests.

Store in Redis.

Benefits:

* Reduced DB load
* Faster responses

---

# Leaderboard Integration

Fantasy sports often consume score events.

Flow:

```text id="r4z6j0"
Score Event
      ↓
Kafka
      ↓
Fantasy Service
      ↓
Point Calculation
      ↓
Leaderboard Update
```

---

# Event Ordering

Critical requirement.

Example:

```text id="m5x8t1"
Ball 1

Ball 2

Ball 3
```

Must remain ordered.

Kafka partitions help maintain sequence.

---

# Failure Handling

Provider Failure:

```text id="q7n4e6"
No New Updates
```

Solution:

* Retry
* Secondary provider
* Alerting

---

Redis Failure:

```text id="g9t2r5"
Cache Unavailable
```

Solution:

* Redis replication
* Redis cluster

---

WebSocket Failure:

```text id="j6m8s4"
Reconnect
```

Client auto-reconnection.

---

# Multi-Region Architecture

Large platforms deploy globally.

```text id="f3p7v2"
Region A

Region B

Region C
```

Benefits:

* Lower latency
* Disaster recovery

---

# Monitoring Metrics

Track:

## Event Processing Latency

Provider to user delay.

---

## Kafka Lag

Consumer health.

---

## Redis Latency

Cache performance.

---

## Active Connections

WebSocket capacity.

---

## Message Delivery Rate

Real-time reliability.

---

# Security Considerations

## Authentication

Protect premium content.

---

## Rate Limiting

Prevent abuse.

---

## Access Control

Restrict internal feeds.

---

## Encryption

Secure data in transit.

---

# Cricket Example

Event:

```text id="h8v4t3"
Six Runs
```

Flow:

```text id="r2m7k5"
Provider
 ↓
Kafka
 ↓
Score Service
 ↓
Redis
 ↓
Socket.IO
 ↓
Users
```

Latency Goal:

```text id="n1q5z8"
< 500 ms
```

---

# Football Example

Event:

```text id="p4s7v6"
Goal
```

Flow identical.

Immediate broadcast to millions of users.

---

# Common Mistakes

## Polling Instead of WebSockets

Creates unnecessary load.

---

## No Event Backbone

Difficult scaling.

Use Kafka.

---

## No Caching

Creates database bottlenecks.

Use Redis.

---

## No Ordering Guarantees

Produces inconsistent scores.

---

## Single Provider Dependency

High availability risk.

---

# Common Interview Questions

### Why use Kafka in live score systems?

For reliable event distribution and scalability.

---

### Why use Redis?

For ultra-fast access to live match state.

---

### Why use WebSockets?

To push updates instantly to users.

---

### How do you scale WebSockets?

Multiple socket servers synchronized through Redis Pub/Sub.

---

### Why is event ordering important?

Sports events must appear in correct sequence.

---

### What happens if Redis fails?

Use replication, clustering, and failover strategies.

---

# Production Lessons

* Kafka is ideal for event distribution.
* Redis is essential for low-latency reads.
* WebSockets provide real-time fan-out.
* Event ordering must be preserved.
* Match subscriptions improve efficiency.
* Multi-region deployment improves availability.
* Observability is critical for real-time systems.

---

# Key Takeaways

* Live score systems are event-driven architectures.
* Kafka, Redis, and WebSockets form the core stack.
* Low latency and high availability are critical requirements.
* Horizontal scaling is mandatory for major sporting events.
* Event ordering and caching are fundamental design considerations.
* Live score systems power many modern sports and fantasy platforms.

---

# Related Documents

* docs/architecture/event-driven-architecture.md
* docs/architecture/distributed-systems.md
* docs/redis/pubsub.md
* docs/kafka/event-streaming.md

Related Diagram:

* diagrams/live-score-system.mmd
