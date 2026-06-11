# 🛍️ The Zudio Incident — Part A + Part B: Bugs Fixed + Architecture Redesign

## 📖 Overview

This project simulates a real-world production incident in the backend of a fast-fashion e-commerce platform similar to Zudio.

The repository contains:

* Production bug investigation and remediation
* Security vulnerability fixes
* Performance optimization and benchmarking
* Database redesign and normalization
* Scalable cloud architecture design
* Complete API contract documentation

The goal was to analyze a broken Node.js + PostgreSQL backend, identify critical production issues, fix them systematically, and redesign the system to support large-scale traffic.

---

## 🚨 Incident Scenario

At 11 PM on a Friday night, multiple production issues were reported:

* Payment endpoint returning random 500 errors
* SQL Injection vulnerability exposing sensitive data
* Order history endpoint taking 14+ seconds to load
* Discount coupons applied multiple times
* Product stock not decrementing after purchases
* Passwords stored in plaintext

The objective was to investigate, profile, fix, verify, and redesign the system.

---

# ✅ Part A — Production Refactor & Bug Fixes

## Bugs Identified and Fixed

| # | Bug                                   | Category    | Severity |
| - | ------------------------------------- | ----------- | -------- |
| 1 | SQL Injection in Product Search       | Security    | Critical |
| 2 | Plaintext Password Storage            | Security    | Critical |
| 3 | Double Coupon Application             | Logic       | High     |
| 4 | Stock Not Decrementing After Checkout | Logic       | High     |
| 5 | N+1 Query in Order History            | Performance | High     |

---

## Security Improvements

### SQL Injection Prevention

**Before**

```sql
SELECT * FROM products
WHERE name LIKE '%${search}%'
```

**After**

```sql
SELECT * FROM products
WHERE name ILIKE $1
```

Implemented parameterized PostgreSQL queries throughout the application.

---

### Password Hashing

Added bcrypt password hashing with secure salt rounds.

```javascript
const hashedPassword = await bcrypt.hash(password, 12);
```

Passwords are no longer stored in plaintext.

---

## Logic Fixes

### Coupon Race Condition

Implemented atomic coupon validation and usage tracking using PostgreSQL transactional updates.

### Inventory Synchronization

Added stock decrement logic within checkout transactions to ensure inventory consistency.

---

## Performance Improvements

### Order History Optimization

Replaced N+1 query pattern with optimized JOIN queries.

**Before**

* 200+ Queries
* ~14 seconds response time

**After**

* 1–2 Queries
* <500ms response time

---

## Verification

All fixes were manually verified through endpoint testing and profiling.

| Feature                   | Status |
| ------------------------- | ------ |
| SQL Injection Protection  | ✅      |
| Password Hashing          | ✅      |
| Coupon Protection         | ✅      |
| Stock Updates             | ✅      |
| Order History Performance | ✅      |

---

# 🏗️ Part B — Architecture Redesign

## Objective

Redesign the backend architecture to support large-scale traffic while maintaining performance, reliability, and security.

Target scale:

* 100,000+ concurrent users
* High checkout throughput
* Fault tolerance
* Read-heavy traffic optimization

---

## Proposed Architecture

### Infrastructure Components

* CDN for static assets and product images
* Application Load Balancer
* Multiple Stateless Node.js Instances
* Redis Cluster
* PostgreSQL Primary Database
* PostgreSQL Read Replicas
* Connection Pooling
* Transaction-based Checkout System

---

## Architecture Improvements

### Redis Cache

Added for:

* Product catalog caching
* Coupon locking
* Reduced database load

### Read Replicas

Added for:

* Product browsing
* Order history queries

### Load Balancer

Added to eliminate the single point of failure and distribute traffic across instances.

---

# 🗄️ Database Redesign

Implemented:

* Fully normalized schema
* Foreign key constraints
* NOT NULL constraints
* CHECK constraints
* Composite indexes
* Transaction-safe operations

Example:

```sql
CHECK (stock >= 0)
CHECK (quantity > 0)
CHECK (total >= 0)
```

---

# 📑 API Documentation

Comprehensive REST API contracts were created covering:

* Authentication
* Products
* Cart
* Checkout
* Orders
* Coupons

Each endpoint includes:

* Request schema
* Success responses
* Error responses
* Validation rules
* Authentication requirements

---

# 📊 Benchmark Results

## Optimization Implemented

Order History Query Optimization

| Metric        | Before | After  |
| ------------- | ------ | ------ |
| Response Time | ~14s   | <500ms |
| Query Count   | 200+   | 1–2    |
| Database Load | High   | Low    |

---

# 🛠️ Tech Stack

### Backend

* Node.js
* Express.js

### Database

* PostgreSQL

### Caching

* Redis

### Security

* bcrypt
* Parameterized SQL Queries

### Infrastructure

* AWS Architecture Design
* Load Balancing
* Read Replicas
* CDN

---

# 📂 Repository Structure

```text
.
├── src/
├── migrations/
├── seeds/
├── part-b/
│   ├── ARCHITECTURE.md
│   ├── SCHEMA.md
│   ├── API-CONTRACTS.md
│   └── BENCHMARK.md
├── AUDIT.md
└── README.md
```

---

# 🎯 Learning Outcomes

This project demonstrates:

* Production debugging
* Incident analysis
* Secure coding practices
* Database optimization
* Query performance tuning
* Architecture scalability design
* Reliability engineering concepts

---

## PR Title

**The Zudio Incident — Part A + Part B: Bugs Fixed + Architecture Redesign**

---

### Author

Rahul B

Production Engineering • Backend Development • System Design
