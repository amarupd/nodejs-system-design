# Notification System Architecture

![Architecture](../../assets/architecture.png)

---

# Overview

Notifications are a core component of modern digital platforms.

Examples:

* Ecommerce order updates
* Payment confirmations
* Password reset emails
* OTP delivery
* Fantasy sports contest updates
* Live score alerts
* Marketing campaigns
* System alerts

A production-grade notification system must be:

* Reliable
* Scalable
* Fault tolerant
* Observable
* Cost efficient

Poor notification architecture leads to:

* Lost messages
* Delayed delivery
* User frustration
* Revenue loss

---

# Notification Types

Most systems support multiple notification channels.

## Email

Examples:

* Welcome emails
* Order confirmations
* Password resets

Providers:

* SES
* SendGrid
* Mailgun

---

## SMS

Examples:

* OTPs
* Transaction alerts

Providers:

* Twilio
* MSG91
* Vonage

---

## Push Notifications

Examples:

* Contest reminders
* Flash sales
* Match updates

Providers:

* Firebase Cloud Messaging (FCM)
* APNs

---

## In-App Notifications

Examples:

* New message alerts
* Wallet updates
* Promotional messages

Stored inside application databases.

---

## WebSocket Notifications

Examples:

* Live scores
* Leaderboard updates
* Real-time events

Technologies:

* Socket.IO
* WebSockets

---

# Notification Requirements

A scalable notification platform must provide:

### Reliability

Messages should not be lost.

---

### Scalability

Support millions of users.

---

### Retry Mechanisms

Handle provider failures.

---

### Multi-Channel Support

Email, SMS, Push, In-App.

---

### Delivery Tracking

Know whether messages were delivered.

---

# High-Level Architecture

```text
Application
      ↓
Notification Service
      ↓
Message Queue
      ↓
Workers
      ↓
Providers
      ↓
Users
```

Benefits:

* Decoupled architecture
* Better scalability
* Fault tolerance

---

# Why Use Queues?

Bad Design:

```text
Order Created
      ↓
Send Email
      ↓
Wait For Provider
      ↓
Response
```

Problems:

* Increased API latency
* Provider dependency
* Poor scalability

---

Good Design:

```text
Order Created
      ↓
Publish Event
      ↓
Queue
      ↓
Worker
      ↓
Provider
```

Benefits:

* Fast API responses
* Independent scaling
* Reliability

---

# Notification Service Responsibilities

Core responsibilities:

* Template management
* Channel selection
* Queue publishing
* Retry handling
* Delivery tracking
* User preferences

---

# Event-Driven Notification Flow

Example:

```text
Order Created
      ↓
Kafka Event
      ↓
Notification Service
      ↓
Email Queue
      ↓
Worker
      ↓
SES
```

Benefits:

* Loose coupling
* Independent services
* Easier scaling

---

# Notification Database Design

Core tables:

```text
notifications

notification_templates

notification_logs

user_preferences
```

---

# Notification Entity Example

```json
{
  "id": 1001,
  "user_id": 500,
  "type": "ORDER_CREATED",
  "channel": "EMAIL",
  "status": "PENDING"
}
```

---

# Template Management

Never hardcode messages.

Bad:

```js
sendEmail(
  "Order Created"
)
```

---

Good:

```text
ORDER_CREATED_TEMPLATE
```

Benefits:

* Easier maintenance
* Localization support
* Marketing flexibility

---

# Multi-Channel Architecture

```text
Notification Service
        │
        ├── Email
        ├── SMS
        ├── Push
        ├── In-App
        └── WebSocket
```

A single event may trigger multiple channels.

---

# User Preferences

Users should control communication.

Example:

```json
{
  "email": true,
  "sms": false,
  "push": true
}
```

Benefits:

* Better user experience
* Regulatory compliance

---

# Retry Mechanism

Provider failures are normal.

Example:

```text
SES Timeout
```

Without retries:

```text
Notification Lost
```

---

Production Flow:

```text
Failed Delivery
      ↓
Retry Queue
      ↓
Retry Worker
      ↓
Provider
```

---

# Retry Strategy

Common pattern:

```text
Attempt 1

Wait 1 Minute

Attempt 2

Wait 5 Minutes

Attempt 3

Wait 15 Minutes
```

Exponential Backoff.

Benefits:

* Reduced provider pressure
* Better recovery

---

