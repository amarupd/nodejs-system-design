# Node.js Memory Management

![Node.js Runtime](../../assets/nodejs-runtime.png)

---

# Overview

Memory management is one of the most critical aspects of building production-grade Node.js applications.

Many large-scale system failures are caused not by CPU bottlenecks or database issues, but by:

* Memory leaks
* Excessive heap growth
* Unbounded caches
* Poor object lifecycle management
* Garbage Collection pauses

Understanding how Node.js manages memory is essential for:

* Backend scalability
* Production reliability
* Performance optimization
* System design interviews
* Senior engineering roles

---

# Why Memory Matters

Consider a Node.js API serving:

```text
100,000 Requests Per Minute
```

If every request leaves behind only:

```text
10 KB
```

of unreleased memory:

```text
100,000 × 10 KB
=
~1 GB leaked
```

Within a short time:

* Memory usage spikes
* Garbage Collection becomes expensive
* Latency increases
* Containers restart
* Services become unstable

---

# Memory Architecture

Node.js memory is primarily managed by the V8 Engine.

High-Level Layout:

```text
Node.js Process
        │
        ├── Call Stack
        │
        ├── Heap Memory
        │
        ├── Native Memory
        │
        └── External Memory
```

Each area serves a different purpose.

---

# Call Stack

Stores:

* Function calls
* Local variables
* Execution context

Example:

```js
function createOrder() {
  const orderId = 123
}
```

When execution completes:

* Stack frame removed
* Memory reclaimed automatically

Characteristics:

* Fast
* Small
* Automatically managed

---

# Heap Memory

Most application data lives here.

Examples:

```js
const user = {
  id: 1,
  name: "Amar"
}
```

Heap stores:

* Objects
* Arrays
* Closures
* Classes
* Buffers

Characteristics:

* Dynamic size
* Garbage collected
* Most common source of memory leaks

---

# V8 Heap Structure

Heap is divided into generations.

```text
Heap
 │
 ├── New Space
 │
 └── Old Space
```

This design optimizes garbage collection.

---

# New Space

Stores:

* Short-lived objects

Examples:

* Temporary request objects
* Function-scoped objects
* Intermediate calculations

Characteristics:

* Small
* Frequently cleaned

Most objects die here.

---

# Old Space

Stores:

* Long-lived objects

Examples:

* Application caches
* Shared configurations
* Persistent data structures

Characteristics:

* Larger
* Less frequent cleanup
* More expensive GC operations

---

# Garbage Collection

V8 automatically frees unused memory.

Goals:

* Reclaim memory
* Prevent leaks
* Optimize performance

Developers generally do not manually free memory.

---

# Minor Garbage Collection

Targets:

```text
New Space
```

Process:

```text
Allocate
↓
Use
↓
Remove Unused Objects
```

Characteristics:

* Fast
* Frequent
* Low overhead

---

# Major Garbage Collection

Targets:

```text
Old Space
```

Characteristics:

* Slower
* More expensive
* Can affect latency

Large heaps increase GC pause times.

---

# Object Lifecycle

Example:

```js
function getUser() {
  const user = {
    id: 1,
    name: "John"
  }

  return user
}
```

Lifecycle:

```text
Create Object
      ↓
Use Object
      ↓
Reference Removed
      ↓
GC Eligible
```

Objects become collectible only when references disappear.

---

# Memory Leaks

Memory leaks occur when memory remains referenced even though it is no longer needed.

Result:

```text
Memory Usage
      ↑
      ↑
      ↑
Never Decreases
```

Eventually:

* OOM errors
* Restarts
* Crashes

---

# Common Memory Leak #1

Global Variables

Bad Example:

```js
const users = []

app.get('/user', (req, res) => {
  users.push(req.body)
})
```

Problem:

```text
users[]
never shrinks
```

Memory grows indefinitely.

---

# Common Memory Leak #2

Unbounded Cache

Bad Example:

```js
const cache = {}

cache[userId] = data
```

Problem:

Entries never expire.

Production Solution:

```text
Redis
+
TTL
```

---

# Common Memory Leak #3

Event Listeners

Bad Example:

```js
emitter.on('event', handler)
```

Repeated registrations:

```text
1
10
100
1000
10000
```

Memory continuously grows.

---

# Common Memory Leak #4

Timers

Bad Example:

```js
setInterval(() => {
  // task
}, 1000)
```

Forgotten intervals:

* Continue forever
* Hold references
* Prevent GC

---

# Common Memory Leak #5

Closures

Example:

```js
function createHandler() {
  const largeData = loadHugeFile()

  return function() {
    return largeData
  }
}
```

