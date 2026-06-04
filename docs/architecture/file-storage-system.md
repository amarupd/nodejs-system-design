# File Storage System Architecture

![Architecture](../../assets/deployment.png)

---

# Overview

Modern applications handle massive amounts of file data.

Examples:

* Product Images
* User Avatars
* Videos
* PDFs
* Invoices
* KYC Documents
* Match Highlights
* Media Assets

A production-grade file storage system must be:

* Scalable
* Durable
* Cost-efficient
* Secure
* Highly Available

Storing files incorrectly can result in:

* Slow APIs
* High infrastructure costs
* Data loss
* Poor user experience

This document covers production-grade file storage architectures used in large-scale systems.

---

# Why Not Store Files in Databases?

A common beginner mistake.

Bad Architecture:

```text
Application
    ↓
MySQL
    ↓
Images
Videos
PDFs
```

Problems:

* Database growth
* Slow backups
* Expensive storage
* Increased query latency
* Difficult scaling

Databases should store metadata, not large binary files.

---

# Recommended Architecture

```text
Client
   ↓
Upload Service
   ↓
Object Storage
   ↓
CDN
   ↓
Users
```

Metadata:

```text
Database
```

Stores:

* File URL
* Owner
* Type
* Size
* Status

---

# High-Level Architecture

```text
Client
   ↓
API Gateway
   ↓
Upload Service
   ↓
Object Storage
   ↓
CDN
```

Examples:

Object Storage:

* AWS S3
* Google Cloud Storage
* Azure Blob Storage

CDN:

* CloudFront
* Cloudflare
* Akamai

---

# Core Components

## Upload Service

Responsibilities:

* File validation
* Virus scanning
* Metadata generation
* Storage integration

---

## Object Storage

Stores actual files.

Benefits:

* Durable
* Scalable
* Cost-effective

---

## Metadata Database

Stores:

```text
file_id
owner_id
file_url
mime_type
size
created_at
```

---

## CDN

Caches files closer to users.

Benefits:

* Lower latency
* Reduced storage traffic
* Better scalability

---

# Upload Flow

```text
User
 ↓
Upload Request
 ↓
Upload Service
 ↓
Store File
 ↓
Store Metadata
 ↓
Return URL
```

---

# File Metadata Example

```json
{
  "id": "FILE_101",
  "url": "https://cdn.example.com/files/a.jpg",
  "size": 204800,
  "mime_type": "image/jpeg"
}
```

---

# Direct Upload Architecture

Recommended approach.

Traditional:

```text
Client
 ↓
Backend
 ↓
S3
```

Problem:

Backend becomes bottleneck.

---

Modern:

```text
Client
 ↓
Pre-Signed URL
 ↓
S3
```

Benefits:

* Lower backend load
* Better scalability
* Faster uploads

---

# Pre-Signed URL Flow

```text
Client
 ↓
Request Upload URL
 ↓
Backend
 ↓
Generate Signed URL
 ↓
Client Uploads Directly
 ↓
Storage
```

Backend never handles the file itself.

---

# Large File Uploads

Examples:

* Videos
* Data exports
* Media archives

Challenges:

* Timeouts
* Network failures
* Large memory usage

---

# Multipart Upload

Solution:

```text
File
 ↓
Part 1
Part 2
Part 3
Part N
```

Each part uploads independently.

Benefits:

* Resume capability
* Parallel uploads
* Better reliability

---

# File Validation

Validate:

## File Type

Examples:

```text
jpg
png
pdf
mp4
```

---

## File Size

Example:

```text
Max 10 MB
```

---

## File Content

Prevent malicious uploads.

---

# Virus Scanning

Important for:

* User uploads
* Documents
* KYC systems

Architecture:

```text
Upload
 ↓
Quarantine
 ↓
Virus Scan
 ↓
Approved
```

Tools:

* ClamAV
* Cloud-based scanners

---

# Image Processing Pipeline

Example:

```text
Upload
 ↓
Queue
 ↓
Worker
 ↓
Resize
 ↓
Compress
 ↓
Generate Thumbnail
 ↓
Store
```

Benefits:

* Faster user experience
* Reduced bandwidth

---

# Video Processing Pipeline

Architecture:

```text
Upload
 ↓
Queue
 ↓
Transcoding Workers
 ↓
Multiple Formats
 ↓
Storage
 ↓
CDN
```