# Dead Letter Queue (DLQ)

After repeated failures:

```text
Notification
      ↓
Dead Letter Queue
```

Benefits:

* Prevents infinite retries
* Enables investigation

---

# Delivery Tracking

Track:

```text
Pending

Sent

Delivered

Failed

Opened

Clicked
```

Critical for:

* Marketing analytics
* Compliance
* Troubleshooting

---

# Email Notification Architecture

```text
Order Service
      ↓
Kafka
      ↓
Notification Service
      ↓
Email Queue
      ↓
Email Worker
      ↓
SES
```

Benefits:

* High throughput
* Reliability

---

# SMS Architecture

```text
Authentication Service
      ↓
OTP Event
      ↓
SMS Queue
      ↓
SMS Worker
      ↓
Provider
```

Requirements:

* Low latency
* High reliability

---

# Push Notification Architecture

```text
Application
      ↓
Notification Service
      ↓
FCM
      ↓
Mobile Devices
```

Benefits:

* Real-time communication

---

# In-App Notification Architecture

```text
Event
 ↓
Database
 ↓
Notification API
 ↓
User
```

Benefits:

* Persistent history
* Easy retrieval

---

# Live Score Notification Example

Architecture:

```text
Score Provider
       ↓
Kafka
       ↓
Score Service
       ↓
Notification Service
       ↓
WebSocket
       ↓
Users
```

Events:

* Wicket
* Goal
* Boundary
* Match Start

---

# Fantasy Sports Example

Events:

```text
Contest Joined

Contest Filled

Match Started

Leaderboard Updated

Winnings Credited
```

Architecture:

```text
Fantasy Service
       ↓
Kafka
       ↓
Notification Service
```

---

# Ecommerce Example

Events:

```text
Order Placed

Order Shipped

Order Delivered

Refund Processed
```

Channels:

* Email
* SMS
* Push

---

# Scaling Notification Systems

Strategies:

## Queue-Based Processing

Independent worker scaling.

---

## Horizontal Workers

```text
Worker 1
Worker 2
Worker 3
Worker 4
```

Scale based on demand.

---

## Kafka Partitions

Increase throughput.

---

## Provider Failover

Primary:

```text
SES
```

Fallback:

```text
SendGrid
```

Improves availability.

---

# Monitoring

Track:

## Messages Sent

Total notifications.

---

## Delivery Rate

Successful deliveries.

---

## Failure Rate

Provider issues.

---

## Retry Rate

System health indicator.

---

## Queue Lag

Worker performance.

---

# Security Considerations

## Prevent Spam

Apply rate limiting.

---

## Template Validation

Prevent malicious content.

---

## Secure Webhooks

Verify provider callbacks.

---

## User Consent

Required for marketing communications.

---

# Common Mistakes

## Sending Notifications Synchronously

Creates latency.

Use queues.

---

## No Retry Mechanism

Leads to message loss.

---

## No DLQ

Makes troubleshooting difficult.

---

## Hardcoded Templates

Difficult to maintain.

---

## Ignoring User Preferences

Poor user experience.

---

# Common Interview Questions

### Why use queues for notifications?

To decouple delivery from request processing.

---

### Why use retries?

External providers occasionally fail.

---

### What is a Dead Letter Queue?

A queue containing messages that repeatedly fail processing.

---

### Why use Kafka for notifications?

Reliable event distribution and scalability.

---

### How do you scale a notification service?

By increasing worker count and queue throughput.

---

### Why track delivery status?

For analytics, debugging, and compliance.

---

# Production Lessons

* Notifications should be asynchronous.
* Kafka and RabbitMQ are excellent notification backbones.
* Retry mechanisms are mandatory.
* DLQs improve reliability.
* Delivery tracking provides valuable visibility.
* User preferences should always be respected.
* Notification systems require strong observability.

---

# Key Takeaways

* Notification systems are event-driven by nature.
* Queues improve reliability and scalability.
* Multi-channel communication is standard.
* Retry and DLQ strategies prevent message loss.
* Kafka, RabbitMQ, Redis, and WebSockets are commonly used.
* Monitoring and delivery tracking are essential.
* Production-grade notification systems are critical infrastructure components.

---

# Related Documents

* docs/architecture/event-driven-architecture.md
* docs/rabbitmq/rabbitmq-patterns.md
* docs/kafka/event-streaming.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/notification-system.mmd
