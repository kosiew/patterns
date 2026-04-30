---

layout: post
title: "Saga Pattern and Compensating Transactions"
date: 2026-04-30 00:00:00 +0000
tags: [distributed-systems, microservices, saga-pattern, compensating-transactions, go]
categories: go
---

# Saga Pattern and Compensating Transactions

## Pattern 1: Saga Pattern

### Idea

Break a big transaction into **small steps across services**.

Each step:

* succeeds independently
* or triggers a rollback action, also called compensation

---

### Go Example: Orchestrated Saga

```go
type Saga struct {
    steps []func() error
    undo  []func()
}

func (s *Saga) Add(step func() error, compensate func()) {
    s.steps = append(s.steps, step)
    s.undo = append([]func(){compensate}, s.undo...) // reverse order
}

func (s *Saga) Execute() error {
    for i, step := range s.steps {
        if err := step(); err != nil {
            // rollback previous steps
            for _, undo := range s.undo[:len(s.steps)-i] {
                undo()
            }
            return err
        }
    }

    return nil
}
```

---

### Real-World Usage

* microservices orchestration: order → payment → shipping
* Kubernetes operators: reconciliation loops
* workflow engines: Temporal, Cadence
* distributed business logic

Go favors:

* explicit orchestration, not hidden workflow engines
* simple functions instead of DSLs

---

### Common Misunderstanding

> “Saga = distributed transaction”

No.

* transactions = atomic: all or nothing
* sagas = **eventual consistency**

The system may be temporarily inconsistent.

---

### Elite Secret

Saga correctness depends on:

> **step ordering + idempotency**

If compensation runs twice:

* does it break things?

If a step partially succeeds:

* can you safely retry?

---

### Breakthrough Reframe

Saga is not about success.

It is about:

> **recovering correctly from partial failure**

---

## Pattern 2: Compensating Transactions

### Idea

For every step:

> define an **undo action**

Example:

* charge payment → refund payment
* reserve inventory → release inventory

---

### Go Example

```go
func charge() error {
    fmt.Println("charged")
    return nil
}

func refund() {
    fmt.Println("refunded")
}

func reserve() error {
    fmt.Println("reserved")
    return errors.New("inventory failure") // simulate failure
}

func release() {
    fmt.Println("released")
}
```

Used in saga:

```go
s := &Saga{}
s.Add(charge, refund)
s.Add(reserve, release)

err := s.Execute()
// Output:
// charged
// reserved
// released
// refunded
```

---

### Real-World Usage

* payment rollback
* inventory release
* booking cancellations
* job compensation in queues

Often backed by:

* message queues
* retry mechanisms
* idempotent handlers

---

### Common Misunderstanding

> “Compensation = exact rollback”

Not always.

Example:

* refund may fail
* inventory may already be taken

Compensation is **best-effort**, not perfect.

---

### Elite Secret

Compensation must be:

* **idempotent**
* **retryable**
* **safe under partial execution**

Otherwise, failure recovery breaks.

---

### Breakthrough Reframe

Compensation is not undo.

It is:

> **a new forward action that restores system correctness**

---

## Saga vs Compensating Transactions

### Pros

#### Saga

* handles multi-step workflows
* scalable across services
* avoids distributed locks

#### Compensation

* flexible recovery mechanism
* works with existing systems
* simple mental model

---

### Cons

#### Saga

* complex coordination
* hard to debug
* eventual consistency issues

#### Compensation

* not always perfect rollback
* requires careful design
* can fail itself

---

## When to Use Saga

* multi-service workflows
* long-running processes
* distributed systems

---

## When to Use Compensation

* any step with side effects
* external systems: payments, APIs
* failure recovery logic

---

## When to Combine Them: Real World

```text
Order Created
    ↓
Charge Payment
    ↓
Reserve Inventory
    ↓
Ship Item
    ↓
Failure
    ↓
Release Inventory
    ↓
Refund Payment
```

This is how real e-commerce systems work.

---

## Deep Go Tradeoff Insight

Go does not give you:

* distributed transactions
* workflow engines by default

Instead, you build:

* explicit steps
* explicit compensation
* explicit retries

Result:

* more code
* but **clear control + debuggability**