# Payment System Architecture

![Payment System](../../assets/payment-system.png)

---

# Overview

Payment systems are among the most critical components in modern software platforms.

Examples:

* Ecommerce Platforms
* Marketplaces
* Fantasy Sports Applications
* Subscription Platforms
* Fintech Products
* Wallet Systems

Unlike many other backend systems, payment systems must prioritize:

* Accuracy
* Consistency
* Auditability
* Security
* Reliability

A single payment bug can result in:

* Revenue loss
* Financial disputes
* Regulatory issues
* Customer trust damage

This document covers production-grade payment architecture used by large-scale systems.

---

# Core Requirements

A payment system must provide:

### Reliability

Payments must not be lost.

---

### Idempotency

Duplicate charges must never occur.

---

### Consistency

Financial records must remain accurate.

---

### Auditability

Every transaction must be traceable.

---

### Security

Sensitive payment data must be protected.

---

# High-Level Payment Architecture

```text id="l0q4gi"
Client
   ↓
API Gateway
   ↓
Payment Service
   ↓
Database
   ↓
Payment Gateway
```

Common Gateways:

* Stripe
* Razorpay
* PayPal
* Adyen
* Cashfree

---

# Payment Flow

```text id="aqcrcw"
User Checkout
      ↓
Create Order
      ↓
Create Payment
      ↓
Gateway Processing
      ↓
Webhook
      ↓
Payment Verification
      ↓
Order Confirmation
```

Every step must be reliable.

---

# Core Components

## Order Service

Tracks:

```text id="trl86m"
Order State
```

Examples:

* Pending
* Paid
* Failed
* Refunded

---

## Payment Service

Responsibilities:

* Payment initiation
* Payment verification
* Reconciliation
* Refunds

---

## Payment Gateway

External provider.

Examples:

```text id="k6h4e7"
Stripe

Razorpay

PayPal
```

Processes actual money movement.

---

## Ledger Service

Records financial activity.

Critical for:

* Audits
* Disputes
* Reconciliation

---

# Payment States

Typical lifecycle:

```text id="4ow0qj"
PENDING
   ↓
PROCESSING
   ↓
SUCCESS
```

or

```text id="udc9u2"
PENDING
   ↓
FAILED
```

or

```text id="3r8r97"
SUCCESS
   ↓
REFUNDED
```

---

# Database Design

Core Tables:

```text id="2uz9mr"
orders

payments

transactions

refunds

ledger_entries
```

---

# Payment Record Example

```json id="4qj9al"
{
  "id": "PAY_1001",
  "order_id": "ORD_500",
  "amount": 999,
  "status": "PENDING"
}
```

---

# Why Idempotency Matters

One of the most important payment concepts.

Scenario:

```text id="7rkn7i"
Create Payment
```

Client:

```text id="y6p9rd"
Network Timeout
```

User retries.

Without protection:

```text id="3hmp95"
Payment #1

Payment #2
```

Double charge.

---

# Idempotency Solution

Client sends:

```http id="lgn9jd"
Idempotency-Key:
abc123
```

Server stores:

```text id="7trqni"
Request Result
```

Repeated requests return:

```text id="l2cefd"
Same Response
```

No duplicate processing.

---

# Payment Gateway Integration

Flow:

```text id="0jlwmn"
Payment Service
       ↓
Gateway API
       ↓
Gateway Response
```

Example:

```text id="sxv4wz"
Payment Intent
```

Created before user payment.

---

# Webhook Architecture

Never trust frontend success responses.

Correct Flow:

```text id="d1ef3v"
Gateway
   ↓
Webhook
   ↓
Payment Service
   ↓
Verify Signature
   ↓
Update Payment
```

Why?

Frontend can be:

* Closed
* Refreshed
* Interrupted

Webhooks are the source of truth.

---

# Webhook Verification

Every webhook should verify:

```text id="f9gt9e"
Signature
```

Example:

```text id="j5vtud"
Stripe Signature
```

Prevents spoofing.

---

# Refund Architecture

Flow:

```text id="7kfj5v"
User Request
      ↓
Refund Service
      ↓
Gateway Refund
      ↓
Ledger Update
      ↓
Notification
```

Refund states:

```text id="l4z6ch"
PENDING

SUCCESS

FAILED
```

---

# Ledger Design

Never directly update balances.

Bad:

```text id="txqajp"
Balance = 100
```

Update:

```text id="iv6tyo"
Balance = 50
```

No history.

---

Good:

```text id="fekg5q"
Debit Entry

Credit Entry
```

Ledger retains complete audit trail.

---

# Double Entry Accounting

