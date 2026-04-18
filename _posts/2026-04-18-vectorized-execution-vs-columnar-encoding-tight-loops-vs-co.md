---

layout: post
title: "Vectorized Execution vs Columnar Encoding (Tight Loops vs Compact Data)"
date: 2026-04-18 00:00:00 +0000
tags: [databases, analytics, performance, systems, rust, columnar, vectorization]
categories: database
---

## Vectorized Execution vs Columnar Encoding (Tight Loops vs Compact Data)

Now we step into *serious analytics engine territory*:

> “How do we make scans over millions of rows actually fast?”

* **Vectorized execution** → process data in batches (tight CPU loops)
* **Columnar encoding** → compress and structure data for memory efficiency

These two are *almost always paired* in modern systems.

---

## 🧠 Core Idea

**Vectorized execution** processes data in chunks (e.g., 1024 values at a time), enabling tight loops, SIMD, and fewer branches. It’s about *how you execute*.

**Columnar encoding** changes *how data is stored*: instead of rows, you store columns (often compressed with techniques like RLE or dictionary encoding). This reduces memory bandwidth—the real bottleneck.

---

## 🦀 Technique A — Vectorized Execution (Batch Processing Loop)

```rust
fn sum_vectorized(data: &[i32], chunk_size: usize) -> i32 {
    let mut total = 0;

    for chunk in data.chunks(chunk_size) {
        // tight loop over small slice
        let mut local_sum = 0;
        for &x in chunk {
            local_sum += x;
        }
        total += local_sum;
    }

    total
}

fn main() {
    let data: Vec<i32> = (0..1_000_000).collect();
    let result = sum_vectorized(&data, 1024);
    println!("{}", result);
}
```

### 💡 Key Property

* Reduces interpreter overhead
* Improves **cache locality**
* Enables SIMD/autovectorization

---

## 🦀 Technique B — Columnar Encoding (Dictionary Encoding Example)

```rust
use std::collections::HashMap;

struct DictionaryColumn {
    dict: Vec<String>,
    indices: Vec<usize>,
}

impl DictionaryColumn {
    fn encode(values: Vec<String>) -> Self {
        let mut dict = Vec::new();
        let mut map = HashMap::new();
        let mut indices = Vec::new();

        for v in values {
            let idx = match map.get(&v) {
                Some(&i) => i,
                None => {
                    let i = dict.len();
                    dict.push(v.clone());
                    map.insert(v.clone(), i);
                    i
                }
            };
            indices.push(idx);
        }

        Self { dict, indices }
    }

    fn get(&self, i: usize) -> &str {
        &self.dict[self.indices[i]]
    }
}

fn main() {
    let data = vec![
        "apple".to_string(),
        "banana".to_string(),
        "apple".to_string(),
    ];

    let col = DictionaryColumn::encode(data);

    println!("{}", col.get(0));
}
```

### 💡 Key Property

* Shrinks memory footprint
* Turns strings → integers (**faster comparisons!**)
* Enables better CPU cache usage

---

## ⚖️ Tradeoffs

| Dimension      | Vectorized (A)      | Columnar (B)           |
| -------------- | ------------------- | ---------------------- |
| CPU efficiency | ✅ very high         | ⚠️ depends on encoding |
| Memory usage   | ❌ same as input     | ✅ much lower           |
| Cache locality | ✅ good              | ✅ excellent            |
| Complexity     | low                 | higher                 |
| Flexibility    | works on any layout | needs column format    |
| Best case      | tight loops         | repeated values        |

---

## 🎯 When to Use

### Pick Vectorized Execution when:

* You already have data in memory
* You want to optimize **CPU execution**
* You control the processing loop

👉 Example: aggregation, filtering, arithmetic scans

---

### Pick Columnar Encoding when:

* Data is large and repetitive
* Memory bandwidth is the bottleneck
* You run analytical queries

👉 Example: OLAP systems, log analytics

---

## 🤝 Combination Pattern (This is the OLAP engine secret 🔥)

```
[Columnar storage]
        ↓
[Decode small batch]
        ↓
[Vectorized execution]
        ↓
[Aggregate/filter]
```

### Why this wins

* Columnar → reduces data size
* Vectorization → processes it efficiently

👉 This combo is why ClickHouse / DuckDB are *insanely fast*

---

## 🌍 Real-World Systems

**ClickHouse / DuckDB**

* Heavy use of **columnar + vectorized pipelines**

**Apache Arrow**

* Standardized **columnar in-memory format**

**Spark (modern engine)**

* Uses vectorized execution for performance

**PostgreSQL (recent improvements)**

* Adding vectorized paths in some operations

---

## ⚠️ Common Misunderstanding (Vectorization)

> “Vectorization = SIMD instructions”

Not necessarily.

Even without SIMD:

* fewer branches
* better cache usage
* less function call overhead

→ already huge gains

---

## 🧪 Elite Secret

The real bottleneck in analytics:

> **Memory bandwidth, not CPU**

Columnar encoding works because:

* you read fewer bytes
* not because computation is faster

---

## 🔄 Breakthrough Reframe (Rust Lens)

Rust makes this distinction very explicit:

* `&[T]` → perfect for **vectorized loops**
* `Vec<usize>` + dictionary → **encoded columns**

And crucially:

```rust
fn process_batch(data: &[i32])
```

forces you into:

* no allocations
* tight loops
* predictable performance