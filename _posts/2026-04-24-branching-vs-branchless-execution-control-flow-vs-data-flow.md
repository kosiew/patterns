---

layout: post
title: "Branching vs Branchless Execution (Control Flow vs Data Flow)"
date: 2026-04-24 00:00:00 +0000
tags: [performance, cpu, simd, rust, vectorization]
categories: database
---

## Branching vs Branchless Execution (Control Flow vs Data Flow)

This builds *directly* on indirection and bitmaps:

> "Is it faster to *decide with branches*… or *compute without branching*?"

* **Branching execution** → `if` statements, control flow
* **Branchless execution** → arithmetic / masks, data flow

This is a *massive deal* in vectorized engines.

---

## 🧠 Core Idea

Branching uses `if` statements to decide what to do per row. It’s intuitive and efficient when predictions are accurate—but modern CPUs suffer when branches are unpredictable.

Branchless execution removes `if` entirely. Instead, it computes results using masks and arithmetic. This keeps execution linear and predictable—perfect for SIMD and tight loops.

---

## 🦀 Technique A — Branching (Control Flow)

```rust
fn branching_filter_sum(data: &[i32]) -> i32 {
    let mut sum = 0;

    for &x in data {
        if x > 10 {
            sum += x;
        }
    }

    sum
}

fn main() {
    let data = vec![5, 20, 3, 15, 8];
    println!("{}", branching_filter_sum(&data));
}
```

### 💡 Key Property

* Simple and readable
* Performance depends on **branch prediction**

---

## 🦀 Technique B — Branchless (Mask-Based)

```rust
fn branchless_filter_sum(data: &[i32]) -> i32 {
    let mut sum = 0;

    for &x in data {
        // (x > 10) as i32 → 1 or 0
        let mask = (x > 10) as i32;
        sum += x * mask;
    }

    sum
}

fn main() {
    let data = vec![5, 20, 3, 15, 8];
    println!("{}", branchless_filter_sum(&data));
}
```

### 💡 Key Property

* No branches
* Predictable, pipeline-friendly
* Enables SIMD/vectorization

---

## ⚖️ Tradeoffs

| Dimension          | Branching (A)     | Branchless (B) |
| ------------------ | ----------------- | -------------- |
| Readability        | ✅ very clear      | ❌ less obvious |
| CPU predictability | ❌ depends         | ✅ stable       |
| SIMD potential     | ❌ limited         | ✅ excellent    |
| Best case          | predictable data  | random data    |
| Worst case         | branch mispredict | extra ops      |

---

## 🎯 When to Use

### Pick Branching when:

* Condition is **highly predictable**
* Code clarity matters
* Work per item is large anyway

👉 Example: filtering already-sorted or skewed data

### Pick Branchless when:

* Data is **random/unpredictable**
* You are in a **tight loop**
* You want **vectorization**

👉 Example: analytical scans over millions of rows

---

## 🤝 Combination Pattern (What engines really do 🔥)

```
[Columnar data]
        ↓
[Bitmap mask]        ← branchless generation
        ↓
[Vectorized ops]     ← branchless compute
        ↓
[Selection vector]   ← optional indirection
```

### Why this wins

* Avoids unpredictable branches
* Keeps CPU pipelines full
* Composes perfectly with SIMD

---

## 🌍 Real-World Systems

### DuckDB / ClickHouse

* Heavy use of **branchless execution**
* Filters become **bitmasks**, not `if` statements

### Apache Arrow

* Uses **bitmaps + SIMD-friendly operations**

### PostgreSQL (vectorized paths)

* Moving toward **branchless filtering**

---

## ⚠️ Common Misunderstanding

> "Branchless is always faster"

Not if:

* branches are perfectly predictable
* or extra math dominates cost

Sometimes a simple `if` wins.

---

## 🧪 Elite Secret

Modern CPUs:

> **Speculate and pipeline aggressively—but hate unpredictability**

A single bad branch can cost:

* ~10–20 cycles (pipeline flush)

Branchless code trades:

* a few extra ops
* for:

  * zero pipeline stalls

---

## 🔁 Breakthrough Reframe (Rust Lens)

Rust makes this explicit via types:

```rust
let mask: i32 = (x > 10) as i32;
```

You’re turning:

> control flow → **data**

### That’s the key shift:

* Branching = *what path do we take?*
* Branchless = *compute everything, select via data*