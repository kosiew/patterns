---

layout: post
title: "Request Coalescing & Singleflight in Go"
date: 2026-04-22 00:00:00 +0000
tags: [go, concurrency, distributed-systems, system-design, performance]
categories: go
---

# 🧵 Pattern 1: Request Coalescing

## 🧠 Idea

If multiple identical requests arrive:

* 👉 **collapse them into one execution**
* 👉 **share the result with all callers**

## ✅ Go Example (manual)

```go
type call struct {
	done chan struct{}
	res  string
	err  error
}

type Coalescer struct {
	mu    sync.Mutex
	calls map[string]*call
}

func NewCoalescer() *Coalescer {
	return &Coalescer{
		calls: make(map[string]*call),
	}
}

func (c *Coalescer) Do(key string, fn func() (string, error)) (string, error) {
	c.mu.Lock()
	if existing, ok := c.calls[key]; ok {
		c.mu.Unlock()
		<-existing.done
		return existing.res, existing.err
	}

	call := &call{done: make(chan struct{})}
	c.calls[key] = call
	c.mu.Unlock()

	// execute once
	call.res, call.err = fn()

	close(call.done)

	c.mu.Lock()
	delete(c.calls, key)
	c.mu.Unlock()

	return call.res, call.err
}
```

## 🌍 Real-World Usage

* cache stampede prevention
* API fan-in layers
* database query deduplication
* config/feature flag loading

Common in:

* high-QPS services
* read-heavy systems
* microservices with shared dependencies

## ⚠️ Common Misunderstanding

“This is just caching”

**❌ No.**

* caching stores results
* coalescing prevents **duplicate in-flight work**

👉 works even when cache is cold

## 🧠 Elite Secret

Coalescing reduces **tail latency under burst**.

Why?

* prevents thundering herd
* avoids resource contention spikes

👉 especially important during cache misses

## 🔄 Breakthrough Reframe

Coalescing is not optimization.

It’s:

> **coordination between concurrent callers**

---

# 🧰 Pattern 2: Singleflight (Go’s built-in version)

## 🧠 Idea

Same as coalescing — but already implemented:

👉 `golang.org/x/sync/singleflight`

## ✅ Go Example

```go
import "golang.org/x/sync/singleflight"

var g singleflight.Group

func fetch(key string) (string, error) {
	v, err, _ := g.Do(key, func() (interface{}, error) {
		return expensiveCall(key)
	})
	if err != nil {
		return "", err
	}
	return v.(string), nil
}
```

## 🌍 Real-World Usage

* used inside Go ecosystem libraries
* Kubernetes controllers
* HTTP clients with caching layers
* internal service SDKs

Why Go devs love it:

* tiny API
* no framework
* composable with existing code

## ⚠️ Common Misunderstanding

“Singleflight replaces caching”

**❌ No.**

It works best **with caching**:

```text
Request → Cache → (miss) → Singleflight → DB
```

## 🧠 Elite Secret

Singleflight does **not persist results**.

👉 Once the call finishes:

* next request runs again

So:

* use it for *in-flight deduplication only*
* combine with cache for full benefit

## 🔄 Breakthrough Reframe

Singleflight is not a data structure.

It’s:

> **a concurrency control primitive**

---

# ⚔️ Coalescing vs Singleflight

## Pros

### Coalescing (custom)

* fully customizable
* can add TTL, caching, metrics
* more control over lifecycle

### Singleflight

* battle-tested
* minimal code
* easy to drop in

## Cons

### Coalescing

* easy to get wrong (race conditions)
* more code
* harder to maintain

### Singleflight

* limited flexibility
* no built-in caching
* less visibility into internals

## When to Use Coalescing

* need custom behavior
* want tight control (timeouts, eviction)
* building infrastructure layer

## When to Use Singleflight

* most applications
* quick win for duplicate suppression
* when simplicity matters

---

# 🔥 When to Combine Them (Real World)

```text
Incoming Requests
    ↓
Cache (fast path)
    ↓
Singleflight (dedupe in-flight)
    ↓
DB / API
```

👉 This is **the standard pattern** for:

* cache stampede prevention
* high-scale read systems

## 🧠 Deep Go Tradeoff Insight

This pattern highlights something subtle:

> Go solves coordination with *shared state + channels*, not frameworks.

You explicitly manage:

* who does the work
* who waits
* when to release

No magic. No hidden queues.