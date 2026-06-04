# Ecommerce System Architecture

![Ecommerce Architecture](../../assets/ecommerce.png)

---

# Overview

Ecommerce platforms are among the most common large-scale distributed systems.

Examples:

* Amazon
* Flipkart
* Shopify
* Myntra
* Nike
* D2C Brands

An ecommerce platform appears simple from the user perspective:

```text
Browse Product
↓
Add To Cart
↓
Checkout
↓
Payment
↓
Delivery
```

However, behind the scenes it consists of multiple highly scalable services that must work together reliably.

A production-grade ecommerce platform must support:

* Millions of products
* Millions of users
* High traffic sales events
* Inventory consistency
* Secure payments
* Order fulfillment
* Real-time notifications

---

# Business Requirements

Core capabilities:

### Product Catalog

Product discovery and browsing.

---

### Inventory Management

Accurate stock tracking.

---

### Cart Management

Persistent shopping carts.

---

### Checkout

Order creation.

---

### Payments

Online and offline payment methods.

---

### Order Management

Order lifecycle tracking.

---

### Notifications

Customer communication.

---

### Search

Fast product discovery.

---

# High-Level Architecture

```text
Users
   ↓
CDN
   ↓
API Gateway
   ↓
Microservices
   ↓
Redis / Kafka / MySQL
```

Core Services:

* User Service
* Catalog Service
* Search Service
* Cart Service
* Inventory Service
* Order Service
* Payment Service
* Notification Service
* Shipment Service

---

# Product Catalog Service

Responsible for:

* Products
* Categories
* Variants
* Attributes
* Images

Example:

```text
Product

Sports Jersey
```

Variants:

```text
S
M
L
XL
```

---

# Product Database Design

Stores:

```text
products

categories

variants

attributes

images
```

Read-heavy workload.

Optimization:

```text
Redis Cache
```

---

# Search Architecture

Searching directly from MySQL does not scale.

Architecture:

```text
Catalog Service
       ↓
Kafka
       ↓
Search Index
       ↓
Search API
```

Common Technologies:

* Elasticsearch
* OpenSearch
* Meilisearch

---

# Search Flow

```text
User Search
      ↓
Search Service
      ↓
Search Index
      ↓
Results
```

Target:

```text
< 100 ms
```

---

# Cart Service

Stores:

* Products
* Quantities
* Selected Variants

Architecture:

```text
User
 ↓
Cart Service
 ↓
Redis
```

Why Redis?

* Fast reads
* Fast writes
* Temporary data

---

# Cart Data Example

```json
{
  "product_id": 100,
  "variant_id": 50,
  "quantity": 2
}
```

---

# Inventory Service

One of the most critical services.

Responsibilities:

* Stock tracking
* Reservation
* Replenishment

---

# Inventory Challenges

Problem:

```text
Stock = 1
```

Two users purchase simultaneously.

Result:

```text
Overselling
```

Must be prevented.

---

# Inventory Reservation Flow

```text
Checkout
    ↓
Reserve Stock
    ↓
Payment
    ↓
Confirm Order
```

If payment fails:

```text
Release Stock
```

---

# Redis Locking

Common solution:

```text
Acquire Lock
 ↓
Check Stock
 ↓
Reserve
 ↓
Release Lock
```

Benefits:

* Prevent race conditions
* Prevent overselling

---

# Checkout Architecture

Flow:

```text
Cart
 ↓
Inventory Validation
 ↓
Order Creation
 ↓
Payment
 ↓
Confirmation
```

All steps must be reliable.

---

# Order Service

Stores:

```text
orders

order_items

addresses

status_history
```

Order States:

```text
PENDING

PAID

PROCESSING

SHIPPED

DELIVERED

CANCELLED
```

---

# Order Flow

```text
Order Created
      ↓
Payment Success
      ↓
Processing
      ↓
Shipment
      ↓
Delivered
```

---

# Payment Integration

Flow:

```text
Checkout
 ↓
Payment Service
 ↓
Gateway
 ↓
Webhook
 ↓
Order Update
```

Requirements:

* Idempotency
* Reconciliation
* Auditability

---

# Shipment Service

Responsibilities:

* Courier assignment
* Tracking
* Delivery updates

Integrations:

* Delhivery
* Shiprocket
* FedEx
* DHL

---

# Shipment Flow

