# Payment System Architecture

![Payment System](../../assets/payment-system.png)

![Architecture](../../assets/architecture.png)

---

# Overview

This document showcases the architecture of a production-grade payment platform designed to process secure, scalable, reliable, and auditable financial transactions.

Payment systems are among the most critical backend systems because they require:

* Financial Correctness
* High Availability
* Strong Security
* Fraud Prevention
* Regulatory Compliance
* Auditability
* Fault Tolerance

Even small mistakes in payment workflows can result in revenue loss, compliance violations, and customer trust issues.

This architecture demonstrates modern payment platform design principles used by fintech, ecommerce, fantasy sports, and digital wallet platforms.

---

# Business Capabilities

## Customer Features

* Payment Initiation
* Payment Verification
* Refund Requests
* Transaction History
* Saved Payment Methods

---

## Merchant Features

* Settlement Tracking
* Refund Management
* Payment Reports
* Reconciliation

---

## Platform Features

* Multi Gateway Support
* Webhook Processing
* Fraud Detection
* Audit Logging
* Retry Handling
* Event Streaming

---

# High-Level Architecture

![Payment System](../../assets/payment-system.png)

```text
Client
 ↓
API Gateway
 ↓
Payment Service
 ↓
Gateway Adapter
 ↓
Payment Gateway
 ↓
Banking Network
```

Supporting Services:

```text
Ledger Service

Settlement Service

Fraud Service

Notification Service

Reporting Service
```

---

# Core Services

```text
Payment Service

Ledger Service

Settlement Service

Fraud Service

Notification Service

Reporting Service
```

Benefits:

* Independent Scaling
* Better Reliability
* Clear Domain Ownership

---

# Payment Service

Responsibilities:

* Payment Creation
* Payment Verification
* Status Tracking
* Gateway Communication
* Refund Management

Primary Storage:

```text
MySQL
```

Benefits:

* Strong Consistency
* Reliable Transaction Tracking

---

# Payment Lifecycle

Typical Workflow:

```text
PAYMENT_INITIATED
        ↓
PAYMENT_PENDING
        ↓
PAYMENT_SUCCESS
```

Failure Scenario:

```text
PAYMENT_INITIATED
        ↓
PAYMENT_PENDING
        ↓
PAYMENT_FAILED
```

Every state transition must be persisted and auditable.

---

# Gateway Integration Layer

Supported Providers:

```text
Stripe

Razorpay

PayU

PayPal
```

Architecture:

```text
Payment Service
      ↓
Gateway Adapter
      ↓
Provider
```

Benefits:

* Vendor Abstraction
* Easier Migration
* Multi-Gateway Support

---

# Idempotency Architecture

Critical Requirement:

```text
Never Process A Payment Twice
```

Implementation:

```text
Idempotency Key
```

Flow:

```text
Request
 ↓
Check Key
 ↓
Already Processed?
 ↓
Return Existing Result
```

Benefits:

* Duplicate Protection
* Financial Correctness

---

# Payment Retry Handling

Failures:

```text
Gateway Timeout

Network Failure

Temporary Error
```

Flow:

```text
Failure
 ↓
Retry Queue
 ↓
Reprocess
```

Benefits:

* Increased Success Rate
* Better Reliability

---

# Ledger Architecture

![Architecture](../../assets/architecture.png)

Golden Rule:

```text
Never Update Balance Directly
```

Instead:

```text
Credit Entry

Debit Entry
```

Balance is derived from transaction history.

Benefits:

* Auditability
* Financial Correctness
* Regulatory Compliance

---

# Double Entry Accounting

Example:

```text
User Wallet +100

Platform Liability +100
```

Every transaction generates balancing entries.

Benefits:

* Accurate Accounting
* Easier Auditing

---

# Wallet Transaction Flow

```text
Payment Success
      ↓
Ledger Entry
      ↓
Wallet Update
      ↓
Notification
```

Benefits:

* Traceability
* Consistency

---

# Webhook Processing

Payment gateways communicate asynchronously using webhooks.

Architecture:

```text
Gateway
 ↓
Webhook
 ↓
Payment Service
```

Challenges:

* Duplicate Webhooks
* Delayed Delivery
* Network Failures

Solution:

```text
Idempotent Processing
```

Benefits:

* Reliable Status Updates

---

# Refund Architecture

Workflow:

```text
Refund Request
      ↓
Refund Validation
      ↓
Gateway Refund
      ↓
Ledger Update
      ↓
Notification
```

States:

```text
REFUND_PENDING

REFUND_SUCCESS

REFUND_FAILED
```

Benefits:

* Full Traceability
* Auditability

---

# Settlement Service

Responsibilities:

* Merchant Settlements
* Payout Processing
* Reconciliation

