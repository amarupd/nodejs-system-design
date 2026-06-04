# Node.js Event Loop Deep Dive

![Node.js Runtime](../../assets/nodejs-runtime.png)

---

# Overview

The Event Loop is the heart of Node.js.

It enables Node.js to handle thousands of concurrent connections using a single JavaScript execution thread while maintaining high performance and low memory consumption.

Understanding the Event Loop is one of the most important topics for:

* Senior Backend Engineers
* System Design Interviews
* Production Performance Tuning
* Scalability Engineering

---

# Why the Event Loop Exists

Traditional web servers often use:

* One thread per request
* Blocking I/O
* Heavy context switching

As traffic grows:

* Memory consumption increases
* CPU overhead increases
* Throughput decreases

Node.js takes a different approach.

Instead of waiting for slow operations to complete, Node.js delegates those operations and continues serving other requests.

This behavior is made possible by the Event Loop.

---

# High-Level Flow

Request arrives

↓

JavaScript execution

↓

Async operation registered

↓

Libuv handles operation

↓

Operation completes

↓

Callback queued

↓

Event Loop executes callback

↓

Response returned

---

# Core Components

## Call Stack

The Call Stack tracks currently executing functions.

Example:

```js
function getUser() {
  return "Amar"
}

function printUser() {
  console.log(getUser())
}

printUser()
```

Execution order:

1. printUser()
2. getUser()
3. return value
4. console.log()

The stack follows:

LIFO

Last In First Out

---

## Web APIs / Node APIs

Some operations take time:

Examples:

* Database calls
* HTTP requests
* File reading
* DNS lookups
* Timers

These operations are delegated outside the JavaScript thread.

---

## Callback Queue

Completed async operations place callbacks into a queue.

Example:

```js
setTimeout(() => {
  console.log("Done")
}, 1000)
```

After 1 second:

Callback enters queue.

It does NOT execute immediately.

The Event Loop decides when it runs.

---

## Event Loop

The Event Loop continuously checks:

1. Is Call Stack empty?
2. Are callbacks waiting?

If yes:

Move callback into stack.

Execute callback.

Repeat forever.

---

# Event Loop Phases

Node.js Event Loop consists of multiple phases.

Understanding these phases is critical for production debugging.

---

## Phase 1: Timers

Executes:

* setTimeout()
* setInterval()

Example:

```js
setTimeout(() => {
  console.log("Timer")
}, 0)
```

Important:

0 milliseconds does NOT mean immediate execution.

The callback runs only when the Event Loop reaches the Timers phase.

---

## Phase 2: Pending Callbacks

Handles:

* TCP errors
* System callbacks
* Deferred operations

Rarely used directly by application developers.

---

## Phase 3: Idle / Prepare

Internal Node.js operations.

Not exposed to developers.

---

## Phase 4: Poll Phase

Most important phase.

Handles:

* Incoming requests
* Database responses
* File system responses
* Socket events

The Poll phase can:

* Execute callbacks
* Wait for new events
* Move to next phase

Most backend application activity occurs here.

---

## Phase 5: Check Phase

Executes:

```js
setImmediate()
```

Example:

```js
setImmediate(() => {
  console.log("Immediate")
})
```

Runs after Poll phase completes.

---

## Phase 6: Close Callbacks

Handles:

* Socket close events
* Connection cleanup

Example:

```js
socket.on("close")
```

---

# Visual Event Loop Lifecycle

Timers

↓

Pending Callbacks

↓

Idle / Prepare

↓

Poll

↓

Check

↓

Close Callbacks

↓

Repeat Forever

---

# Microtasks Queue

Microtasks have higher priority than normal callbacks.

Examples:

* Promise.then()
* Promise.catch()
* Promise.finally()
* queueMicrotask()

Example:

```js
Promise.resolve().then(() => {
  console.log("Promise")
})

setTimeout(() => {
  console.log("Timer")
}, 0)
```

Output:

```text
Promise
Timer
```

Reason:

Microtasks execute before the next Event Loop phase.

---

# process.nextTick()

Special Node.js queue.

Highest priority queue.

Example:

```js
process.nextTick(() => {
  console.log("Next Tick")
})

Promise.resolve().then(() => {
  console.log("Promise")
})
```

