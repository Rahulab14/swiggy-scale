<div align="center">

<img src="assets/image.png" alt="Swiggy Scale Simulation Banner" width="100%">

# 🚀 Swiggy Scale Simulation

### From Monolith Meltdown to 10 Million Users at Scale

**Failure Cascade Analysis • Scalable Architecture • AWS Cost Estimation • Incident Runbook**



</div>

---

# 📖 Overview

This repository is a comprehensive **Site Reliability Engineering (SRE)** and **System Design** case study that analyzes how a Swiggy-like food delivery platform behaves under extreme traffic conditions and how to redesign the system to survive massive scale.

The project simulates a real-world scenario:

> **India vs Pakistan World Cup Final**
>
> Swiggy launches a **50% OFF promotion** to its entire user base.
>
> **180 million notifications** are delivered.
>
> **10 million users** attempt to order simultaneously.
>
> The backend consists of:
>
> - Single Node.js server
> - Single PostgreSQL database
> - No cache
> - No CDN
> - No Load Balancer
> - Synchronous payments

The goal is to determine:

- What breaks first?
- Why does it break?
- What is the financial impact?
- How should the architecture be redesigned?
- What would the new infrastructure cost?
- How should engineers respond during the incident?

---

# 🎯 Project Objectives

This repository demonstrates:

- Failure Cascade Analysis
- Capacity Planning
- Distributed System Design
- Cloud Architecture Design
- AWS Cost Estimation
- Incident Response Engineering
- Production Runbook Creation

---

# 📊 Traffic Simulation

### Promotion Blast

```text
Notification Recipients: 180,000,000
Click Through Rate:      8%
Active Users:            14,400,000
```

### User Behaviour

Each user performs approximately:

```text
GET /restaurants
GET /restaurant/:id
POST /orders
```

Average:

```text
3 API requests per user
```

### Peak Request Rate

```text
Peak RPS =
(10,000,000 × 3) / 60

= 500,000 Requests/Second
```

---

# 🚨 Key Findings

## 1. PostgreSQL Dies First

The database is configured as:

```text
max_connections = 100
```

Synchronous payment processing holds connections for:

```text
200ms – 2000ms
```

The connection pool is exhausted within seconds.

---

## 2. Node.js Is Overloaded by 41×

Single server capacity:

```text
≈ 12,000 RPS
```

Traffic demand:

```text
≈ 500,000 RPS
```

Result:

```text
500,000 / 12,000

≈ 41x overload
```

---

## 3. Payment Calls Amplify Failure

Each order:

```text
Acquire DB Connection
↓
Call Payment Gateway
↓
Wait 200–2000ms
↓
Release Connection
```

This dramatically reduces effective throughput.

---

## 4. Promo Code Race Condition

Current flow:

```text
SELECT promo
UPDATE promo
```

Under heavy concurrency:

```text
Millions validated
Before updates commit
```

Result:

- Promo oversubscription
- Revenue leakage
- Customer complaints

---

## 5. Images Can Crash The Platform

Without a CDN:

```text
10M users
× 20 images
× 200 KB
```

Produces:

```text
≈ 40 TB
```

of image traffic in the first minute.

The application server becomes network-bound before processing business requests.

---

# 📂 Repository Structure

```text
swiggy-scale/
│
├── README.md
│
├── docs/
│   ├── FAILURE-CASCADE.md
│   ├── ARCHITECTURE.md
│   ├── COST-ESTIMATE.md
│   └── RUNBOOK.md
│
└── assets/
    └── image.png
```

---

# 📚 Documentation

| Document | Description |
|-----------|-------------|
| FAILURE-CASCADE.md | Full failure cascade analysis including RPS calculations, bottlenecks, failure triggers, and incident timeline |
| ARCHITECTURE.md | Redesigned scalable architecture with CDN, Redis, ALB, PgBouncer, Read Replicas, and SQS |
| COST-ESTIMATE.md | AWS infrastructure pricing with baseline and peak-event calculations |
| RUNBOOK.md | Production incident response guide with alarms, triage, recovery, rollback, and postmortem process |