Common outputs:

```text
360p
720p
1080p
4K
```

---

# Event-Driven File Processing

Modern systems use queues.

Example:

```text
Upload Complete
      ↓
Kafka Event
      ↓
Media Service
      ↓
Thumbnail Generation
```

Benefits:

* Decoupled architecture
* Better scalability

---

# Storage Tiering

Frequently accessed files:

```text
Hot Storage
```

Rarely accessed files:

```text
Cold Storage
```

Example:

```text
AWS S3

↓

Glacier
```

Benefits:

* Cost optimization

---

# CDN Architecture

```text
User
 ↓
CDN Edge
 ↓
Storage
```

Benefits:

* Global performance
* Reduced origin traffic

---

# File Download Architecture

```text
User
 ↓
CDN
 ↓
Storage
```

Backend typically not involved.

Benefits:

* Reduced infrastructure load

---

# Access Control

Not every file should be public.

Options:

## Public Files

Examples:

* Product Images
* Marketing Assets

---

## Private Files

Examples:

* Invoices
* KYC Documents
* Medical Records

---

# Secure File Access

Flow:

```text
User
 ↓
Authentication
 ↓
Generate Signed URL
 ↓
Temporary Access
```

Benefits:

* Strong security
* Controlled access

---

# File Deletion Strategy

Soft Delete:

```text
Marked Deleted
```

Benefits:

* Recovery possible

---

Hard Delete:

```text
Permanent Removal
```

Use carefully.

---

# Ecommerce Example

Files:

```text
Product Images
Product Videos
Size Charts
```

Architecture:

```text
Admin
 ↓
Upload Service
 ↓
S3
 ↓
CloudFront
 ↓
Customers
```

Benefits:

* Fast product browsing
* Lower server load

---

# Fantasy Sports Example

Files:

```text
Team Logos
Player Images
Match Media
```

Architecture:

```text
Media Service
 ↓
Object Storage
 ↓
CDN
```

---

# KYC Document Example

Files:

```text
PAN Card
Passport
Driving License
```

Requirements:

* Encryption
* Access Control
* Audit Logs

Architecture:

```text
Private Storage
 ↓
Signed URLs
```

---

# Monitoring Metrics

Track:

## Upload Success Rate

Reliability indicator.

---

## Storage Usage

Capacity planning.

---

## CDN Hit Ratio

Performance indicator.

---

## File Processing Time

Worker efficiency.

---

## Failed Uploads

User experience metric.

---

# Security Best Practices

## Validate Files

Never trust uploads.

---

## Scan Files

Prevent malware.

---

## Encrypt Sensitive Files

Protect user data.

---

## Use Signed URLs

Avoid permanent public access.

---

## Audit Access

Track downloads and uploads.

---

# Common Mistakes

## Storing Files in Databases

Creates scaling problems.

---

## No Validation

Security risk.

---

## No CDN

Poor performance.

---

## No Virus Scanning

Potential malware exposure.

---

## Public Sensitive Files

Major security issue.

---

# Common Interview Questions

### Why not store files in MySQL?

Large files degrade database performance and scalability.

---

### What is object storage?

A scalable storage system designed for large file workloads.

---

### Why use pre-signed URLs?

To allow direct uploads without burdening backend servers.

---

### Why use a CDN?

To reduce latency and improve global performance.

---

### What is multipart upload?

Splitting large files into smaller uploadable chunks.

---

### Why separate metadata from file storage?

Improves scalability and maintainability.

---

# Production Lessons

* Store metadata in databases and files in object storage.
* Use pre-signed URLs for scalable uploads.
* Process large files asynchronously.
* Use CDNs aggressively.
* Encrypt sensitive files.
* Scan uploads before use.
* Monitor storage and processing metrics.

---

# Key Takeaways

* File storage systems require specialized architecture.
* Object storage is the industry standard.
* Databases should only store metadata.
* CDNs dramatically improve performance.
* Pre-signed URLs improve scalability.
* Security and access control are critical.
* Production-grade file systems are foundational infrastructure components.

---

# Related Documents

* docs/fundamentals/streams.md
* docs/architecture/distributed-systems.md
* docs/architecture/event-driven-architecture.md
* docs/architecture/notification-system.md

Related Diagram:

* diagrams/deployment-architecture.mmd
