---

layout: post
title: "Go Concurrency, Resilience & Architecture Patterns"
date: 2026-04-17 07:36:21 +0000
tags: [go, concurrency, system-design, resilience, architecture]
categories: go
---

## Overview

This guide covers essential **Go concurrency, resilience, and architecture patterns** used in real-world systems:

* Concurrency patterns (fan-out, fan-in, channels)
* Event systems (observer, broadcasting)
* State management (mutex vs channels)
* Reliability patterns (retry, circuit breaker)
* System protection (rate limiting, bulkheads)
* Architecture & testing patterns

Each section focuses on practical tradeoffs and when to use each approach.

---

# 🔀 Fan-Out & Fan-In Patterns

## Fan-Out

Fan-out distributes work across multiple goroutines.

```go
jobs := make(chan int)

for i := 0; i < 3; i++ {
    go worker(i, jobs)
}
```

### Insight

> Parallelism improves throughput—but must be controlled.

---

## Fan-In

Fan-in merges results from multiple goroutines.

```go
func merge(ch1, ch2 <-chan int) <-chan int
```

### Insight

> Always merge concurrently to avoid blocking.

---

## Combined Pattern

```
tasks → fan-out → workers → fan-in → result
```

---

# 📡 Observer vs Broadcasting

## Channel (Queue Model)

* One event → one consumer
* Competing consumers model

## Broadcasting

* One event → many consumers
* Requires manual fan-out

### Insight

> Channels distribute work. Broadcasting distributes events.

---

# 🔒 Mutex vs Channel Ownership

## Mutex

* Shared state with locks

## Channel Ownership

* Single goroutine owns state
* Others communicate via messages

### Insight

> Mutex = shared memory
> Channel = message passing

---

# ⏱ Rate Limiting & Token Bucket

## Rate Limiting

* Controls request rate

## Token Bucket

* Allows bursts + steady flow

### Insight

> Token bucket = time-based resource allocator

---

# 🔁 Retry vs Circuit Breaker

## Retry

* Handles transient failures

## Circuit Breaker

* Stops repeated failures

### Insight

> Retry is optimistic. Circuit breaker is protective.

---

# 🧱 Bulkhead & Worker Isolation

## Bulkhead

* Isolate resources per dependency

## Worker Pools

* Separate workloads by type

### Insight

> Prevent noisy neighbors from impacting critical work.

---

# 🏗 Layered Architecture & Boundaries

## Layers

```
Handler → Service → Repository
```

## Boundaries

* Interfaces at edges
* DTO separation

### Insight

> Good boundaries prevent system chaos.

---

# 🧪 Testing Patterns

## Table-Driven Tests

* Data-driven testing

## Interfaces for Fakes

* Replace dependencies

### Insight

> Interfaces are test seams.

---

# 🧠 Functional Core / Imperative Shell

## Functional Core

* Pure logic

## Imperative Shell

* IO and orchestration

### Insight

> Separate logic from side effects.

---

# 🔌 Strategy vs Function Injection

## Strategy (Interfaces)

* Explicit behavior contracts

## Function Injection

* Lightweight, flexible

### Insight

> Functions are often the simplest strategy.

---

# 🧠 Final Principles

* Control concurrency explicitly
* Isolate failures early
* Prefer simple patterns first
* Separate concerns cleanly
* Optimize for clarity and predictability

> Great Go systems are not just concurrent — they are controlled, resilient, and understandable.

<!-- gh-pages-taxonomy-links:start -->
Categories: [go](/categories/go/)
Tags: [go](/tags/go/), [concurrency](/tags/concurrency/), [system-design](/tags/system-design/), [resilience](/tags/resilience/), [architecture](/tags/architecture/)
<!-- gh-pages-taxonomy-links:end -->
