---

layout: post
title: "Hedged Requests & Timeout Racing in Go"
date: 2026-04-21 00:00:00 +0000
tags: [go, distributed-systems, concurrency, latency, system-design]
categories: go
---

# 🚗 Pattern 1: Hedged Requests

## 🧠 Idea

If a request is taking too long:

* 👉 Send a duplicate request to another replica
* 👉 Use whichever finishes first
* 👉 Cancel the rest

---

## ✅ Go Example

```go
func hedgedRequest(ctx context.Context, fn func(context.Context) (string, error)) (string, error) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()

    resultCh := make(chan string, 2)

    // first request
    go func() {
        res, _ := fn(ctx)
        select {
        case resultCh <- res:
        case <-ctx.Done():
        }
    }()

    // hedge after delay
    time.AfterFunc(50*time.Millisecond, func() {
        go func() {
            res, _ := fn(ctx)
            select {
            case resultCh <- res:
            case <-ctx.Done():
            }
        }()
    })

    select {
    case res := <-resultCh:
        cancel() // cancel the slower one
        return res, nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}
```

---

## 🌍 Real-World Usage

* Google search systems (classic paper)
* distributed RPC systems
* multi-replica database reads
* CDN edge fetches

In Go:

* common in high-performance clients
* layered on top of `context` cancellation

---

## ⚠️ Common Misunderstanding

“**Hedging always improves performance**”

❌ It improves **tail latency**, not average latency.

Tradeoff:

* latency ↓
* resource usage ↑

---

## 🧠 Elite Secret

The delay before hedging is critical.

Too early:

* doubles load unnecessarily

Too late:

* no benefit

👉 Often set to **p95 latency**

---

## 🔁 Breakthrough Reframe

Hedging is not retrying.

It’s:

> **racing uncertainty**

---

# ⏱️ Pattern 2: Timeout Racing

## 🧠 Idea

Run multiple strategies with different timeouts and **race them**.

Example:

* fast cache (short timeout)
* slow DB (long timeout)

---

## ✅ Go Example

```go
func timeoutRace(ctx context.Context) (string, error) {
    ctx, cancel := context.WithCancel(ctx)
    defer cancel()

    resultCh := make(chan string, 2)

    // fast path (cache)
    go func() {
        cctx, cancel := context.WithTimeout(ctx, 20*time.Millisecond)
        defer cancel()

        if res, err := cacheCall(cctx); err == nil {
            resultCh <- res
        }
    }()

    // slow path (DB)
    go func() {
        cctx, cancel := context.WithTimeout(ctx, 100*time.Millisecond)
        defer cancel()

        if res, err := dbCall(cctx); err == nil {
            resultCh <- res
        }
    }()

    select {
    case res := <-resultCh:
        cancel()
        return res, nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}
```

---

## 🌍 Real-World Usage

* cache + database fallback
* multi-region requests
* different consistency levels (fast vs strong)
* read replicas vs primary DB

Go makes this natural with:

* goroutines
* `context`
* `select`

---

## ⚠️ Common Misunderstanding

“**This is just retry logic**”

❌ No.

Retries are sequential.
This is **parallel competition**.

---

## 🧠 Elite Secret

Timeout racing works best when:

* strategies have **different latency profiles**
* failure modes are independent

👉 Otherwise you just duplicate failure

---

## 🔁 Breakthrough Reframe

Timeout racing is not fallback.

It’s:

> **parallel exploration of possible success paths**

---

# ⚔️ Hedged Requests vs Timeout Racing

## Pros

### Hedged Requests

* Reduces tail latency (p99)
* Handles unpredictable slow replicas
* Simple mental model

### Timeout Racing

* Exploits different data sources
* Optimizes for fastest path
* Flexible architecture

---

## Cons

### Hedged Requests

* Doubles load
* Can amplify traffic
* Needs careful tuning

### Timeout Racing

* More complex logic
* Requires multiple strategies
* Harder to debug

---

## When to Use Hedged Requests

* identical replicas
* flaky latency
* distributed systems

---

## When to Use Timeout Racing

* multiple data sources (cache, DB, API)
* different latency guarantees
* fallback strategies

---

## When to Combine Them (🔥 Real World)

```text
Client Request
    ↓
Timeout Race (cache vs DB)
    ↓
Hedged Requests (within DB replicas)
    ↓
Fastest Result Wins
```

👉 This is how large-scale systems get **insanely low latency**.

---

# 🧠 Deep Go Tradeoff Insight

These patterns highlight Go’s strength:

> Concurrency is cheap enough to *compete strategies in parallel*

But also the danger:

* more goroutines
* more allocations
* more downstream pressure

👉 You must pair this with:

* context cancellation
* rate limiting
* load shedding