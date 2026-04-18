---

layout: post
title: "Adaptive Concurrency: AIMD vs Latency-Based Feedback Control"
date: 2026-04-18 10:00:00 +0000
tags: [distributed-systems, concurrency, control-theory, golang, system-design]
categories: go
---

# 📈 Pattern 1: Adaptive Concurrency (AIMD)

## 🧠 Idea

Adjust concurrency based on success/failure:

* ✅ Success → increase concurrency slowly
* ❌ Failure → decrease quickly

This is called **AIMD (Additive Increase, Multiplicative Decrease)**.

## ✅ Go Example

```go
type AIMD struct {
    mu    sync.Mutex
    limit int
    max   int
}

func NewAIMD(initial, max int) *AIMD {
    return &AIMD{
        limit: initial,
        max:   max,
    }
}

func (a *AIMD) Allow() bool {
    a.mu.Lock()
    defer a.mu.Unlock()

    if a.limit <= 0 {
        return false
    }
    a.limit--
    return true
}

func (a *AIMD) Done(success bool) {
    a.mu.Lock()
    defer a.mu.Unlock()

    if success {
        if a.limit < a.max {
            a.limit++ // slow increase
        }
    } else {
        a.limit /= 2 // fast decrease
    }
}
```

### Usage

```go
if !aimd.Allow() {
    return errors.New("overloaded")
}

err := callService()
aimd.Done(err == nil)
```

## 🌍 Real-World Usage

* TCP congestion control (the original AIMD!)
* gRPC adaptive concurrency
* Envoy proxy load balancing
* High-throughput Go services auto-tuning limits

### Why Go fits well

* Explicit control
* No hidden feedback loops
* Easy integration with request lifecycle

## ⚠️ Common Misunderstanding

> "AIMD finds the perfect limit"

❌ Not exactly.

It **oscillates around optimal capacity**.

That’s intentional:

* Too stable → slow adaptation
* Too reactive → instability

## 🧠 Elite Secret

AIMD works best when your signal is **latency**, not just errors.

Why?

* Errors are late signals
* Latency increases *before* failure

👉 Advanced systems reduce concurrency when latency rises

## 🔁 Breakthrough Reframe

AIMD is not a limiter.

It’s:

> A learning system probing system capacity

---

# 📊 Pattern 2: Feedback Control (Latency-Based)

## 🧠 Idea

Instead of reacting to success/failure...

👉 Use **observed latency** as feedback.

* Latency ↑ → reduce concurrency
* Latency ↓ → increase concurrency

## ✅ Go Example (simplified)

```go
type Controller struct {
    mu        sync.Mutex
    limit     int
    targetLat time.Duration
}

func (c *Controller) Adjust(observed time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()

    if observed > c.targetLat {
        c.limit-- // too slow → reduce load
    } else {
        c.limit++ // healthy → increase
    }

    if c.limit < 1 {
        c.limit = 1
    }
}
```

### Usage

```go
start := time.Now()
err := callService()
latency := time.Since(start)

controller.Adjust(latency)
```

## 🌍 Real-World Usage

* Netflix concurrency-limits library
* Envoy adaptive concurrency filter
* Database connection tuning
* RPC systems with SLA targets

### Common in

* Latency-sensitive systems
* High-QPS services

## ⚠️ Common Misunderstanding

> "Lower latency always means increase concurrency"

❌ Not always.

Sometimes:

* Low latency = underutilized
* High latency = saturation

👉 You must define a **target latency**, not chase zero

## 🧠 Elite Secret

Latency signals must be:

* Smoothed (rolling average)
* Not raw per-request values

Otherwise:

* Noise causes instability
* System "thrashes"

## 🔁 Breakthrough Reframe

This is not about limits.

It’s about:

> Keeping the system in a healthy operating zone

---

# ⚔️ AIMD vs Feedback Control

## Pros

### AIMD

* Simple
* Proven (TCP-level reliability)
* Stable under many conditions

### Feedback Control

* More precise
* Reacts earlier (via latency)
* Better for tight SLAs

## Cons

### AIMD

* Reactive (waits for failure)
* Can overshoot
* Less precise

### Feedback Control

* More complex
* Needs tuning (target latency)
* Sensitive to noisy metrics

---

## When to Use AIMD

* Simple systems
* When errors are acceptable signals
* When you want robustness over precision

## When to Use Feedback Control

* Latency-sensitive APIs
* High-throughput services
* Systems with strict SLOs

---

## 🔥 When to Combine Them (Real World)

```
Requests
   ↓
AIMD (coarse control)
   ↓
Latency Controller (fine tuning)
   ↓
Execution
```

👉 AIMD gives stability
👉 Feedback control gives precision

---

## 🧠 Deep Go Tradeoff Insight

This pattern exposes something subtle:

> You’re building a control loop inside your service.

Go forces you to make it:

* Explicit
* Observable
* Testable

No framework is hiding:

* Thresholds
* Signals
* Reactions

That’s power — and responsibility.