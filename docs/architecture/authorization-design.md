# Authorization System Design

![Architecture](../../assets/architecture.png)

---

# Overview

Authorization determines what an authenticated user is allowed to do.

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to access?
```

Authorization is one of the most critical components of production systems because improper access control can expose:

* Customer data
* Financial information
* Administrative functionality
* Internal services
* Sensitive business operations

A scalable authorization architecture must remain secure while supporting millions of users and thousands of permission checks per second.

---

# Authentication vs Authorization

Example:

```text
User Logs In
```

Authentication:

```text
Identity Verified
```

Authorization:

```text
Can User Access Resource?
```

Example:

```text
Admin Dashboard
```

Authenticated User:

✅ Yes

Authorized User:

Depends on permissions.

---

# Authorization Requirements

A production-grade authorization system should provide:

### Security

Prevent unauthorized access.

---

### Scalability

Handle large volumes of permission checks.

---

### Flexibility

Support evolving business rules.

---

### Auditability

Track permission changes.

---

### Maintainability

Easy to manage roles and permissions.

---

# High-Level Authorization Architecture

```text
Client
   ↓
API Gateway
   ↓
Authorization Layer
   ↓
Permission Store
   ↓
Database
```

Authorization Layer Responsibilities:

* Validate permissions
* Enforce policies
* Evaluate access rules

Permission Store:

* Roles
* Permissions
* Policies

---

# Authorization Flow

```text
User Request
      ↓
Authentication
      ↓
Identity Verified
      ↓
Authorization Check
      ↓
Access Granted / Denied
```

Authorization always happens after authentication.

---

# Access Control Models

The three most common models are:

1. RBAC
2. ABAC
3. Policy-Based Authorization

---

# Role-Based Access Control (RBAC)

Most widely used model.

Permissions are grouped into roles.

Users receive roles.

Example:

```text
Admin
Manager
Support
Customer
```

---

# RBAC Architecture

```text
Permission
      ↓
Role
      ↓
User
```

Example:

```text
Create Product
Delete Product
Manage Inventory
```

Assigned to:

```text
Admin
```

---

# Ecommerce RBAC Example

Roles:

```text
Customer
```

Permissions:

* View Products
* Place Orders

---

Roles:

```text
Admin
```

Permissions:

* Create Products
* Update Products
* Delete Products
* Manage Orders

---

Roles:

```text
Support Agent
```

Permissions:

* View Orders
* Manage Tickets

---

# Fantasy Sports RBAC Example

Roles:

```text
User
```

Permissions:

* Join Contest
* Create Team

---

Roles:

```text
Operations
```

Permissions:

* Cancel Contest
* Adjust Leaderboard

---

Roles:

```text
Finance
```

Permissions:

* Manage Wallets
* Process Refunds

---

# RBAC Advantages

* Easy to understand
* Easy to implement
* High performance
* Suitable for most systems

---

# RBAC Limitations

Problem:

Large enterprises may have:

```text
100+
Roles
```

Result:

Role Explosion

Difficult management.

---

# Attribute-Based Access Control (ABAC)

More flexible approach.

Access decisions depend on attributes.

Example:

User Attributes:

```text
Department
Country
Role
```

Resource Attributes:

```text
Owner
Region
Type
```

Environment Attributes:

```text
Time
IP Address
Location
```

---

# ABAC Example

Policy:

```text
Support Agent

Can Access

Only Tickets
Assigned To Them
```

Decision based on:

```text
User ID
Ticket Owner
```

Not just role.

---

# ABAC Flow

```text
Request
   ↓
Attributes
   ↓
Policy Engine
   ↓
Decision
```

Benefits:

* Fine-grained control
* Dynamic permissions

---

# Policy-Based Authorization

Most flexible model.

Rules expressed as policies.

Example:

```text
Allow

If:
User Department = Finance

AND

Transaction Amount < $10,000
```

---

# Policy Engine Architecture

```text
User
 ↓
Policy Engine
 ↓
Rules
 ↓
Decision
```

Popular systems:

* OPA (Open Policy Agent)
* Cedar
* Casbin

---

# Resource Ownership

Common authorization pattern.

Example:

```text
Order Owner
```

Can access:

```text
Own Orders
```

Cannot access:

```text
Other Users' Orders
```

Rule:

```text
order.user_id
==
currentUser.id
```

---

# Multi-Tenant Authorization

Critical for SaaS platforms.

Example:

```text
Tenant A
```

Must never access:

```text
Tenant B Data
```

Architecture:

```text
Tenant Isolation
```

Permission checks include:

```text
tenant_id
```

---

# Authorization Database Design

Common tables:

Users

↓

Roles

↓

Permissions

---

Example:

```text
users