Output:

```text
Next Tick
Promise
```

Priority:

1. process.nextTick
2. Promise Microtasks
3. Event Loop Callbacks

---

# Example Execution Order

Code:

```js
console.log("Start")

setTimeout(() => {
  console.log("Timeout")
}, 0)

setImmediate(() => {
  console.log("Immediate")
})

Promise.resolve().then(() => {
  console.log("Promise")
})

process.nextTick(() => {
  console.log("NextTick")
})

console.log("End")
```

Typical Output:

```text
Start
End
NextTick
Promise
Timeout
Immediate
```

Understanding this order is a common interview topic.

---

# How Databases Use the Event Loop

Example:

```js
await db.query(...)
```

Actual flow:

1. Query sent
2. Database works independently
3. Event Loop continues serving requests
4. Result returns
5. Callback executed

The Event Loop is never blocked waiting for the database.

This enables massive concurrency.

---

# Event Loop Blocking

The biggest Node.js scalability problem.

Example:

```js
while(true) {
}
```

Result:

* CPU reaches 100%
* Requests stop
* Server freezes

Reason:

Call Stack never empties.

Event Loop cannot continue.

---

# CPU Bound Operations

Dangerous workloads:

* Video transcoding
* PDF generation
* Image processing
* Large loops
* Data analytics

Impact:

* Increased latency
* Request timeouts
* Poor throughput

---

# Solutions for CPU Work

## Worker Threads

Move CPU work to separate threads.

Good for:

* Encryption
* Image processing
* Data transformation

---

## Queues

Examples:

* RabbitMQ
* Kafka

Move heavy work into background jobs.

Benefits:

* Better API performance
* Fault isolation

---

## Dedicated Services

Examples:

Image Service

Video Service

Analytics Service

Each service scales independently.

---

# Monitoring Event Loop Lag

Production systems measure:

Event Loop Lag

Definition:

Time delay before queued callbacks execute.

High lag indicates:

* CPU bottlenecks
* Blocking code
* Memory pressure

Common tools:

* Prometheus
* Grafana
* New Relic
* Datadog

---

# Production Example

![Architecture](../../assets/architecture.png)

Fantasy Sports Platform:

Users:

500,000+

Concurrent Requests:

20,000+

Architecture:

Load Balancer

↓

Node.js APIs

↓

Redis Cache

↓

MySQL

↓

Kafka

↓

Worker Services

Event Loop handles:

* API requests
* Live score updates
* User sessions
* Contest joins

Heavy processing is delegated to:

* Kafka consumers
* Worker services

This prevents Event Loop blocking.

---

# Common Interview Questions

### What is the Event Loop?

A mechanism that continuously checks for queued callbacks and executes them when the Call Stack becomes empty.

---

### Why can Node.js handle many concurrent requests?

Because it uses non-blocking I/O and delegates long-running operations.

---

### Difference between setTimeout and setImmediate?

setTimeout executes during the Timers phase.

setImmediate executes during the Check phase.

---

### Difference between process.nextTick and Promise?

process.nextTick executes before Promise microtasks.

---

### What is Event Loop Lag?

The delay between when a callback becomes ready and when it actually executes.

---

### What blocks the Event Loop?

CPU-intensive operations, synchronous APIs, infinite loops, and excessive computation.

---

# Production Lessons

* Avoid synchronous APIs.
* Keep request handlers lightweight.
* Move heavy processing to queues.
* Monitor Event Loop lag.
* Use Worker Threads when necessary.
* Design APIs around non-blocking operations.
* Never perform CPU-heavy work in request-response cycles.

---

# Key Takeaways

* The Event Loop is the foundation of Node.js scalability.
* Async I/O allows Node.js to serve thousands of concurrent requests.
* Microtasks have higher priority than callbacks.
* process.nextTick has the highest execution priority.
* Blocking the Event Loop directly impacts application performance.
* Production systems rely on queues and workers to keep the Event Loop responsive.

---

# Related Documents

* docs/fundamentals/nodejs-runtime.md
* docs/fundamentals/streams.md
* docs/fundamentals/clustering.md
* docs/fundamentals/worker-threads.md

Related Diagram:

* diagrams/event-loop.mmd