Problem:

Closure retains:

```text
largeData
```

Memory never released.

---

# Memory Leak Detection

Indicators:

```text
Memory Usage
 ↑
 ↑
 ↑
 ↑
Constant Growth
```

Symptoms:

* Slow APIs
* Increased GC
* Container restarts
* High memory alerts

---

# Heap Snapshots

One of the most important debugging tools.

Can identify:

* Retained objects
* Leaked references
* Heap growth

Tools:

* Chrome DevTools
* Node Inspector
* Heap Profiler

---

# Memory Monitoring

Production metrics:

## Heap Used

Current heap usage.

---

## Heap Total

Allocated heap.

---

## RSS

Resident Set Size.

Includes:

* Heap
* Native memory
* Buffers

---

## Event Loop Lag

Often increases with memory pressure.

---

# Memory Limits

Default Node.js heap limits vary by platform and version.

Common production setting:

```bash
node --max-old-space-size=4096 app.js
```

Example:

```text
4096 MB
```

Useful for large workloads.

---

# Buffers and Memory

Node.js often processes:

* Files
* Streams
* Network traffic

Using Buffers.

Example:

```js
Buffer.from("hello")
```

Buffers allocate memory outside the V8 heap.

Important:

Heap may appear healthy while RSS grows dramatically.

---

# Memory and Streams

Bad:

```js
fs.readFile()
```

Good:

```js
fs.createReadStream()
```

Benefits:

* Constant memory usage
* Reduced GC pressure
* Better scalability

---

# Redis vs In-Memory Cache

Bad:

```js
const cache = {}
```

Problems:

* Worker isolation
* Memory growth
* No TTL

Better:

```text
Redis
```

Benefits:

* Shared cache
* Expiration support
* Memory control

---

# Kubernetes Considerations

Example:

```yaml
resources:
  limits:
    memory: "512Mi"
```

If application exceeds:

```text
512Mi
```

Container:

```text
OOMKilled
```

Production systems must monitor memory continuously.

---

# Ecommerce Example

Product Catalog Service

Bad Design:

```text
Load Entire Catalog
Into Memory
```

Result:

* Huge heap growth
* Long GC pauses

Better Design:

```text
Database
    ↓
Pagination
    ↓
Redis Cache
```

Benefits:

* Predictable memory usage
* Better scalability

---

# Fantasy Sports Example

Leaderboard Service

Bad:

```text
Entire Leaderboard
Stored In Memory
```

Result:

Millions of users consume excessive RAM.

Better:

```text
Redis Sorted Sets
```

Benefits:

* Efficient ranking
* Lower memory pressure
* Horizontal scalability

---

# Memory Optimization Techniques

## Use Streams

Process incrementally.

---

## Use Pagination

Avoid loading massive datasets.

---

## Apply Cache TTLs

Prevent stale data accumulation.

---

## Remove Event Listeners

Cleanup resources.

---

## Monitor Heap Growth

Track trends continuously.

---

## Use Redis

Offload memory-heavy workloads.

---

# Common Interview Questions

### What is the difference between Stack and Heap?

Stack stores function execution context.

Heap stores dynamic objects.

---

### What causes memory leaks?

Retained references, caches, listeners, timers, and closures.

---

### What is Garbage Collection?

Automatic memory reclamation performed by V8.

---

### Why are memory leaks dangerous?

They eventually lead to increased latency, crashes, and service instability.

---

### How can memory leaks be detected?

Heap snapshots, profiling, monitoring, and memory trend analysis.

---

### Why are streams better for large files?

They avoid loading the entire file into memory.

---

# Production Lessons

* Most Node.js outages involve memory issues.
* Monitor memory before problems occur.
* Use Redis instead of massive in-process caches.
* Favor streams over large in-memory operations.
* Investigate continuously growing heaps immediately.
* Understand RSS, Heap, and GC metrics.
* Design services with predictable memory usage.

---

# Key Takeaways

* Memory management is a core backend engineering skill.
* V8 uses heap generations and garbage collection.
* Most memory leaks are caused by retained references.
* Streams help control memory usage.
* Redis often replaces large in-memory caches.
* Monitoring is essential in production.
* Senior engineers should be comfortable diagnosing and preventing memory-related failures.

---

# Related Documents

* docs/fundamentals/nodejs-runtime.md
* docs/fundamentals/event-loop.md
* docs/fundamentals/streams.md
* docs/redis/caching-patterns.md
* docs/architecture/distributed-systems.md

Related Diagram:

* diagrams/nodejs-runtime.mmd
