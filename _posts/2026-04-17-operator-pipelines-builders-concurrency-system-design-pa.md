---

layout: post
title: "Operator Pipelines, Builders, Concurrency & System Design Patterns in Rust"
date: 2026-04-17 07:15:59 +0000
tags: [rust, system-design, concurrency, patterns, performance]
categories: rust
---

## Overview

This compendium explores key **systems design and Rust programming patterns**, focusing on performance, correctness, and scalability tradeoffs.

Each section compares two techniques solving similar problems with different philosophies.

---

# Operator Pipeline vs Operator Fusion

## 🧠 Core Idea

* **Operator Pipeline** → modular stages with buffers
* **Operator Fusion** → single tight loop

Same computation. Different execution model.

## Tradeoffs

| Dimension      | Pipeline | Fusion    |
| -------------- | -------- | --------- |
| Modularity     | High     | Low       |
| Memory         | Higher   | Minimal   |
| Cache locality | Moderate | Excellent |
| Speed          | Lower    | Higher    |

## Insight

The real gain from fusion is **reducing memory traffic**, not function calls.

---

# Builder vs Typestate Builder

## 🧠 Core Idea

* **Builder** → ergonomic construction
* **Typestate Builder** → compile-time correctness

## Tradeoffs

| Dimension  | Builder | Typestate    |
| ---------- | ------- | ------------ |
| Simplicity | High    | Lower        |
| Safety     | Runtime | Compile-time |

## Insight

Builders act as a **configuration DSL**, while typestate encodes **valid construction paths**.

---

# Unbounded Queue vs Backpressure

## 🧠 Core Idea

* **Unbounded Queue** → never blocks, risk of explosion
* **Backpressure** → bounded, forces system stability

## Insight

Failures usually come from **latency explosion**, not memory exhaustion.

---

# Option vs Result

## 🧠 Core Idea

* `Option<T>` → absence
* `Result<T, E>` → failure with reason

## Insight

Use `Option` for **missing data**, `Result` for **recoverable errors**.

---

# Fixed vs Adaptive Batching

## 🧠 Core Idea

* **Fixed** → predictable
* **Adaptive** → responsive

## Insight

Batch size is usually tuned to **CPU cache**, not dataset size.

---

# impl Trait vs Generics

## 🧠 Core Idea

* `impl Trait` → ergonomic
* Generics → expressive relationships

## Insight

Use `impl Trait` for readability, generics for **type relationships**.

---

# Key Salting vs Key Splitting

## 🧠 Core Idea

* **Salting** → random distribution
* **Splitting** → deterministic partitioning

## Insight

The real challenge is **detecting hot keys early**.

---

# impl Iterator vs Box<dyn Iterator>

## 🧠 Core Idea

* `impl Iterator` → zero-cost abstraction
* `Box<dyn Iterator>` → runtime flexibility

## Insight

Use `impl Iterator` by default; box only when **types diverge at runtime**.

---

# Count-Min Sketch vs Space-Saving

## 🧠 Core Idea

* **CMS** → approximate counts
* **Space-Saving** → exact Top-K

## Insight

CMS reduces memory; Space-Saving tracks **actual heavy hitters**.

---

# Rc + RefCell vs Arc + Mutex

## 🧠 Core Idea

* `Rc<RefCell<T>>` → single-thread shared mutability
* `Arc<Mutex<T>>` → multi-thread safe sharing

## Insight

Same concept, different safety model: **compile-time vs thread-safe runtime control**.

---

# HyperLogLog vs Bloom Filter

## 🧠 Core Idea

* **HLL** → count distinct
* **Bloom** → membership test

## Insight

HLL saves memory. Bloom filters reduce **unnecessary work**.

---

# Shared State vs Message Passing

## 🧠 Core Idea

* **Shared State** → locks
* **Message Passing** → ownership transfer

## Insight

Great systems design revolves around **clear ownership boundaries**.

---

# RAII vs Manual Cleanup

## 🧠 Core Idea

* **RAII** → automatic cleanup
* **Manual** → explicit control

## Insight

Rust is about **resource safety**, not just memory safety.

---

# AsRef vs Into

## 🧠 Core Idea

* `AsRef` → borrow
* `Into` → own

## Insight

Borrow for operations, own for storage.

---

# Enum vs Trait Object Polymorphism

## 🧠 Core Idea

* **Enum** → closed, optimized
* **Trait Object** → open, extensible

## Insight

Enums give performance. Traits give extensibility.

---

# Central Queue vs Work-Stealing

## 🧠 Core Idea

* **Central Queue** → simple but contended
* **Work-Stealing** → scalable but complex

## Insight

Performance depends heavily on **task granularity**.

---

# Data Parallelism vs Pipeline Parallelism

## 🧠 Core Idea

* **Data Parallelism** → split data
* **Pipeline Parallelism** → split stages

## Insight

Modern systems combine both.

---

# State Machines + Futures

## 🧠 Core Idea

Rust async = **compiled state machine**.

## Insight

Every `.await` is a **state transition**.

---

## Final Thought

All these patterns orbit a few deep principles:

* **Control memory movement**
* **Make ownership explicit**
* **Prefer compile-time guarantees**
* **Optimize for cache and data locality**

He gives a small nod 😄

> "Master these, and you're not just writing Rust… you're designing systems."

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [system-design](/tags/system-design/), [concurrency](/tags/concurrency/), [patterns](/tags/patterns/), [performance](/tags/performance/)
<!-- gh-pages-taxonomy-links:end -->
