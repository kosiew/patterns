---

layout: post
title: "Selection Vectors vs Bitmaps (Row Tracking in Vectorized Pipelines)"
date: 2026-04-22 00:00:00 +0000
tags: [databases, query-engines, vectorization, performance, systems]
categories: rust
---

## Selection Vectors vs Bitmaps (Row Tracking in Vectorized Pipelines)

We continue directly from yesterday’s idea:

> “How do we represent *which rows survive* filtering… efficiently?”

* **Selection Vector (`Vec<usize>`)** → explicit row indices
* **Bitmap (`Vec<bool>` / bitset)** → compact inclusion mask

This decision shows up *everywhere* in query engines.

---

## 🧠 Core Idea

A **selection vector** stores the exact indices of rows that passed a filter. It’s compact when few rows survive and very fast to iterate.

A **bitmap** stores one bit per row (keep/drop). It’s fixed-size and enables fast logical operations (AND/OR), making it ideal when combining many filters.

---

## 🦀 Technique A — Selection Vector (Sparse Representation)

```rust
fn selection_vector(data: &[i32]) -> Vec<usize> {
    data.iter()
        .enumerate()
        .filter(|(_, &x)| x > 10)
        .map(|(i, _)| i)
        .collect()
}

fn apply_selection(data: &[i32], sel: &[usize]) -> Vec<i32> {
    sel.iter().map(|&i| data[i]).collect()
}

fn main() {
    let data = vec![5, 20, 3, 15, 8];

    let sel = selection_vector(&data);
    let result = apply_selection(&data, &sel);

    println!("{:?}", result);
}
```

### 💡 Key Property

* Only stores **matching rows**
* Iteration = tight loop over survivors

---

## 🦀 Technique B — Bitmap (Dense Representation)

```rust
fn bitmap_filter(data: &[i32]) -> Vec<bool> {
    data.iter().map(|&x| x > 10).collect()
}

fn apply_bitmap(data: &[i32], mask: &[bool]) -> Vec<i32> {
    data.iter()
        .zip(mask)
        .filter(|(_, &keep)| keep)
        .map(|(&x, _)| x)
        .collect()
}

fn main() {
    let data = vec![5, 20, 3, 15, 8];

    let mask = bitmap_filter(&data);
    let result = apply_bitmap(&data, &mask);

    println!("{:?}", result);
}
```

### 💡 Key Property

* One bit per row (can be packed tighter than `bool`)
* Enables **vectorized logical ops**

---

## ⚖️ Tradeoffs

| Dimension         | Selection Vector (A) | Bitmap (B)           |
| ----------------- | -------------------- | -------------------- |
| Memory            | ✅ small (sparse)     | ❌ fixed size         |
| Iteration cost    | ✅ only survivors     | ❌ scans all rows     |
| Combining filters | ❌ harder             | ✅ very fast (AND/OR) |
| Cache behavior    | great if selective   | predictable          |
| Best case         | few matches          | many filters         |

---

## 🎯 When to Use

### Pick Selection Vector when:

* Filter is **highly selective**
* You want to skip as much work as possible
* Downstream ops are expensive

👉 Example: `WHERE user_id = 42`

### Pick Bitmap when:

* You combine **many predicates**
* Selectivity is moderate/high
* You want SIMD-friendly operations

👉 Example: `(age > 18 AND country = 'US' AND active = true)`

---

## 🤝 Combination Pattern (What real engines do 🔥)

```text
[Initial scan]
        ↓
[Bitmap filters]  ← combine predicates cheaply
        ↓
[Convert to selection vector]
        ↓
[Late materialization / heavy work]
```

---

## Why this works

* Bitmap → cheap logical composition
* Selection vector → efficient final execution

👉 Best of both worlds

---

## 🌍 Real-World Systems

### DuckDB / Vectorized engines

* Use **bitmaps for filtering**
* Convert to **selection vectors for execution**

### Apache Arrow

* Uses **bitmaps extensively** (validity + filters)

### ClickHouse

* Similar idea with **filter masks**

### PostgreSQL (modern paths)

* Moving toward **vectorized + bitmap-style filtering**

---

## ⚠️ Common Misunderstanding

*(Applies to Selection Vectors)*

> “They’re always faster because they skip rows”

Not if:

* most rows match
* or you repeatedly rebuild them

Then bitmap can be cheaper.

---

## 🧪 Elite Secret

High-end engines often:

> Delay materializing selection vectors as long as possible

### Why?

* Bitmaps can be combined **without branching**
* Selection vectors introduce **indirection**

So:

* bitmap early
* selection late

---

## 🔄 Breakthrough Reframe (Rust Lens)

Rust makes this tradeoff very tangible:

### Selection vector

```rust
Vec<usize> // sparse, pointer chasing
```

### Bitmap

```rust
Vec<bool> // (or packed bits) dense, linear scan
```

You’re choosing between:

> pointer chasing vs linear memory scan

…and modern CPUs *love* linear scans.