Industry standard.

Example:

```text id="ahbn7w"
User Wallet
Debit ₹100

Platform Account
Credit ₹100
```

Benefits:

* Accuracy
* Auditability
* Reconciliation

---

# Wallet Architecture

Common in:

* Fantasy Sports
* Fintech
* Gaming

Flow:

```text id="c3m1ye"
Deposit
 ↓
Ledger
 ↓
Wallet Balance
```

Withdrawal:

```text id="cm4zwg"
Withdrawal
 ↓
Ledger
 ↓
Bank Transfer
```

---

# Reconciliation

Critical process.

Purpose:

Compare:

```text id="r5cz7d"
Internal Records
```

vs

```text id="8dfk0w"
Gateway Records
```

Detect:

* Missing transactions
* Duplicate transactions
* Gateway discrepancies

---

# Reconciliation Flow

```text id="l3xlcn"
Gateway Report
      ↓
Reconciliation Job
      ↓
Differences
      ↓
Investigation
```

Often runs daily.

---

# Event-Driven Payments

Modern systems use events.

Example:

```text id="yvovvj"
Payment Success
      ↓
Kafka Event
```

Consumers:

```text id="t94v2u"
Order Service

Notification Service

Analytics Service
```

Benefits:

* Loose coupling
* Scalability

---

# Payment Failure Handling

Common failures:

* Gateway timeout
* Network errors
* Card declined
* Fraud checks

Architecture:

```text id="7x0h4m"
Failure
 ↓
Retry Logic
 ↓
Escalation
```

---

# Retry Strategy

Example:

```text id="vydcyr"
Attempt 1

Wait 1 Minute

Attempt 2

Wait 5 Minutes

Attempt 3
```

Use exponential backoff.

---

# Payment Security

Requirements:

## HTTPS

Mandatory.

---

## PCI Compliance

Protect card data.

---

## Encryption

Sensitive information encrypted.

---

## Audit Logs

Track:

* Payments
* Refunds
* Failures

---

## Access Controls

Limit operational access.

---

# Ecommerce Example

Checkout Flow:

```text id="mny4x9"
Cart
 ↓
Order
 ↓
Payment
 ↓
Gateway
 ↓
Webhook
 ↓
Success
```

Events:

```text id="ucvnko"
ORDER_PAID
```

Triggers:

* Inventory update
* Shipment creation
* Notifications

---

# Fantasy Sports Example

Wallet Deposit:

```text id="s9v4em"
User Deposit
      ↓
Gateway
      ↓
Wallet Credit
      ↓
Contest Participation
```

Withdrawal:

```text id="wfnw3v"
User Withdrawal
      ↓
Bank Transfer
      ↓
Ledger Update
```

---

# Monitoring Metrics

Track:

## Payment Success Rate

Business-critical metric.

---

## Payment Failures

Gateway issues.

---

## Refund Rate

Financial health indicator.

---

## Webhook Failures

Integration issues.

---

## Reconciliation Mismatches

Financial risk indicator.

---

# Common Mistakes

## No Idempotency

Causes duplicate charges.

---

## Trusting Frontend Success

Always verify through webhooks.

---

## No Ledger

Creates audit problems.

---

## Missing Reconciliation

Leads to financial discrepancies.

---

## Weak Security

High-risk financial exposure.

---

# Common Interview Questions

### Why is idempotency important?

Prevents duplicate payments.

---

### Why use webhooks?

They provide reliable payment confirmation.

---

### What is reconciliation?

Comparing internal records with gateway records.

---

### Why use a ledger?

To maintain a complete audit trail.

---

### What is double-entry accounting?

Every transaction records equal debit and credit entries.

---

### Why are payment systems event-driven?

To notify multiple downstream services reliably.

---

# Production Lessons

* Payments must be treated as financial systems.
* Idempotency is mandatory.
* Webhooks are the source of truth.
* Ledgers provide auditability.
* Reconciliation protects financial integrity.
* Event-driven architectures improve scalability.
* Monitoring payment metrics is business critical.

---

# Key Takeaways

* Payment systems require strong consistency and reliability.
* Idempotency prevents duplicate charges.
* Webhooks verify transaction outcomes.
* Ledgers and double-entry accounting improve auditability.
* Reconciliation is essential.
* Event-driven payments scale effectively.
* Production-grade payment architecture is a cornerstone of modern digital platforms.

---

# Related Documents

* docs/architecture/api-design.md
* docs/architecture/notification-system.md
* docs/architecture/event-driven-architecture.md
* docs/kafka/event-streaming.md

Related Diagram:

* diagrams/payment-system.mmd
