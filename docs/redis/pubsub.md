# Redis Pub/Sub Architecture

![Redis](../../assets/redis.png)

---

# Overview

Redis Pub/Sub (Publish/Subscribe) is a lightweight messaging pattern that enables real-time communication between services.

It allows applications to:

* Broadcast events
* Deliver real-time updates
* Synchronize services
* Push notifications
* Power WebSocket infrastructures

Redis Pub/Sub is commonly used in:

* Live Score Systems
* Chat Applications
* Notification Systems
* Multiplayer Games
* WebSocket Clusters
* Real-Time Dashboards

While Kafka and RabbitMQ are often used for durable messaging, Redis Pub/Sub excels at low-latency event delivery.

---

# What Is Publish/Subscribe?

Publish/Subscribe is a messaging pattern where:

### Publisher

Sends messages.

---

### Subscriber

Receives messages.

---

### Channel

Communication medium.

---

Architecture:

```text id="n83x1s"
Publisher
    ↓
Redis Channel
    ↓
Subscribers
```

Publishers and subscribers do not know about each other.

---

# Why Pub/Sub Matters

Without Pub/Sub:

```text id="n0u3dy"
Service A
   ↓
Service B
   ↓
Service C
```

Problems:

* Tight coupling
* Difficult scaling

---

With Pub/Sub:

```text id="9k4r6u"
Service A
   ↓
Redis
   ↓
Many Consumers
```

Benefits:

* Decoupling
* Scalability
* Simplicity

---

# Core Components

## Publisher

Creates messages.

Example:

```text id="z0j9pq"
Score Service
```

Publishes:

```text id="q3u7mk"
MATCH_UPDATED
```

---

## Channel

Message topic.

Examples:

```text id="t7x1nr"
live-score

notifications

chat-room-101
```

---

## Subscriber

Consumes messages.

Examples:

```text id="f1r8jd"
Socket Server

Notification Service

Analytics Service
```

---

# Basic Architecture

```text id="y9c4kt"
Publisher
    ↓
Redis Channel
    ↓
Subscriber A

Subscriber B

Subscriber C
```

One message reaches all subscribers.

---

# Node.js Publisher Example

```js
await redis.publish(
  "live-score",
  JSON.stringify({
    matchId: 101,
    score: "145/4"
  })
)
```

Publishes to:

```text id="k8w6my"
live-score
```

channel.

---

# Node.js Subscriber Example

```js
subscriber.subscribe(
  "live-score"
)

subscriber.on(
  "message",
  (channel, message) => {
    console.log(message)
  }
)
```

Receives all published messages.

---

# Real-Time Score Architecture

One of the most common Redis Pub/Sub use cases.

Architecture:

```text id="j2p8q5"
Score Service
      ↓
Redis Pub/Sub
      ↓
Socket Servers
      ↓
Users
```

Benefits:

* Near real-time updates
* Low latency

---

# Live Cricket Example

Event:

```text id="c7w2m1"
FOUR
```

Flow:

```text id="h8r5p4"
Scoring Engine
       ↓
Redis Channel
       ↓
Socket Servers
       ↓
Millions Of Users
```

Latency:

```text id="s1v7q8"
Milliseconds
```

---

# WebSocket Scaling

Problem:

Multiple WebSocket servers.

Architecture:

```text id="n4m9x6"
Socket Server A

Socket Server B

Socket Server C
```

Without synchronization:

Updates become inconsistent.

---

Solution:

```text id="g3t6k2"
Redis Pub/Sub
```

Flow:

```text id="a9q4r7"
Publisher
 ↓
Redis
 ↓
All Socket Servers
```

Benefits:

* Consistent broadcasts
* Horizontal scaling

---

# Chat Application Example

Architecture:

```text id="m7p2v5"
User A
 ↓
Chat Service
 ↓
Redis Pub/Sub
 ↓
User B
```

Channel:

```text id="u5j8c1"
chat:room:101
```

Benefits:

* Real-time messaging

---

# Notification Broadcasting

Architecture:

```text id="b6r3n8"
Notification Service
        ↓
Redis Pub/Sub
        ↓
Workers
```

Events:

```text id="p2m7w4"
ORDER_CREATED

PAYMENT_SUCCESS

MATCH_STARTED
```

---

# Presence Systems

Store:

```text id="y1v4q9"
online:user:101
```

Publish:

```text id="l7p5k2"
USER_ONLINE
```

Benefits:

* Real-time status updates

---

# Multiplayer Gaming Example

Events:

```text id="x8t1m6"
Player Joined

Player Moved

Player Left
```

Flow:

```text id="r4v7p3"
Game Server
      ↓
Redis Pub/Sub
      ↓
Players
```

Benefits:

