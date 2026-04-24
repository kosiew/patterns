---

layout: post
title: "Idempotency Keys vs Exactly-Once Semantics"
date: 2026-04-24 00:00:00 +0000
tags: [distributed-systems, idempotency, exactly-once, system-design, go]
categories: go
---

## Core Distributed Systems Truth

> You will process requests **more than once**.
> Your job is to make it **safe when that happens**.

This is critical for:

* payments 💳
* order creation 🛒
* job processing ⚙️
* message queues 📬

---

## Pattern 1: Idempotency Keys

### 🧠 Idea

Client sends a unique key per operation:

👉 Server ensures the same key → **same result (no duplication)**

### ✅ Go Example

```go
type Store struct {
    mu    sync.Mutex
    cache map[string]string
}

func NewStore() *Store {
    return &Store{cache: make(map[string]string)}
}

func (s *Store) Handle(key string, fn func() (string, error)) (string, error) {
    s.mu.Lock()
    if res, ok := s.cache[key]; ok {
        s.mu.Unlock()
        return res, nil // return previous result
    }
    s.mu.Unlock()

    // perform operation
    res, err := fn()
    if err != nil {
        return "", err
    }

    s.mu.Lock()
    s.cache[key] = res
    s.mu.Unlock()

    return res, nil
}
```

### 🌍 Real-World Usage

* Stripe API (`Idempotency-Key` header)
* payment processing systems
* REST APIs for POST safety
* retry-safe job submission

In Go:

* often implemented in middleware
* backed by Redis / DB (not just memory)

### ⚠️ Common Misunderstanding

**“Idempotency = no duplicates ever”**

❌ Not exactly.

It guarantees:

* same input key → same outcome

But:

* different keys → still duplicate risk

### 🧠 Elite Secret

The hardest part is what you store:

* full response? (safe, expensive)
* resource ID? (lighter, indirect)
* status only? (risky)

👉 Tradeoff = correctness vs storage cost

### 🔁 Breakthrough Reframe

Idempotency is not about preventing retries.

It’s about:

> making retries harmless

---

## Pattern 2: Exactly-Once Semantics (Simulated)

### 🧠 Idea

True **“exactly once”** is nearly impossible in distributed systems.

So we simulate it using:

* idempotency
* deduplication
* transactional guarantees

### ✅ Go Example (DB-backed)

```go
func ProcessPayment(db *sql.DB, key string, amount int) error {
    tx, err := db.Begin()
    if err != nil {
        return err
    }

    // check if already processed
    var exists bool
    err = tx.QueryRow("SELECT EXISTS (SELECT 1 FROM payments WHERE key = ?)", key).Scan(&exists)
    if err != nil {
        tx.Rollback()
        return err
    }

    if exists {
        tx.Rollback()
        return nil // already processed
    }

    // perform insert (unique key constraint helps)
    _, err = tx.Exec("INSERT INTO payments (key, amount) VALUES (?, ?)", key, amount)
    if err != nil {
        tx.Rollback()
        return err
    }

    return tx.Commit()
}
```

### 🌍 Real-World Usage

* Kafka consumers (offset + idempotency)
* payment systems
* distributed job processors
* event-driven architectures

### ⚠️ Common Misunderstanding

**“Exactly-once is guaranteed”**

❌ In reality:

* networks fail
* retries happen
* messages duplicate

👉 you’re building an *illusion*, not a guarantee

### 🧠 Elite Secret

The real power comes from:

👉 **database constraints (UNIQUE keys)**

They act as:

* final line of defense
* race condition protection

Without them:

* idempotency logic can still fail under concurrency

### 🔁 Breakthrough Reframe

Exactly-once is not delivery.

It’s:

> deduplicated side effects

---

## ⚔️ Idempotency Keys vs Exactly-Once

### Pros

#### Idempotency Keys

* simple API-level solution
* works well with retries
* easy to reason about

#### Exactly-Once (Simulated)

* stronger guarantees
* handles concurrency
* integrates with storage layer

### Cons

#### Idempotency Keys

* requires client cooperation
* storage overhead
* tricky expiration policies

#### Exactly-Once

* complex implementation
* requires DB or coordination
* harder to scale

---

## When to Use Idempotency Keys

* HTTP APIs
* client-driven retries
* external integrations

## When to Use Exactly-Once

* internal processing systems
* financial transactions
* message consumers

---

## When to Combine Them 🔥 (Real World)

```
Client Request (Idempotency Key)
        ↓
API Layer (dedupe)
        ↓
DB Transaction (unique constraint)
        ↓
Side Effect (safe)
```

👉 This is how payment systems avoid double charges.

---

## 🧠 Deep Go Tradeoff Insight

Go doesn’t give you:

* distributed transactions
* exactly-once frameworks

Instead you compose:

* `context` (timeouts/retries)
* DB constraints
* explicit logic

👉 This keeps systems:

* transparent
* debuggable
* predictable