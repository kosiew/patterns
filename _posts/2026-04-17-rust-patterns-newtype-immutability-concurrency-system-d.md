---

layout: post
title: "Rust Patterns: Newtype, Immutability, Concurrency & System Design"
date: 2026-04-17 07:09:18 +0000
tags: [rust, design-patterns, systems, concurrency, ownership, performance]
categories: rust
---

# 🧩 Newtype Pattern + `TryFrom` / `From`

## 💡 Newtype Pattern (Strong Domain Types)

Wrap a primitive in a struct to give it **meaning** and prevent misuse.

```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
struct Port(u16);
```

### Key Ideas

* `Port` ≠ `u16`
* Attach invariants and behavior
* Prevent mixing unrelated values

### 🧠 Insight

> You’re not wrapping a value — you’re promoting it into a concept.

---

## 🔄 `TryFrom` / `From` (Validated Conversion)

Convert raw data into validated domain types.

```rust
use std::convert::TryFrom;

impl TryFrom<u16> for Port {
    type Error = &'static str;

    fn try_from(value: u16) -> Result<Self, Self::Error> {
        if value == 0 {
            Err("invalid port")
        } else {
            Ok(Port(value))
        }
    }
}
```

### 🧠 Insight

> `TryFrom` defines your system’s trust boundary.

---

# 🧊 Immutability + Functional Update

## 💡 Immutability by Default

Rust values are immutable unless marked `mut`.

### Why it matters

* Safer concurrent code
* No hidden mutations
* Easier reasoning

---

## 🔄 Functional Update

Create new values instead of mutating.

```rust
impl Config {
    fn with_timeout(self, timeout_ms: u64) -> Self {
        Self { timeout_ms, ..self }
    }
}
```

### 🧠 Insight

> Values don’t change — they flow.

---

# 🔁 Concurrency Optimization Patterns

## ⚡ Read Batching

* Combine multiple reads into one operation
* Improves throughput and locality

## 🔗 Request Coalescing (Single-Flight)

* Deduplicate identical in-flight requests
* Prevent thundering herd

### 🔥 Combined Pattern

```
requests → coalesce → batch → execute
```

---

# 📦 Data Access Strategies

## 📖 Read-Ahead

* Assumes sequential access
* Low risk, predictable

## 🔮 Speculative Prefetch

* Predicts future access
* High reward, higher risk

### 🧠 Insight

> Read-ahead handles the obvious future. Speculation handles the likely future.

---

# ⚡ Lazy Initialization (`OnceLock` / `LazyLock`) + `Arc`

## 💤 Lazy Initialization

```rust
static CONFIG: OnceLock<String> = OnceLock::new();
```

* Thread-safe
* Runs once
* No `unsafe`

---

## 🔗 Shared State with `Arc`

```rust
Arc<AppState>
```

* Cheap cloning
* Multi-thread safe

### 🧠 Insight

> Global state is fine — uncontrolled mutation isn’t.

---

# 📚 Borrowing vs Interior Mutability

## Borrowing (`&self` vs `&mut self`)

* Compile-time guarantees
* Clear ownership rules

## Interior Mutability (`RefCell`, `Cell`)

* Runtime borrow checking
* More flexible APIs

### 🧠 Insight

> Interior mutability is an explicit tradeoff, not a loophole.

---

# 📊 Aggregation Strategies

## ⚡ Hash Aggregation

* O(N) average
* Random access
* Fast but cache-unfriendly

## 📈 Sort-Based Aggregation

* O(N log N)
* Sequential access
* Cache-friendly

### 🧠 Insight

> Sequential memory access often beats theoretical complexity.

---

# 💾 External (Disk-Based) Processing

## 🧩 External Hash Aggregation

* Partition → aggregate
* Sensitive to skew

## 🔀 External Sort + Merge

* Sort → merge → aggregate
* Predictable and sequential

---

# 🔗 Join Strategies at Scale

## 🧩 Grace Hash Join

* Partition both sides
* Join per partition

## 🔀 External Merge Join

* Sort both sides
* Merge join

### 🧠 Insight

> Hardware behavior often dominates Big-O.

---

# 🧾 Borrowed vs Owned Data

## Borrowed (`&str`, `&[T]`)

* Zero allocation
* Flexible APIs

## Owned (`String`, `Vec<T>`)

* Required for storage
* Clear ownership

### 🧠 Rule

> Borrow for use, own for storage.

---

# 🔁 `IntoIterator` + `AsRef`

## `IntoIterator`

Accept any iterable input.

## `AsRef`

Accept flexible reference types.

```rust
fn print_lines<I, S>(lines: I)
where
    I: IntoIterator<Item = S>,
    S: AsRef<str>,
```

### 🧠 Insight

> Accept streams, not containers.

---

# 🎯 Final Principles

* Encode correctness in types
* Prefer borrowing over owning (until you must own)
* Favor sequential memory access
* Make concurrency explicit
* Control initialization and ownership

> Rust isn’t about writing code.
> It’s about designing systems that cannot break.
