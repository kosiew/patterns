---

layout: post
title: "Event Replay, Iterators, Execution & System Design Patterns"
date: 2026-04-17 07:21:57 +0000
tags: [rust, system-design, streaming, data-processing, performance]
categories: rust
---

# ♻️ Full Replay vs Snapshotting

## 🧠 Core Idea

* **Full Replay** → rebuild state from all events
* **Snapshotting** → checkpoint state + replay recent events

## Tradeoffs

| Dimension      | Full Replay | Snapshotting |
| -------------- | ----------- | ------------ |
| Simplicity     | High        | Medium       |
| Recovery speed | Slow        | Fast         |
| Storage        | Minimal     | Extra        |

## Insight

> Snapshots are a cache — events remain the source of truth.

---

# 📦 Ownership Receivers + Copy vs Move

## 🧠 Core Idea

* `&T` → read-only
* `&mut T` → mutate
* `T` → take ownership

## Insight

> Choosing the receiver defines your API contract.

---

# 🔁 Iterator-Based APIs

## 🧠 Core Idea

* Accept `IntoIterator` → flexible input
* Return `impl Iterator` → lazy output

## Pattern

```rust
fn process<I: IntoIterator<Item = i32>>(input: I) -> impl Iterator<Item = i32>
```

## Insight

> Iterators give zero-cost abstraction and composability.

---

# 🔄 Versioned Events vs Upcasting

## 🧠 Core Idea

* Versioned → handle multiple versions
* Upcasting → normalize to one version

## Insight

> Mature systems store versions but process a unified model.

---

# ⚡ Materialized Views vs On-Demand Queries

## 🧠 Core Idea

* Materialized → precompute
* On-demand → compute at query time

## Insight

> Incremental updates turn views into state machines.

---

# ❗ The `?` Operator + Error Conversion

## 🧠 Core Idea

* `?` → propagate errors
* `From` → convert errors automatically

## Insight

> Error handling becomes control flow.

---

# 🔗 Deref / DerefMut Ergonomics

## 🧠 Core Idea

* `Deref` → access inner methods
* `DerefMut` → mutate inner value

## Insight

> Wrapper types can behave like their inner type — but use carefully.

---

# ⚡ Incremental vs Full Recompute

## 🧠 Core Idea

* Full recompute → O(N)
* Incremental → O(Δ)

## Insight

> Model updates as deltas (+Δ / -Δ).

---

# 🔒 Sealed Traits

## 🧠 Core Idea

* Public trait + private implementation control

## Insight

> Sealing protects invariants and future evolution.

---

# 🏦 OLTP vs OLAP

## 🧠 Core Idea

* OLTP → transactions
* OLAP → analytics

## Insight

> Different workloads require different storage and execution models.

---

# 🏗 Lambda vs Kappa Architecture

## 🧠 Core Idea

* Lambda → batch + streaming
* Kappa → streaming only

## Insight

> Kappa relies on replayable event logs.

---

# 🧩 Blanket Impl + Extension Traits

## 🧠 Core Idea

* Blanket impl → apply behavior broadly
* Extension traits → add methods to existing types

## Insight

> Traits enable composition over inheritance.

---

# ⚙️ Row vs Vectorized Execution

## 🧠 Core Idea

* Row → one record at a time
* Vectorized → batch processing

## Insight

> Performance comes from cache locality and SIMD.

---

# 🚫 Orphan Rule + Newtype

## 🧠 Core Idea

* Cannot implement external trait for external type
* Use newtype wrapper

## Insight

> You must own either the trait or the type.

---

# 🧠 Final Principles

* Prefer incremental over full recomputation
* Model systems as state + events
* Use types to encode correctness
* Optimize for data movement, not just CPU

> Great systems are not just fast — they are predictable, composable, and resilient.

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [system-design](/tags/system-design/), [streaming](/tags/streaming/), [data-processing](/tags/data-processing/), [performance](/tags/performance/)
<!-- gh-pages-taxonomy-links:end -->
