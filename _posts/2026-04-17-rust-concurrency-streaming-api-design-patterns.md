---

layout: post
title: "Rust Concurrency, Streaming & API Design Patterns"
date: 2026-04-17 07:20:36 +0000
tags: [rust, concurrency, streaming, system-design, patterns]
categories: rust
---

# 🔒 Mutex vs RwLock

## 🧠 Core Idea

* `Mutex<T>` → one thread at a time
* `RwLock<T>` → many readers or one writer

## When to Use

* **Mutex**: frequent writes, simple state
* **RwLock**: read-heavy workloads

## Insight

> A `Mutex` is often faster under contention due to lower overhead.

---

# ⏱ Fixed Windows vs Session Windows

## 🧠 Core Idea

* Fixed windows → time-based buckets
* Session windows → activity-based grouping

## Tradeoffs

| Dimension   | Fixed     | Session |
| ----------- | --------- | ------- |
| Simplicity  | High      | Medium  |
| Accuracy    | Lower     | Higher  |
| Parallelism | Excellent | Harder  |

## Insight

> Behavior-driven grouping (sessions) better reflects real usage.

---

# 🐄 `Cow<'a, T>` + API Design

## 🧠 Core Idea

* `Cow` → borrowed or owned
* Avoid allocation unless needed

## Pattern

```rust
fn normalize(input: &str) -> Cow<str>
```

## Insight

> `Cow` is a lazy ownership upgrade.

---

# ⏳ Event Time vs Processing Time + Watermarks

## 🧠 Core Idea

* Event time → correct but complex
* Processing time → simple but inaccurate
* Watermarks → control lateness

## Insight

> The real tuning knob is *allowed lateness*.

---

# ⚡ OnceCell / Lazy Initialization

## 🧠 Core Idea

* Initialize once, safely
* Avoid repeated `Option` checks

## When to Use

* configs
* caches
* global state

## Insight

> `OnceCell<T>` encodes “set exactly once.”

---

# 🔁 At-Least-Once vs Exactly-Once

## 🧠 Core Idea

* At-least-once → duplicates possible
* Exactly-once → no duplicate effects

## Insight

> Exactly-once is built from idempotency + durability + atomicity.

---

# 🔄 From vs TryFrom

## 🧠 Core Idea

* `From` → infallible
* `TryFrom` → fallible

## Insight

> Validate at boundaries, not everywhere.

---

# 📦 Dual Writes vs Outbox Pattern

## 🧠 Core Idea

* Dual writes → unsafe
* Outbox → atomic + reliable

## Insight

> Move from cross-system atomicity to single-system atomicity.

---

# 🧱 Newtypes + Validation

## 🧠 Core Idea

* Wrap primitives with meaning
* Validate once via `TryFrom`

## Insight

> Make invalid states unrepresentable.

---

# 🔄 Outbox Polling vs CDC

## 🧠 Core Idea

* Polling → simple, higher latency
* CDC → log-based, real-time

## Insight

> CDC reuses database guarantees instead of reimplementing them.

---

# 👻 Phantom Types + Typestate

## 🧠 Core Idea

* Encode state in types
* Zero runtime cost

## Insight

> Add meaning without adding data.

---

# 📡 Event Sourcing vs CDC

## 🧠 Core Idea

* Event sourcing → events are truth
* CDC → state is truth

## Insight

> “What happened?” vs “What is.”

---

# 📦 Ownership vs Borrowing

## 🧠 Core Idea

* Ownership → control + storage
* Borrowing → flexibility + zero-copy

## Insight

> Borrow for use, own for storage.

---

# 🧠 Final Principles

* Model correctness in types
* Prefer simple primitives first
* Optimize for data movement and locality
* Embrace deterministic state transitions

> Great Rust systems aren’t just safe — they’re *predictable by design*.

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [concurrency](/tags/concurrency/), [streaming](/tags/streaming/), [system-design](/tags/system-design/), [patterns](/tags/patterns/)
<!-- gh-pages-taxonomy-links:end -->
