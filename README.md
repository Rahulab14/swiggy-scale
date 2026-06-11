# Swiggy Scale Simulation & Incident Architecture

A deep-dive Site Reliability Engineering (SRE) case study that analyzes how a Swiggy-like food delivery platform would behave during a massive traffic spike triggered by a World Cup Final promotion, and how to redesign the system to survive 10 million concurrent users.

> Scenario: India vs Pakistan World Cup Final, 8 PM IST.  
> 50% discount notification sent to 180 million users.  
> 10+ million users attempt to order simultaneously.  
> Existing backend: Single Node.js server + Single PostgreSQL database.  
> Question: Can it survive? If not, what architecture should replace it?

Source material: :contentReference[oaicite:0]{index=0} :contentReference[oaicite:1]{index=1}

---

## Repository Structure

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
```

---

## Project Objective

This repository demonstrates:

- Failure cascade analysis under extreme load
- Capacity planning using real infrastructure numbers
- High-scale architecture design
- AWS cost estimation
- Production incident response procedures

The goal is not to build code.

The goal is to answer:

> "If 10 million users arrive at once, what breaks first, why does it break, how much does it cost, and how do we prevent it?"

---

## Scenario Overview

### Existing Production System

```text
Users
  │
  ▼
Node.js Express
(1 CPU, 4GB RAM)
  │
  ▼
PostgreSQL
(max_connections = 100)
```

Characteristics:

- Single Node.js process
- Single PostgreSQL instance
- No Redis
- No CDN
- No Load Balancer
- No Auto Scaling
- Synchronous payment processing
- Images served directly from application server

---

## Traffic Simulation

### Notification Blast

```text
Notification recipients      = 180,000,000
CTR                           = 8%

Users opening app            = 14,400,000
```

### API Activity

Each user performs:

1. Browse restaurants
2. Open restaurant
3. Place order

```text
3 API calls/user
```

### Peak RPS

```text
Peak RPS =
(10,000,000 × 3) / 60

= 500,000 requests/sec
```

---

## Key Findings

### 1. PostgreSQL Dies First

```text
max_connections = 100
```

Synchronous payment calls hold database connections for up to:

```text
200ms – 2000ms
```

Connection pool exhaustion occurs within seconds.

---

### 2. Node.js Is Asked To Handle 41× Capacity

```text
Node capacity ≈ 12,000 RPS

Traffic demand ≈ 500,000 RPS
```

Result:

```text
500,000 / 12,000

≈ 41x overload
```

---

### 3. Payment Calls Amplify Failure

Every payment request:

```text
DB connection
    +
External API wait
```

This dramatically increases connection hold time and exhausts PostgreSQL far earlier than expected.

---

### 4. Promo Code Logic Overspends Budget

Classic race condition:

```text
SELECT promo
UPDATE promo
```

Under massive concurrency:

```text
Millions validated
Before first updates commit
```

Result:

- Oversold discounts
- Revenue loss
- Customer disputes

---

### 5. Images Alone Can Kill The Server

```text
10M users
× 20 images
× 200 KB
```

≈ 40 TB of transfer in one minute.

Without a CDN:

```text
Node.js NIC saturates
Before APIs finish loading
```

---

## Documents Included

| Document | Description |
|-----------|-------------|
| FAILURE-CASCADE.md | Complete incident simulation with traffic math, component limits, trigger points, and timeline |
| ARCHITECTURE.md | Redesigned multi-tier architecture capable of handling event-scale traffic |
| COST-ESTIMATE.md | AWS pricing model including baseline and World Cup peak scaling |
| RUNBOOK.md | Production incident runbook for on-call engineers |

---

## Architecture Overview

The redesigned platform introduces:

```text
CloudFront CDN
        │
        ▼
Application Load Balancer
        │
        ▼
Node.js Auto-Scaling Fleet
        │
        ▼
Redis Cluster
        │
        ▼
PgBouncer
        │
        ▼
PostgreSQL Primary
        │
        ├── Read Replica 1
        └── Read Replica 2

SQS Payment Queue
        │
        ▼
Payment Workers
```

### What Changes?

| Problem | Solution |
|----------|----------|
| Image traffic overload | CloudFront CDN |
| Single server failure | Load Balancer + Auto Scaling |
| DB connection exhaustion | Redis + PgBouncer |
| Payment latency | SQS asynchronous processing |
| Read/write contention | PostgreSQL replicas |
| Promo race condition | Redis atomic locking |

---

## Cost Summary

### Baseline Infrastructure

Approximate monthly cost:

```text
~$1,024/month
```

Includes:

- EC2
- RDS
- Read Replicas
- Redis Cluster
- CloudFront
- ALB
- SQS

---

### Peak Event Scaling

Additional cost for a 4-hour World Cup Final event:

```text
~$455
```

---

### Cost vs Outage

Estimated business loss:

```text
₹4.2 crore/minute
```

45-minute outage:

```text
₹189 crore
```

Infrastructure cost:

```text
~$1,024/month
```

Conclusion:

```text
Infrastructure is dramatically cheaper
than downtime.
```

---

## Technologies Covered

### Application Layer

- Node.js
- Express.js

### Data Layer

- PostgreSQL
- Read Replicas
- PgBouncer

### Cache Layer

- Redis
- ElastiCache

### Messaging

- Amazon SQS

### Infrastructure

- EC2
- ALB
- CloudFront
- CloudWatch
- Auto Scaling Groups

### Reliability Engineering

- Capacity Planning
- Failure Analysis
- Incident Response
- Cost Optimization
- Runbook Design

---

## Expected Learning Outcomes

After reading this repository you should be able to:

- Calculate peak traffic loads
- Predict failure cascades
- Identify infrastructure bottlenecks
- Design scalable distributed systems
- Estimate AWS operating costs
- Write production-grade runbooks
- Justify architecture decisions with numbers

---

## Final Takeaway

The most important lesson is:

> Systems do not fail randomly. They fail at their bottlenecks.

In this scenario:

```text
100 PostgreSQL connections
+
Synchronous payment calls
+
500,000 RPS demand
```

creates an unavoidable cascade:

```text
DB Pool Exhaustion
        ↓
Request Queueing
        ↓
Node Saturation
        ↓
Timeouts
        ↓
Process Crash
        ↓
Revenue Loss
```

The redesigned architecture removes each bottleneck individually and replaces a fragile monolith with a resilient, horizontally scalable platform capable of surviving major sporting-event traffic spikes.