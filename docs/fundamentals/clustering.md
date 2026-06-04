# Node.js Clustering Architecture

![Architecture](../../assets/architecture.png)

---

# Overview

Node.js executes JavaScript on a single thread.

While this model is excellent for handling I/O-bound workloads, a single Node.js process can only utilize one CPU core at a time.

Modern production servers often have:

* 4 CPU cores
* 8 CPU cores
* 16 CPU cores
* 32+ CPU cores

Without clustering, most CPU resources remain underutilized.

Node.js Clustering enables applications to fully utilize multi-core machines by running multiple Node.js processes behind a shared server port.

This is a fundamental scalability technique used in production-grade backend systems.

---

# The Problem

Single Node.js Process:

```text
CPU Core 1  ✓ Used
CPU Core 2  ✗ Idle
CPU Core 3  ✗ Idle
CPU Core 4  ✗ Idle
```

Result:

* Limited throughput
* Lower concurrency
* Wasted hardware resources

Even though Node.js handles many concurrent requests efficiently, a single process cannot leverage all available CPU cores.

---

# Clustering Solution

Multiple Node.js Workers

```text
Load Balancer
        │
        ▼
 Master Process
   ├── Worker 1
   ├── Worker 2
   ├── Worker 3
   └── Worker 4
```

Benefits:

* Full CPU utilization
* Increased throughput
* Better fault tolerance
* Improved scalability

---

# Cluster Components

## Primary (Master) Process

Responsibilities:

* Spawn worker processes
* Monitor worker health
* Restart failed workers
* Coordinate worker lifecycle

The primary process does not typically handle application traffic.

---

## Worker Processes

Each worker:

* Runs a separate Node.js instance
* Has its own memory space
* Has its own Event Loop
* Handles requests independently

Important:

Workers do not share memory.

Communication requires IPC (Inter-Process Communication).

---

# Basic Cluster Example

```js
const cluster = require("cluster")
const os = require("os")

if (cluster.isPrimary) {
  const cpuCount = os.cpus().length

  for (let i = 0; i < cpuCount; i++) {
    cluster.fork()
  }
} else {
  require("./server")
}
```

Result:

One worker process per CPU core.

---

# Request Distribution

Incoming requests arrive on a shared port.

Node.js distributes requests among workers.

Example:

```text
Client Requests

       ↓

Load Balancer

       ↓

Worker 1
Worker 2
Worker 3
Worker 4
```

This improves overall throughput significantly.

---

# Scaling Impact

Example API:

Single Process

```text
5,000 req/sec
```

4 Core Server

```text
20,000 req/sec
```

8 Core Server

```text
40,000 req/sec
```

Actual results vary depending on:

* Database bottlenecks
* Network latency
* Redis usage
* Application logic

---

# Worker Failure Recovery

Production systems assume failures will occur.

Worker Crash:

```text
Worker 2

✗ Crashed
```

Primary Process:

```text
Detect Failure

↓

Spawn Replacement Worker
```

Example:

```js
cluster.on("exit", worker => {
  cluster.fork()
})
```

Benefits:

* High availability
* Self-healing systems
* Reduced downtime

---

# Memory Architecture

Important concept:

Workers do NOT share memory.

Example:

Worker 1:

```js
users = 100
```

Worker 2:

```js
users = 0
```

Each worker has an isolated memory space.

Implications:

* In-memory caches are not shared
* User sessions are not shared
* Application state is not shared

---

# Session Management Problem

Without shared storage:

User logs in via Worker 1

↓

Session stored in memory

↓

Next request hits Worker 3

↓

Session not found

↓

User appears logged out

---

# Production Solution

Use Redis.

Architecture:

```text
Worker 1 ─┐
Worker 2 ─┼── Redis
Worker 3 ─┤
Worker 4 ─┘
```

Benefits:

* Shared sessions
* Shared cache
* Consistent state

---

# Clustering and Redis

Common architecture:

```text
Load Balancer
      ↓
Node Cluster
      ↓
Redis
      ↓
MySQL
```

Redis acts as:

* Shared cache
* Session store
* Pub/Sub broker