roles

permissions

user_roles

role_permissions
```

Provides flexibility and scalability.

---

# Permission Caching

Authorization checks happen frequently.

Problem:

Database lookup per request.

Solution:

```text
Redis Cache
```

Flow:

```text
User Request
      ↓
Redis Permission Cache
      ↓
Database Fallback
```

Benefits:

* Lower latency
* Reduced database load

---

# API Authorization

Example:

```http
DELETE /products/101
```

Required Permission:

```text
product.delete
```

Authorization Middleware:

```text
Validate Permission
```

Before controller execution.

---

# Microservices Authorization

Architecture:

```text
Gateway
   ↓
Auth Service
   ↓
Microservices
```

Authorization may occur:

1. Gateway level
2. Service level
3. Both

---

# Centralized Authorization

Architecture:

```text
Service
   ↓
Authorization Service
   ↓
Decision
```

Benefits:

* Single source of truth
* Consistent policies

Challenges:

* Additional latency
* Service dependency

---

# Distributed Authorization

Each service evaluates permissions.

Benefits:

* Faster
* Independent

Challenges:

* Policy duplication

---

# Audit Logging

Every sensitive action should be logged.

Example:

```text
User
Action
Timestamp
IP
Result
```

Example:

```text
Admin Deleted Product
```

Benefits:

* Compliance
* Security investigations

---

# Permission Revocation

Problem:

Admin removes permission.

User token remains valid.

Solutions:

```text
Short Token Lifetime
```

or

```text
Permission Versioning
```

or

```text
Redis Revocation Cache
```

---

# Security Best Practices

## Least Privilege

Users receive only necessary permissions.

---

## Deny By Default

If permission missing:

```text
Access Denied
```

---

## Audit Everything

Track critical actions.

---

## Centralized Policies

Reduce inconsistencies.

---

## Cache Carefully

Permissions change.

Cache invalidation matters.

---

# Ecommerce Example

Product Management:

```text
Admin
```

Can:

* Create Product
* Delete Product
* Update Inventory

Customer:

Cannot.

---

# Fantasy Sports Example

Operations Team:

Can:

* Cancel Contest
* Recalculate Results

Normal Users:

Cannot.

---

# Monitoring Authorization Systems

Track:

* Authorization failures
* Permission cache hit rate
* Policy evaluation latency
* Audit events
* Role changes

Tools:

* Prometheus
* Grafana
* Datadog

---

# Common Authorization Mistakes

## Client-Side Authorization Only

Never trust frontend permissions.

Always validate on backend.

---

## Excessive Permissions

Violates least privilege.

---

## Missing Audit Logs

Makes investigations difficult.

---

## Hardcoded Permissions

Becomes unmanageable.

---

## No Tenant Isolation

Critical SaaS security risk.

---

# Common Interview Questions

### What is authorization?

The process of determining what actions a user can perform.

---

### Difference between RBAC and ABAC?

RBAC uses roles.

ABAC uses attributes and policies.

---

### Why cache permissions?

To reduce latency and database load.

---

### What is least privilege?

Granting only the minimum required access.

---

### Why audit authorization decisions?

For security, compliance, and investigations.

---

### What is multi-tenant authorization?

Ensuring users can only access resources belonging to their tenant.

---

# Production Lessons

* Authorization is a business-critical security layer.
* RBAC works for most applications.
* ABAC provides greater flexibility.
* Redis improves authorization performance.
* Multi-tenant systems require strict isolation.
* Audit logs are essential.
* Never rely solely on frontend permission checks.

---

# Key Takeaways

* Authorization determines access rights.
* RBAC, ABAC, and policy-based systems are the primary approaches.
* Permission caching improves scalability.
* Auditability is a core requirement.
* Multi-tenant systems require additional safeguards.
* Authorization architecture must evolve with business complexity.
* Strong authorization design is fundamental to secure distributed systems.

---

# Related Documents

* docs/architecture/authentication-design.md
* docs/architecture/api-design.md
* docs/architecture/distributed-systems.md
* docs/redis/caching-patterns.md

Related Diagram:

* diagrams/authorization-flow.mmd