* Fast state propagation

---

# Notification Service Example

Architecture:

```text id="c3k8n1"
Order Service
      ↓
Redis Pub/Sub
      ↓
Email Worker

SMS Worker

Push Worker
```

Benefits:

* Decoupled delivery

---

# Redis Pub/Sub Limitations

Important interview topic.

Redis Pub/Sub is:

```text id="j6m3w8"
Transient Messaging
```

Messages are not stored.

---

If subscriber is offline:

```text id="f5q9r2"
Message Lost
```

This is expected behavior.

---

# Pub/Sub vs Kafka

Redis Pub/Sub:

```text id="q2v6m9"
Fast

Simple

No Persistence
```

---

Kafka:

```text id="e4t8p1"
Persistent

Replayable

Scalable
```

---

Use Redis when:

```text id="w7n3k5"
Real-Time Delivery
```

matters most.

---

Use Kafka when:

```text id="b8r1p4"
Durability
```

matters most.

---

# Pub/Sub vs RabbitMQ

Redis Pub/Sub:

```text id="n5m7q3"
Broadcast Messaging
```

---

RabbitMQ:

```text id="u9t2k8"
Reliable Queues
```

---

RabbitMQ supports:

* Acknowledgements
* Retries
* DLQs

Redis Pub/Sub does not.

---

# Redis Streams Alternative

For reliability:

Use:

```text id="v4r8m1"
Redis Streams
```

Benefits:

* Persistence
* Consumer Groups
* Replay

Architecture:

```text id="h2q5p9"
Producer
 ↓
Redis Stream
 ↓
Consumers
```

---

# Monitoring Pub/Sub Systems

Track:

## Published Messages

Traffic volume.

---

## Subscriber Count

Consumer health.

---

## Delivery Latency

Performance.

---

## Connection Count

System load.

---

## Redis CPU Usage

Infrastructure health.

---

# Security Considerations

## Authentication

Restrict Redis access.

---

## Encryption

Protect data in transit.

---

## Network Isolation

Private networks preferred.

---

## Channel Design

Avoid exposing sensitive data.

---

# Ecommerce Example

Events:

```text id="y3m6r8"
PRODUCT_UPDATED

INVENTORY_CHANGED

ORDER_CREATED
```

Flow:

```text id="s1t7q4"
Service
 ↓
Redis
 ↓
Consumers
```

---

# Fantasy Sports Example

Events:

```text id="k8v2p5"
POINTS_UPDATED

LEADERBOARD_UPDATED
```

Flow:

```text id="f4r9n1"
Scoring Service
       ↓
Redis Pub/Sub
       ↓
Socket Servers
```

---

# Live Score Example

Events:

```text id="x7m5q3"
RUN

FOUR

SIX

WICKET
```

Flow:

```text id="g2p8r6"
Scoring Service
       ↓
Redis
       ↓
Socket.IO
```

---

# Common Mistakes

## Using Pub/Sub For Durable Messaging

Messages can be lost.

---

## No Reconnection Logic

Subscribers may disconnect.

---

## Massive Payloads

Increase latency.

---

## Exposing Redis Publicly

Security risk.

---

## Ignoring Monitoring

Operational risk.

---

# Common Interview Questions

### What is Redis Pub/Sub?

A publish-subscribe messaging system for real-time communication.

---

### Are messages persisted?

No.

Messages are transient.

---

### What happens if a subscriber is offline?

Messages are lost.

---

### When should you use Redis Pub/Sub?

Real-time event broadcasting.

---

### Why use Redis Pub/Sub with WebSockets?

To synchronize updates across multiple socket servers.

---

### What is a better alternative when persistence is required?

Kafka, RabbitMQ, or Redis Streams.

---

# Production Lessons

* Redis Pub/Sub is excellent for low-latency messaging.
* It is widely used for WebSocket scaling.
* Messages are not durable.
* Redis Streams should be considered for reliable delivery.
* Monitoring connection counts is important.
* Pub/Sub simplifies real-time architectures.
* It remains a core building block for live systems.

---

# Key Takeaways

* Redis Pub/Sub enables lightweight event broadcasting.
* Publishers and subscribers are decoupled.
* Messages are delivered in real time but are not persisted.
* It is heavily used in live score, chat, and notification systems.
* WebSocket clusters commonly rely on Redis Pub/Sub.
* For durable messaging, Kafka or Redis Streams are better choices.
* Understanding Pub/Sub is important for modern backend architecture and system design interviews.

---

# Related Documents

* docs/redis/redis-patterns.md
* docs/architecture/live-score-system.md
* docs/architecture/notification-system.md
* docs/kafka/event-streaming.md

Related Diagram:

* diagrams/redis-architecture.mmd