---

# Sticky Sessions

Some applications require requests from a user to always hit the same worker.

Examples:

* WebSocket connections
* Socket.IO applications

Problem:

Connection established on Worker 1.

Subsequent requests hit Worker 3.

Connection state is lost.

---

# Sticky Session Solution

Load Balancer:

```text
User A → Worker 1

User B → Worker 2

User C → Worker 3
```

Common tools:

* NGINX
* HAProxy
* AWS ALB

---

# Clustering and Socket.IO

Production architecture:

```text
Users
  ↓
Load Balancer
  ↓
Node Cluster
  ↓
Redis Adapter
  ↓
Socket.IO Workers
```

Redis synchronizes events between workers.

Example:

Message sent to Worker 1.

Redis broadcasts to:

* Worker 2
* Worker 3
* Worker 4

---

# Clustering vs Worker Threads

Common interview question.

## Cluster

Purpose:

Scale applications across CPU cores.

Characteristics:

* Multiple processes
* Separate memory
* Fault isolation

Best For:

* API servers
* Web applications
* Microservices

---

## Worker Threads

Purpose:

Run CPU-heavy workloads.

Characteristics:

* Same process
* Separate threads
* Shared memory possible

Best For:

* Image processing
* Encryption
* Analytics

---

# Kubernetes and Clustering

Modern deployments often use both:

```text
Kubernetes Pods

Pod 1
 ├─ Worker 1
 ├─ Worker 2

Pod 2
 ├─ Worker 1
 ├─ Worker 2
```

Benefits:

* Horizontal scaling
* Multi-core utilization
* High availability

---

# Production Example

Fantasy Sports Platform

Traffic:

* 500k users
* Match days
* Heavy concurrent activity

Architecture:

```text
Cloud Load Balancer
        ↓
Kubernetes
        ↓
Node.js Cluster
        ↓
Redis
        ↓
MySQL
        ↓
Kafka
```

Cluster Responsibilities:

* User authentication
* Contest joins
* Team creation
* Wallet operations

Heavy processing delegated to:

* Kafka consumers
* Background workers

---

# Monitoring Cluster Health

Track:

* CPU usage
* Memory usage
* Worker restarts
* Event Loop lag
* Throughput
* Error rates

Tools:

* Prometheus
* Grafana
* Datadog
* New Relic

---

# Common Mistakes

## Storing Sessions In Memory

Problem:

Not shared between workers.

Solution:

Redis.

---

## Assuming Workers Share State

They do not.

Use:

* Redis
* Databases
* Message queues

---

## Ignoring Worker Crashes

Always restart failed workers.

---

## Excessive Worker Count

Workers should generally match CPU cores.

Too many workers:

* Increased context switching
* Reduced performance

---

# Common Interview Questions

### Why is clustering needed?

A single Node.js process only utilizes one CPU core.

---

### Do cluster workers share memory?

No.

Each worker has an isolated memory space.

---

### How do clustered applications share state?

Using external systems such as Redis or databases.

---

### Difference between Cluster and Worker Threads?

Cluster scales application processes.

Worker Threads scale CPU-intensive workloads.

---

### Why are sticky sessions important?

They ensure requests consistently reach the same worker when connection state is required.

---

# Production Lessons

* Use clustering to utilize all CPU cores.
* Store sessions in Redis.
* Never depend on in-memory state.
* Monitor worker failures.
* Combine clustering with load balancing.
* Use Kubernetes for horizontal scaling.
* Use Worker Threads for CPU-intensive tasks.

---

# Key Takeaways

* Clustering is a core Node.js scalability strategy.
* Multiple workers increase throughput dramatically.
* Workers are isolated processes.
* Redis is commonly used for shared state.
* Sticky sessions are important for WebSockets.
* Clustering and Kubernetes work well together.
* Production-grade Node.js systems almost always leverage clustering or container-based horizontal scaling.

---

# Related Documents

* docs/fundamentals/nodejs-runtime.md
* docs/fundamentals/event-loop.md
* docs/fundamentals/worker-threads.md
* docs/redis/pubsub.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/deployment-architecture.mmd
