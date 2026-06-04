# Node.js Streams Architecture

![Node.js Runtime](../../assets/nodejs-runtime.png)

---

# Overview

Streams are one of the most powerful features of Node.js.

They allow applications to process data incrementally rather than loading entire datasets into memory.

Streams are fundamental for building:

* High-performance APIs
* File processing systems
* Media platforms
* Data pipelines
* Real-time applications
* Large-scale distributed systems

Understanding streams is essential for senior backend engineering and system design interviews.

---

# Why Streams Matter

Consider a 5 GB file.

Without streams:

```js
const data = fs.readFileSync("large-file.csv")
```

Problems:

* Entire file loaded into memory
* High RAM consumption
* Event Loop pressure
* Potential crashes

With streams:

```js
const stream = fs.createReadStream("large-file.csv")
```

Benefits:

* Small chunks processed at a time
* Constant memory usage
* Better scalability
* Faster processing

---

# Traditional Processing vs Stream Processing

## Traditional Approach

File

↓

Load Entire File

↓

Process

↓

Return Result

Memory Usage:

Very High

---

## Stream-Based Approach

File

↓

Chunk 1

↓

Chunk 2

↓

Chunk 3

↓

Process Incrementally

Memory Usage:

Low and Predictable

---

# Stream Types

Node.js provides four primary stream types.

---

## Readable Streams

Used to consume data.

Examples:

* Reading files
* HTTP requests
* Database exports
* Kafka consumers

Example:

```js
const fs = require("fs")

const stream = fs.createReadStream("data.csv")

stream.on("data", chunk => {
  console.log(chunk)
})
```

---

## Writable Streams

Used to write data.

Examples:

* File uploads
* Logging
* Response generation

Example:

```js
const fs = require("fs")

const stream = fs.createWriteStream("output.txt")

stream.write("Hello")
stream.end()
```

---

## Duplex Streams

Support both:

* Reading
* Writing

Examples:

* TCP sockets
* WebSockets
* Database connections

Example:

```js
socket.write("ping")
socket.on("data", ...)
```

---

## Transform Streams

Modify data while streaming.

Examples:

* Compression
* Encryption
* Data formatting

Example:

```js
input
↓
transform
↓
output
```

Common use cases:

* Gzip
* CSV processing
* ETL pipelines

---

# Stream Lifecycle

Readable Stream

↓

Open

↓

Read Chunks

↓

Emit Events

↓

End

↓

Close

---

# Important Stream Events

## data

Triggered when a chunk arrives.

```js
stream.on("data", chunk => {
  console.log(chunk)
})
```

---

## end

Triggered when reading completes.

```js
stream.on("end", () => {
  console.log("Finished")
})
```

---

## error

Triggered when a failure occurs.

```js
stream.on("error", err => {
  console.error(err)
})
```

---

## close

Triggered when stream closes.

```js
stream.on("close", () => {
  console.log("Closed")
})
```

---

# Backpressure

One of the most important stream concepts.

Problem:

Producer is faster than consumer.

Example:

Fast:

* Kafka producer
* File reader

Slow:

* Database writes
* API requests

Without protection:

Memory grows uncontrollably.

---

# Backpressure Example

Producer:

10,000 messages/sec

Consumer:

1,000 messages/sec

Result:

Queue grows indefinitely.

Memory increases.

Eventually:

* Crashes
* OOM errors
* High latency

---

# Node.js Backpressure Solution

Streams automatically manage flow.

Example:

```js
if (!stream.write(data)) {
  source.pause()
}
```

Resume:

```js
stream.on("drain", () => {
  source.resume()
})
```

Benefits:

* Stable memory usage
* Better throughput
* Predictable performance

---

# Pipe Architecture

The pipe method is one of the most useful stream features.

Example:

```js
readable.pipe(writable)
```

Flow:

Source

↓

Readable Stream

↓

Pipe

↓

Writable Stream

---

# Real Example

```js
const fs = require("fs")

fs.createReadStream("input.txt")
  .pipe(fs.createWriteStream("output.txt"))
```

Advantages:

* Minimal memory usage
* Simple implementation
* Automatic backpressure