Architecture:

```text
Transactions
      ↓
Settlement Engine
      ↓
Merchant Payout
```

Benefits:

* Automated Settlements
* Reliable Accounting

---

# Settlement Lifecycle

```text
Payment Success
      ↓
Settlement Eligible
      ↓
Settlement Created
      ↓
Merchant Paid
```

Benefits:

* Clear Settlement Tracking

---

# Fraud Detection System

![Architecture](../../assets/architecture.png)

Signals:

```text
IP Address

Device Fingerprint

Velocity Checks

Geo Location

Risk Score
```

Benefits:

* Reduced Fraud
* Better Security

---

# Fraud Processing Pipeline

```text
Payment Request
       ↓
Risk Engine
       ↓
Risk Score
       ↓
Approve / Review / Reject
```

Benefits:

* Automated Protection

---

# Kafka Architecture

![Kafka](../../assets/kafka.png)

Topics:

```text
payments

refunds

settlements

wallet-events

notification-events
```

Benefits:

* Event Streaming
* Replayability
* Audit History

---

# Event Driven Architecture

Flow:

```text
Payment Service
      ↓
Kafka
      ↓
Ledger Service

Settlement Service

Analytics Service

Notification Service
```

Benefits:

* Loose Coupling
* Scalability
* Independent Deployments

---

# RabbitMQ Architecture

![RabbitMQ](../../assets/rabbitmq.png)

Used For:

```text
Email Notifications

Receipt Generation

Webhook Retries

Background Jobs
```

Benefits:

* Reliable Processing
* Failure Recovery

---

# Notification Service

Channels:

```text
Email

SMS

Push Notification
```

Events:

```text
PAYMENT_SUCCESS

PAYMENT_FAILED

REFUND_SUCCESS
```

Benefits:

* User Awareness
* Better Customer Experience

---

# Redis Architecture

![Redis](../../assets/redis.png)

Stores:

```text
Idempotency Keys

Rate Limits

Sessions

Temporary Payment State
```

Benefits:

* Fast Validation
* Reduced Database Load

---

# Database Architecture

Primary Database:

```text
MySQL
```

Stores:

* Transactions
* Ledgers
* Refunds
* Settlements
* Audit Records

Read Scaling:

```text
Primary
 ↓
Read Replicas
```

Benefits:

* Better Reporting Performance

---

# Reconciliation System

Purpose:

```text
Verify Financial Correctness
```

Compare:

```text
Internal Transactions

Gateway Records

Bank Records
```

Benefits:

* Detect Inconsistencies
* Improve Reliability

---

# Audit Logging

Track:

```text
Payments

Refunds

Settlements

User Actions

System Events
```

Benefits:

* Compliance
* Troubleshooting
* Forensics

---

# Security Architecture

Controls:

```text
TLS Encryption

JWT Authentication

RBAC

Audit Logging

Rate Limiting
```

PCI Principles:

```text
Tokenization

Encryption

Least Privilege
```

Benefits:

* Regulatory Compliance
* Data Protection

---

# Monitoring & Observability

Critical Metrics:

```text
Payment Success Rate

Failure Rate

Refund Success Rate

Webhook Errors

Settlement Delays

Gateway Latency
```

Tools:

* Prometheus
* Grafana
* OpenTelemetry

Benefits:

* Faster Incident Detection

---

# Disaster Recovery

Strategies:

```text
Database Backups

Kafka Replay

Multi-AZ Deployment

Infrastructure As Code
```

Benefits:

* Faster Recovery
* Reduced Downtime

---

# Scaling Strategy

## Stage 1

```text
Single Payment Service
```

---

## Stage 2

```text
Dedicated Ledger Service
```

---

## Stage 3

```text
Event Driven Architecture
```

---

## Stage 4

```text
Multi Gateway Platform
```

---

## Stage 5

```text
Global Payment Infrastructure
```

---

# Engineering Lessons

* Financial correctness is more important than speed.
* Idempotency is mandatory.
* Ledger systems improve reliability.
* Reconciliation is essential.
* Fraud detection should be proactive.
* Event-driven systems improve scalability.
* Monitoring is critical for payment platforms.
* Auditability must be built from the beginning.

---

# Key Takeaways

* Payment systems require strong consistency and reliability.
* Ledgers are foundational components.
* Idempotency protects against duplicate transactions.
* Kafka improves auditability and integration scalability.
* Fraud prevention is a first-class requirement.
* Reconciliation ensures long-term correctness.
* Payment architecture is one of the most important system design interview topics.

---

# Related Documents

* docs/case-studies/payment-case-study.md
* docs/architecture/payment-system.md
* docs/kafka/exactly-once.md
* docs/rabbitmq/retries.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/payment-system.mmd
