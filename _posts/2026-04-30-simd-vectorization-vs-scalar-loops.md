---

layout: post
title: "SIMD Vectorization vs Scalar Loops"
date: 2026-04-30 00:00:00 +0000
tags: [performance, simd, rust, systems, optimization]
categories: database
---

# SIMD Vectorization vs Scalar Loops (Hardware Parallelism vs Sequential Execution)

Now we zoom *all the way down* to the hardware level:

> “How do we make one CPU instruction process many values at once?”

* **Scalar loops** → one value per iteration
* **SIMD (Single Instruction, Multiple Data)** → many values per instruction

This is where everything you’ve learned *(vectorization, branchless, bitmaps)* pays off massively.

---

## 🧠 Core Idea

A **scalar loop** processes one element at a time. Even if it’s optimized, it fundamentally does *sequential work*.

**SIMD vectorization** uses CPU instructions that operate on multiple values packed into registers (e.g., 4, 8, 16 integers at once). The trick: your data and code must be structured to allow it *(contiguous memory + branchless logic)*.

---

## 🦀 Technique A — Scalar Loop (Baseline)

```rust
fn scalar_sum(data: &[i32]) -> i32 {
    let mut sum = 0;

    for &x in data {
        if x > 10 {
            sum += x;
        }
    }

    sum
}

fn main() {
    let data: Vec<i32> = (0..1_000_000).collect();
    println!("{}", scalar_sum(&data));
}
```

### 💡 Key Property

* Simple, flexible
* Hard to fully utilize CPU execution units

---

## 🦀 Technique B — SIMD-Style (Manual Chunking + Branchless)

Rust stable doesn’t expose full portable SIMD everywhere yet, but you can structure code to enable it:

```rust
fn simd_friendly_sum(data: &[i32]) -> i32 {
    let mut sum = 0;

    // Process in chunks (helps auto-vectorization)
    for chunk in data.chunks(8) {
        for &x in chunk {
            let mask = (x > 10) as i32;
            sum += x * mask;
        }
    }

    sum
}

fn main() {
    let data: Vec<i32> = (0..1_000_000).collect();
    println!("{}", simd_friendly_sum(&data));
}
```

### 💡 Key Property

* Chunking + branchless → compiler can auto-vectorize
* Maps to SIMD instructions under the hood

---

## ⚖️ Tradeoffs

| Dimension       | Scalar (A)  | SIMD-Friendly (B)          |
| --------------- | ----------- | -------------------------- |
| Simplicity      | ✅ very high | ⚠️ slightly more structure |
| Raw throughput  | ❌ limited   | ✅ much higher              |
| CPU utilization | ❌ underused | ✅ full usage               |
| Portability     | ✅ universal | ⚠️ depends on CPU          |
| Best case       | small data  | large scans                |

---

## 🎯 When to Use

### Pick Scalar when:

* Data size is small
* Logic is complex / branch-heavy
* You don’t control layout

👉 Example: business logic, OLTP paths

### Pick SIMD-friendly when:

* You scan large arrays
* Logic is simple + branchless
* Data is contiguous

👉 Example: analytics, filtering, aggregation

---

## 🤝 Combination Pattern (Modern engine inner loop 🔥)

```
[Columnar data]
        ↓
[Bitmap filter]      ← branchless
        ↓
[SIMD vectorized loop] ← parallel compute
        ↓
[Aggregation]
```

### Why this wins

* Columnar → contiguous memory
* Bitmap → no branches
* SIMD → parallel execution

👉 This is the **core tight loop of OLAP systems**

---

## 🌍 Real-World Systems

### DuckDB

* Heavy use of SIMD vectorization
* Carefully structured execution loops

### ClickHouse

* Uses SIMD for filtering, aggregation, compression

### Apache Arrow

* Designed for SIMD-friendly memory layout

### Snowflake / modern engines

* Similar vectorized execution principles

---

## ⚠️ Common Misunderstanding (Applies to SIMD)

> “The compiler will always vectorize automatically”

Nope.

It often fails if:

* branches exist
* memory is not contiguous
* aliasing is unclear

You must **structure code for it**.

---

## 🧪 Elite Secret

The real trick isn’t writing SIMD intrinsics.

> It’s writing code the compiler can *safely vectorize*

That means:

* no aliasing
* no branches
* predictable memory access

Rust helps a lot here.

---

## 🔁 Breakthrough Reframe (Rust Lens)

Rust gives the compiler strong guarantees:

* `&[T]` → contiguous memory
* no aliasing (vs C/C++)
* clear ownership

👉 This makes **auto-vectorization far more reliable**

---

## ✨ Subtle but Powerful

```rust
fn process(data: &[i32])
```

This signature already tells the compiler:

* safe to optimize
* safe to reorder
* safe to vectorize