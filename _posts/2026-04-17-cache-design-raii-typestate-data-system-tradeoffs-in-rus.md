---

layout: post
title: "Cache Design, RAII, Typestate & Data System Tradeoffs in Rust"
date: 2026-04-17 07:23:35 +0000
tags: [rust, caching, system-design, streaming, performance]
categories: rust
---

## Overview

This post explores several **core design tradeoffs in systems and Rust**, covering:

* Cache eviction strategies
* Resource management patterns
* Data system computation models
* Iterator and zero-cost abstractions

Each section contrasts two approaches with different philosophies and performance characteristics.

---

# 🧠 LRU vs TinyLFU (Cache Admission & Eviction)

## Core Idea

* **LRU (Least Recently Used)** → evicts based on recency
* **TinyLFU-style admission** → filters entries based on frequency before insertion

**LRU** assumes recent access predicts future access.
**TinyLFU** asks whether an item is *worth caching at all*.

---

## ⚖️ Tradeoffs

| Dimension  | LRU              | TinyLFU         |
| ---------- | ---------------- | --------------- |
| Latency    | Fast             | Slight overhead |
| Accuracy   | Weak under scans | Strong          |
| Throughput | Lower            | Higher          |
| Memory     | Moderate         | Extra metadata  |

---

## 🎯 When to Use

### Use LRU when:

* strong temporal locality
* simple systems
* small caches

### Use TinyLFU-style when:

* scan-heavy workloads
* large-scale systems
* hit rate matters

---

## 🤝 Combination Pattern

**TinyLFU + LRU (W-TinyLFU)**

* TinyLFU decides admission
* LRU manages recency

> Separation of concerns: *who enters vs who stays*

---

# ♻️ RAII (Resource Acquisition Is Initialization)

## 💡 Idea

Tie resource lifetime to scope using ownership and `Drop`.

```rust
use std::fs::File;

fn read_file() {
    let file = File::open("data.txt").unwrap();
} // automatically closed
```

---

## 🧠 Insight

> Cleanup is guaranteed — even during early returns or panics.

---

# 🔐 Typestate (Lifecycle Encoding)

## 💡 Idea

Encode valid states and transitions in types.

```rust
struct Disconnected;
struct Connected;

struct Socket<State> {
    _state: std::marker::PhantomData<State>,
}
```

---

## 🧠 Insight

> Illegal states become unrepresentable.

---

# ⚖️ RAII vs Typestate

| Dimension  | RAII           | Typestate              |
| ---------- | -------------- | ---------------------- |
| Purpose    | Cleanup        | Correct usage ordering |
| Guarantees | Runtime (Drop) | Compile-time           |
| Complexity | Low            | Higher                 |

---

# 🧠 Merge-on-Read vs Pre-Materialization

## Core Idea

* **Merge-on-read** → compute at query time
* **Materialized views** → compute ahead of time

---

## ⚖️ Tradeoffs

| Dimension   | Merge-on-Read | Materialized |
| ----------- | ------------- | ------------ |
| Read speed  | Slow          | Fast         |
| Write cost  | Cheap         | Higher       |
| Flexibility | High          | Low          |

---

## 🧠 Insight

> Materialization is effectively a cache over raw data.

---

# 🔗 Iterator Adaptors

## 💡 Idea

Compose transformations lazily.

```rust
let result: Vec<i32> = (1..10)
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .collect();
```

---

## 🧠 Insight

> Iterators are computation pipelines, not collections.

---

# ⚡ Zero-Cost Abstractions

## 💡 Idea

High-level abstractions with no runtime overhead.

```rust
fn sum_even(nums: &[i32]) -> i32 {
    nums.iter()
        .filter(|x| **x % 2 == 0)
        .map(|x| x * x)
        .sum()
}
```

---

## 🧠 Insight

> Abstractions disappear at compile time.

---

# 🧠 Final Principles

* Separate admission from storage (caching)
* Model correctness with types
* Prefer incremental and precomputed systems when needed
* Optimize for memory movement and locality

> Great systems are not just fast — they are predictable and composable.

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [caching](/tags/caching/), [system-design](/tags/system-design/), [streaming](/tags/streaming/), [performance](/tags/performance/)
<!-- gh-pages-taxonomy-links:end -->
