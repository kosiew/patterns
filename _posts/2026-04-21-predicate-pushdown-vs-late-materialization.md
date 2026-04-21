---

layout: post
title: "Predicate Pushdown vs Late Materialization"
date: 2026-04-21 00:00:00 +0000
tags: [databases, query-optimization, columnar-storage, performance, system-design]
categories: rust
---

# Predicate Pushdown vs Late Materialization

*Don’t Touch Data vs Touch Less Data*

Now we unlock a *huge performance lever* in analytics systems:

> “How do we avoid doing unnecessary work when scanning data?”

* **Predicate Pushdown** → skip irrelevant data early
* **Late Materialization** → delay expensive data access as long as possible

These are *core to why columnar systems feel “magically fast”*.

---

## 🧠 Core Idea

**Predicate pushdown** moves filtering as close to the data source as possible. Instead of reading everything and filtering later, you *avoid reading irrelevant data entirely*.

**Late materialization** delays fetching full records (or expensive columns) until you're sure you need them. You operate on lightweight representations (like IDs or bitmaps) first.

---

## 🦀 Technique A — Predicate Pushdown (Filter Early)

```rust
fn predicate_pushdown(data: &[i32]) -> Vec<i32> {
    data.iter()
        .filter(|&&x| x > 10) // filter early
        .map(|&x| x * 2)
        .collect()
}

fn main() {
    let data = vec![5, 20, 3, 15, 8];
    let result = predicate_pushdown(&data);
    println!("{:?}", result);
}
```

### 💡 Key Property

* Reduces data volume immediately
* Prevents unnecessary downstream work

---

## 🦀 Technique B — Late Materialization (Filter IDs First)

```rust
fn late_materialization(values: &[i32], payloads: &[String]) -> Vec<String> {
    // Step 1: filter lightweight column
    let selected_indices: Vec<usize> = values
        .iter()
        .enumerate()
        .filter(|(_, &v)| v > 10)
        .map(|(i, _)| i)
        .collect();

    // Step 2: fetch expensive data only for matches
    selected_indices
        .into_iter()
        .map(|i| payloads[i].clone())
        .collect()
}

fn main() {
    let values = vec![5, 20, 3, 15, 8];
    let payloads = vec![
        "a".to_string(),
        "b".to_string(),
        "c".to_string(),
        "d".to_string(),
        "e".to_string(),
    ];

    let result = late_materialization(&values, &payloads);
    println!("{:?}", result);
}
```

### 💡 Key Property

* Avoids touching **heavy data** (strings, blobs)
* Works beautifully with **columnar layouts**

---

## ⚖️ Tradeoffs

| Dimension        | Predicate Pushdown (A) | Late Materialization (B)     |
| ---------------- | ---------------------- | ---------------------------- |
| Data scanned     | ✅ minimal              | ⚠️ still scans filter column |
| CPU usage        | ✅ reduced early        | ✅ reduced on heavy columns   |
| Memory bandwidth | ✅ lowest               | ✅ very low                   |
| Complexity       | low                    | moderate                     |
| Flexibility      | depends on storage     | very flexible                |
| Best case        | selective filters      | wide rows, expensive columns |

---

## 🎯 When to Use

### Pick Predicate Pushdown when:

* You can filter at the **storage layer**
* Data source supports it (files, DB, index)
* Filters are **highly selective**

👉 Example: `WHERE user_id = 42` in a Parquet scan

### Pick Late Materialization when:

* Rows are **wide** (many columns)
* Some columns are **expensive** (strings, JSON, blobs)
* You only need a subset after filtering

👉 Example: search results where only top matches need full payload

---

## 🤝 Combination Pattern (This Is Where Systems Win Big 🔥)

```text
[Columnar storage]
        ↓
[Predicate pushdown]   ← skip chunks entirely
        ↓
[Scan lightweight columns]
        ↓
[Late materialization] ← fetch heavy columns last
```

### Why this is powerful

* Pushdown → *don’t read data*
* Late materialization → *don’t decode data*

👉 Together: **minimum possible work**

---

## 🌍 Real-World Systems

### ClickHouse / DuckDB

* Aggressive **predicate pushdown**
* Heavy use of **late materialization**

### Apache Parquet / Arrow

* Pushdown via **min/max statistics**
* Columnar layout enables **late materialization**

### Elasticsearch

* Filters first, then fetches `_source` later

### Spark SQL

* Pushdown into file readers + column pruning

---

## ⚠️ Common Misunderstanding

*(Applies to Predicate Pushdown)*

> “Filtering is cheap, just do it anywhere”

No — *where* you filter matters more than *how*.

Filtering late means:

* wasted IO
* wasted CPU
* worse cache behavior

---

## 🧪 Elite Secret

Late materialization often uses:

> **selection vectors** (`Vec<usize>`) instead of copying rows

That tiny decision:

* avoids allocations
* avoids moving large structs
* keeps pipelines vectorized

---

## 🔄 Breakthrough Reframe (Rust Lens)

Rust makes this pattern *beautifully explicit*:

```rust
let selected: Vec<usize> = ...; // lightweight selection

for &i in &selected {
    process(&heavy_column[i]);
}
```

Instead of:

```rust
Vec<Row> // expensive copies
```

👉 You’re not moving data — you’re moving **indexes**.