# System Architecture Overview

![Architecture](../../assets/architecture.png)

---

# Overview

This repository focuses on production-grade backend architecture patterns used in modern scalable systems.

The diagrams and visual ../../assets included throughout this repository are designed to help engineers understand how large-scale platforms are built, scaled, monitored, and operated.

---

# Architecture Domains Covered

## Node.js Runtime

![Node.js Runtime](../../assets/nodejs-runtime.png)

Topics:

- Event Loop
- Worker Threads
- Streams
- Clustering
- Memory Management

Related Documents:

- docs/fundamentals/nodejs-runtime.md
- docs/fundamentals/event-loop.md
- docs/fundamentals/worker-threads.md

---

## Distributed Systems

![Architecture](../../assets/architecture.png)

Topics:

- Event Driven Architecture
- Distributed Systems
- Service Communication
- Scalability Patterns

Related Documents:

- docs/architecture/distributed-systems.md
- docs/architecture/event-driven-architecture.md

---

## Redis Architecture

![Redis](../../assets/redis.png)

Topics:

- Caching
- Distributed Locking
- Pub/Sub
- Cache Invalidation

Related Documents:

- docs/redis/redis-patterns.md
- docs/redis/distributed-locking.md

---

## RabbitMQ Architecture

![RabbitMQ](../../assets/rabbitmq.png)

Topics:

- Queue Patterns
- Retries
- Dead Letter Queues
- Message Ordering

Related Documents:

- docs/rabbitmq/rabbitmq-patterns.md
- docs/rabbitmq/retries.md

---

## Kafka Architecture

![Kafka](../../assets/kafka.png)

Topics:

- Event Streaming
- Consumer Groups
- Partitions
- Exactly Once Processing

Related Documents:

- docs/kafka/kafka-patterns.md
- docs/kafka/event-streaming.md

---

## Ecommerce Architecture

![Ecommerce](../../assets/ecommerce.png)

Topics:

- Catalog Systems
- Inventory Management
- Order Processing
- Checkout Flows

Related Documents:

- docs/architecture/ecommerce-architecture.md
- docs/case-studies/ecommerce-case-study.md

---

## Fantasy Sports Architecture

![Fantasy Sports](../../assets/fantasy-sports.png)

Topics:

- Live Scores
- Real-Time Leaderboards
- Contest Systems
- Wallet Systems

Related Documents:

- docs/architecture/fantasy-sports-architecture.md
- docs/case-studies/fantasy-sports-case-study.md

---

## Payment Systems

![Payment System](../../assets/payment-system.png)

Topics:

- Payment Processing
- Ledger Systems
- Reconciliation
- Idempotency

Related Documents:

- docs/architecture/payment-system.md
- docs/case-studies/payment-case-study.md

---

## Live Score Systems

![Live Score](../../assets/live-score.png)

Topics:

- Kafka Streaming
- Redis
- Socket.IO
- Real-Time Updates

Related Documents:

- docs/architecture/live-score-system.md

---

## Deployment Architecture

![Deployment](../../assets/deployment.png)

Topics:

- AWS
- Docker
- Kubernetes
- Auto Scaling

Related Documents:

- diagrams/deployment-architecture.mmd

---

# Repository Learning Path

Recommended order:

1. Fundamentals
2. Redis
3. RabbitMQ
4. Kafka
5. Architecture Modules
6. Case Studies
7. Engineering Lessons
8. Diagrams

---

# Target Audience

This repository is intended for:

- Backend Engineers
- Software Engineers
- Full Stack Developers
- System Design Interview Preparation
- Engineering Managers
- Technical Architects

---

# Related Diagrams

- diagrams/nodejs-runtime.mmd
- diagrams/event-loop.mmd
- diagrams/redis-architecture.mmd
- diagrams/rabbitmq-architecture.mmd
- diagrams/kafka-architecture.mmd
- diagrams/ecommerce-system.mmd
- diagrams/payment-system.mmd
- diagrams/fantasy-sports.mmd
- diagrams/deployment-architecture.mmd