---

# 🏗 Architecture Overview

## Existing Monolith

```text
Users
  │
  ▼
Node.js
  │
  ▼
PostgreSQL
```

Problems:

- Single point of failure
- No horizontal scaling
- No caching
- No connection pooling
- No async processing
- No CDN

---

## Redesigned Architecture

```text
Users
  │
  ▼
CloudFront CDN
  │
  ▼
Application Load Balancer
  │
  ▼
Auto Scaling Node.js Fleet
  │
  ├─────────────┐
  ▼             ▼
Redis       SQS Queue
  │             │
  ▼             ▼
PgBouncer   Payment Workers
  │
  ▼
PostgreSQL Primary
  │
  ├── Read Replica 1
  └── Read Replica 2
```

---

# 🔧 Components Introduced

| Component | Purpose |
|------------|----------|
| CloudFront | Offloads static assets and images |
| ALB | Load balancing and health checks |
| Auto Scaling | Dynamic capacity during traffic spikes |
| Redis | Menu caching and promo locking |
| PgBouncer | Database connection pooling |
| PostgreSQL Replicas | Read scaling |
| SQS | Asynchronous payment processing |
| Payment Workers | Background payment execution |

---

# 💰 Cost Summary

## Baseline Infrastructure

| Service | Monthly Cost |
|----------|--------------|
| EC2 Fleet | $119 |
| PostgreSQL Primary | $131 |
| Read Replicas | $262 |
| Redis Cluster | $358 |
| ALB | $56 |
| CloudFront | $85 |
| SQS | $12 |
| **Total** | **~$1,024/month** |

---

## World Cup Final Scaling

Additional 4-hour event cost:

```text
≈ $455
```

---

## Business Justification

Revenue loss estimate:

```text
₹4.2 Crore / Minute
```

45-minute outage:

```text
₹189 Crore
```

Infrastructure investment:

```text
~$1,024 / Month
```

The cost of prevention is insignificant compared to the cost of downtime.

---

# 📋 Incident Response

The repository includes a complete operational runbook covering:

- Alert thresholds
- Incident detection
- Triage decision tree
- Component-specific remediation
- Rollback procedures
- Postmortem template

Designed for:

```text
Junior Engineers
On-Call Engineers
SRE Teams
DevOps Teams
Platform Engineers
```

---

# 🛠 Technologies Covered

### Backend

- Node.js
- Express.js

### Database

- PostgreSQL
- Read Replicas
- PgBouncer

### Caching

- Redis
- ElastiCache

### Messaging

- Amazon SQS

### Infrastructure

- AWS EC2
- AWS ALB
- AWS CloudFront
- AWS CloudWatch
- AWS Auto Scaling

### Reliability Engineering

- Capacity Planning
- Incident Response
- Failure Analysis
- Runbook Design
- Cost Optimization

---

# 🎓 Learning Outcomes

After studying this repository you will understand:

- How to calculate system capacity limits
- How bottlenecks create cascading failures
- How to scale Node.js applications
- How to scale PostgreSQL databases
- How Redis reduces database load
- How asynchronous systems improve reliability
- How to estimate AWS infrastructure costs
- How to write production-grade runbooks

---

# 🚀 Final Takeaway

The most important lesson from this project is:

> **Systems do not fail randomly. They fail at their bottlenecks.**

In this scenario:

```text
100 PostgreSQL Connections
+
Synchronous Payment Calls
+
500,000 RPS
```

creates an unavoidable failure cascade:

```text
Database Pool Exhaustion
          ↓
Request Queueing
          ↓
Node.js Saturation
          ↓
Timeouts
          ↓
Application Crash
          ↓
Revenue Loss
```

The redesigned architecture removes each bottleneck individually and transforms a fragile monolith into a resilient, scalable platform capable of handling major sporting-event traffic surges.

---

<div align="center">

### ⭐ If you found this project useful, consider starring the repository.

Built with System Design, SRE, Scalability Engineering, and Production Reliability principles.

</div>