```text
Order Paid
      ↓
Create Shipment
      ↓
Courier Pickup
      ↓
Tracking Updates
      ↓
Delivered
```

---

# Notification Architecture

Events:

```text
Order Placed

Payment Success

Order Shipped

Order Delivered
```

Flow:

```text
Kafka
 ↓
Notification Service
```

Channels:

* Email
* SMS
* Push

---

# Event-Driven Architecture

Core ecommerce events:

```text
ORDER_CREATED

PAYMENT_SUCCESS

ORDER_CANCELLED

ORDER_SHIPPED
```

Architecture:

```text
Service
 ↓
Kafka
 ↓
Consumers
```

Benefits:

* Loose coupling
* Better scalability

---

# Caching Strategy

Cache:

```text
Products

Categories

Popular Searches

Inventory Snapshots
```

Architecture:

```text
API
 ↓
Redis
 ↓
MySQL
```

Benefits:

* Lower latency
* Reduced database load

---

# Product Image Architecture

Files stored in:

```text
AWS S3
```

Served through:

```text
CloudFront CDN
```

Benefits:

* Faster delivery
* Lower infrastructure cost

---

# Flash Sale Architecture

Most challenging ecommerce scenario.

Example:

```text
100,000 Users

1,000 Products
```

Available stock:

```text
100 Units
```

---

# Flash Sale Solution

Architecture:

```text
Users
 ↓
Queue
 ↓
Inventory Service
 ↓
Reservation
```

Components:

* Redis
* Kafka
* Distributed Locks

---

# Recommendation System

Inputs:

```text
User Activity

Purchases

Views
```

Outputs:

```text
Recommended Products
```

Architecture:

```text
Events
 ↓
Kafka
 ↓
Recommendation Engine
```

---

# Multi-Region Deployment

Architecture:

```text
Region A

Region B

Region C
```

Benefits:

* Lower latency
* Disaster recovery

---

# Database Strategy

MySQL:

```text
Orders
Users
Payments
```

Redis:

```text
Cart
Cache
Sessions
Locks
```

Kafka:

```text
Events
```

Object Storage:

```text
Images
Videos
```

---

# Monitoring Metrics

Track:

## Revenue

Business KPI.

---

## Order Success Rate

Checkout health.

---

## Payment Success Rate

Gateway performance.

---

## Inventory Accuracy

Operational reliability.

---

## Search Latency

Customer experience.

---

# Security Requirements

## Authentication

Protect user accounts.

---

## Authorization

Admin access control.

---

## PCI Compliance

Payment security.

---

## Audit Logs

Track critical actions.

---

## Rate Limiting

Protect APIs.

---

# Common Mistakes

## Inventory Updates Without Locking

Causes overselling.

---

## Storing Carts In MySQL

Creates unnecessary load.

Use Redis.

---

## No Event Architecture

Creates tight coupling.

---

## Weak Payment Verification

Use webhooks.

---

## No Cache Strategy

Poor scalability.

---

# Common Interview Questions

### Why use Redis for carts?

Fast access and temporary storage.

---

### How do you prevent overselling?

Inventory reservations and distributed locking.

---

### Why use Kafka?

Reliable event distribution across services.

---

### Why use Elasticsearch?

Fast full-text search.

---

### How do flash sales scale?

Queues, caching, and inventory reservation.

---

### Why separate inventory from catalog?

Different scalability requirements.

---

# Production Lessons

* Ecommerce platforms are highly distributed systems.
* Inventory management is one of the hardest challenges.
* Redis significantly improves performance.
* Kafka enables scalable workflows.
* Search requires specialized infrastructure.
* Flash sales require careful architecture.
* Event-driven design improves maintainability.

---

# Key Takeaways

* Ecommerce systems combine catalog, inventory, payments, logistics, and notifications.
* Redis, Kafka, MySQL, and object storage are core building blocks.
* Inventory consistency is critical.
* Search and recommendations require specialized services.
* Event-driven architecture improves scalability.
* Flash sale handling is a common system design interview topic.
* Ecommerce platforms are excellent examples of real-world distributed systems.

---

# Related Documents

* docs/architecture/payment-system.md
* docs/architecture/file-storage-system.md
* docs/architecture/notification-system.md
* docs/architecture/event-driven-architecture.md
* docs/case-studies/ecommerce-case-study.md

Related Diagram:

* diagrams/ecommerce-system.mmd