---

# Transform Streams

Transform streams process data in transit.

Example:

File

↓

Transform

↓

Compressed File

---

# Gzip Example

```js
const fs = require("fs")
const zlib = require("zlib")

fs.createReadStream("large.log")
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream("large.log.gz"))
```

Benefits:

* Efficient compression
* Minimal memory footprint

---

# Streams in HTTP Systems

HTTP requests and responses are streams.

Request Flow:

Client

↓

Readable Stream

↓

API

↓

Writable Stream

↓

Response

---

# File Upload Architecture

Large File Upload

↓

API Gateway

↓

Node.js Stream

↓

S3 Upload Stream

↓

AWS S3

Benefits:

* Constant memory usage
* Faster uploads
* Supports very large files

---

# Video Streaming Architecture

![Architecture](../../assets/architecture.png)

Client

↓

Load Balancer

↓

Node.js API

↓

Video Stream

↓

CDN

↓

User

Advantages:

* Low latency
* Efficient bandwidth usage
* High concurrency

---

# CSV Processing System

Example:

50 GB CSV Import

Without Streams:

* Server crash
* Memory exhaustion

With Streams:

CSV

↓

Chunk Processing

↓

Validation

↓

Batch Inserts

↓

Database

Benefits:

* Predictable RAM usage
* Faster processing
* Better scalability

---

# Streams and Redis

Example:

Massive cache export.

Redis

↓

Readable Stream

↓

Transform

↓

File

Allows exporting millions of records safely.

---

# Streams and Kafka

Kafka Consumer

↓

Readable Stream

↓

Transform

↓

Database

Benefits:

* Real-time processing
* Event-driven architecture
* Horizontal scaling

---

# Streams and RabbitMQ

Queue

↓

Consumer Stream

↓

Processor

↓

Database

Helps handle:

* High throughput
* Continuous workloads
* Background jobs

---

# Performance Advantages

Streams provide:

* Lower memory usage
* Better throughput
* Faster startup
* Improved scalability
* Reduced GC pressure

Particularly important in:

* Ecommerce platforms
* Fantasy sports systems
* Video platforms
* Analytics systems

---

# Common Mistakes

## Loading Entire Files

Bad:

```js
fs.readFile()
```

Good:

```js
fs.createReadStream()
```

---

## Ignoring Errors

Bad:

No error handlers.

Good:

```js
stream.on("error", handler)
```

---

## Ignoring Backpressure

Can cause:

* Memory spikes
* Performance degradation

Always respect stream flow control.

---

# Production Example

Ecommerce Order Export

Millions of orders

Architecture:

MySQL

↓

Readable Stream

↓

CSV Transform

↓

Compression

↓

S3 Upload

↓

Download Link

Benefits:

* Handles huge datasets
* Minimal RAM usage
* Fast export generation

---

# Common Interview Questions

### What is a stream?

A mechanism for processing data incrementally instead of loading everything into memory.

---

### Why are streams important?

They enable efficient processing of large datasets while maintaining low memory usage.

---

### What is backpressure?

A flow-control mechanism that prevents fast producers from overwhelming slow consumers.

---

### Difference between Readable and Writable streams?

Readable streams consume data.

Writable streams receive data.

---

### What are Transform streams?

Streams that modify data while it is flowing through the pipeline.

---

### Why are streams useful in distributed systems?

They enable scalable processing of large data volumes with predictable resource usage.

---

# Key Takeaways

* Streams are a core scalability feature of Node.js.
* They allow chunk-based processing.
* Backpressure prevents memory overload.
* Pipe simplifies stream composition.
* Transform streams enable real-time data processing.
* Streams are heavily used in production systems involving files, APIs, Kafka, Redis, RabbitMQ, and cloud storage.
* Senior backend engineers should be comfortable designing stream-based architectures.

---

# Related Documents

* docs/fundamentals/nodejs-runtime.md
* docs/fundamentals/event-loop.md
* docs/fundamentals/clustering.md
* docs/fundamentals/worker-threads.md
* docs/architecture/file-storage-system.md

Related Diagram:

* diagrams/nodejs-runtime.mmd
