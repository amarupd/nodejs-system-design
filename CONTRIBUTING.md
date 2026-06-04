# Contributing Guide

Thank you for your interest in contributing to **nodejs-system-design**.

This repository is designed as a production-grade learning resource focused on:

* Node.js internals
* Backend architecture
* Distributed systems
* Scalability patterns
* Event-driven systems
* Redis
* RabbitMQ
* Kafka
* AWS
* Kubernetes
* Real-world system design case studies

The goal is to maintain high-quality, architecture-focused content that reflects senior backend engineering practices used in modern production systems.

---

# Repository Principles

All contributions should follow these principles:

### Architecture First

Content should explain:

* Why a system is designed a certain way
* Tradeoffs involved
* Scaling limitations
* Production considerations

Avoid purely theoretical explanations.

---

### Production Grade

Every document should consider:

* Reliability
* Scalability
* Availability
* Performance
* Security
* Observability

Production concerns are more important than academic explanations.

---

### Vendor Neutral

Examples may use:

* AWS
* GCP
* Azure

But concepts should remain platform independent whenever possible.

---

### Real World Focus

Good contributions include:

* Actual engineering challenges
* Failure scenarios
* Incident analysis
* Bottlenecks
* Scaling decisions

---

# Repository Structure

## Fundamentals

Located in:

docs/fundamentals/

Topics include:

* Node.js Runtime
* Event Loop
* Streams
* Worker Threads
* Clustering
* Memory Management

---

## Architecture

Located in:

docs/architecture/

Topics include:

* API Design
* Authentication
* Authorization
* Rate Limiting
* Caching
* Notification Systems
* Payment Systems
* Distributed Systems
* Event Driven Architecture

---

## Redis

Located in:

docs/redis/

Topics include:

* Caching
* Pub/Sub
* Distributed Locks
* Cache Invalidation

---

## RabbitMQ

Located in:

docs/rabbitmq/

Topics include:

* Queues
* Retries
* DLQs
* Ordering
* Reliability

---

## Kafka

Located in:

docs/kafka/

Topics include:

* Partitions
* Consumer Groups
* Event Streaming
* Exactly Once Processing

---

## Case Studies

Located in:

docs/case-studies/

Topics include:

* Ecommerce
* Fantasy Sports
* Payments
* Notifications

---

## Diagrams

Located in:

diagrams/

Mermaid diagrams should be saved as:

.mmd files

Do not wrap content in markdown code blocks.

Example:

graph TD
A[Client] --> B[API Gateway]
B --> C[Service]

---

# Writing Standards

Every document should include:

## Problem Statement

What problem are we solving?

---

## Architecture Overview

High-level system design.

---

## Core Components

Detailed explanation of major services.

---

## Data Flow

How requests move through the system.

---

## Scaling Strategy

How the architecture scales.

---

## Failure Scenarios

Potential failures and mitigation techniques.

---

## Tradeoffs

Benefits and limitations of the approach.

---

## Production Considerations

Monitoring, security, deployments, and operations.

---

# Diagram Standards

All architecture documents should have matching Mermaid diagrams when applicable.

Example location:

diagrams/payment-system.mmd

Referenced from:

docs/architecture/payment-system.md

---

# Image Standards

Architecture screenshots and diagrams should use assets from:

assets/

Examples:

![Architecture](assets/architecture.png)

![Redis](assets/redis.png)

![Kafka](assets/kafka.png)

![RabbitMQ](assets/rabbitmq.png)

Every screenshot document should reference its corresponding image asset.

---

# Pull Request Guidelines

Before submitting a pull request:

* Verify technical accuracy
* Check grammar and formatting
* Maintain architecture-first writing style
* Include diagrams where appropriate
* Include production considerations
* Include scaling discussion
* Include tradeoff analysis

---

# What We Avoid

The repository should not contain:

* Proprietary company code
* Internal company architecture
* Sensitive production data
* API keys
* Secrets
* Customer information

Only educational and portfolio-quality content should be published.

---

# Recommended Contribution Areas

Examples of valuable contributions:

* New architecture case studies
* Distributed systems concepts
* Scaling patterns
* Redis strategies
* Kafka architecture
* RabbitMQ reliability patterns
* Observability techniques
* Production incident learnings
* System design interview material

---

# Questions

If you find an issue or would like to improve a document:

1. Open an issue
2. Describe the problem
3. Propose a solution
4. Submit a pull request

---

Thank you for helping improve nodejs-system-design.

Together we can build a world-class backend architecture and system design repository for the engineering community.
