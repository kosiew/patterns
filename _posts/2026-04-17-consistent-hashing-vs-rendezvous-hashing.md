---

layout: post
title: "Consistent Hashing vs Rendezvous Hashing"
date: 2026-04-17 07:11:05 +0000
tags: [distributed-systems, hashing, load-balancing, system-design]
categories: rust
---

# 🔀 Consistent Hashing vs Rendezvous Hashing

Two classic techniques for **stable key distribution** in distributed systems:

* **Consistent Hashing** (ring-based)
* **Rendezvous Hashing (HRW)** (score-based)

Both aim to:

* distribute keys evenly
* minimize movement when nodes join/leave
* avoid central coordination

---

## 🧠 Short Explanation

**Consistent hashing** places nodes on a ring; keys map to the next node clockwise.

**Rendezvous hashing** computes a score for each `(key, node)` pair and selects the highest.

> Same goal: stable partitioning.
> Different tradeoff: ring structure vs scoring simplicity.

---

# 🧩 Technique A — Consistent Hashing (Ring-Based)

## Concept

```text
Hash space (0..2^64)
    ↓
Nodes placed on ring
    ↓
Key hashes to position
    ↓
Pick next node clockwise
```

Virtual nodes improve balance.

## Properties

* O(log N) lookup (tree search)
* Requires virtual nodes for good balance
* More complex structure
* Widely used in older systems

---

# 🎯 Technique B — Rendezvous Hashing (HRW)

## Concept

```text
score(node) = hash(key, node)
pick node with max score
```

No ring. No virtual nodes required.

---

## Rust Example

```rust
use std::hash::{Hash, Hasher};
use std::collections::hash_map::DefaultHasher;

fn hash_pair<K: Hash, N: Hash>(key: &K, node: &N) -> u64 {
    let mut h = DefaultHasher::new();
    key.hash(&mut h);
    node.hash(&mut h);
    h.finish()
}

fn rendezvous<K: Hash>(key: &K, nodes: &[&str]) -> Option<&str> {
    nodes
        .iter()
        .max_by_key(|node| hash_pair(key, node))
        .copied()
}
```

---

# ⚖️ Tradeoffs

| Dimension       | Consistent Hashing | Rendezvous Hashing |
| --------------- | ------------------ | ------------------ |
| Lookup cost     | O(log N)           | O(N)               |
| Data structure  | Ring               | None               |
| Balance quality | Needs tuning       | Naturally good     |
| Implementation  | Complex            | Simple             |
| Node removal    | Local remap        | Local remap        |
| Weighted nodes  | Harder             | Easier             |

---

# 🧭 When to Use Which

## Use Consistent Hashing when:

* very large node counts
* need predictable structure
* legacy compatibility matters

## Use Rendezvous Hashing when:

* moderate cluster size
* want simple implementation
* need weighted nodes
* prefer easier reasoning

> In modern systems, Rendezvous hashing is often preferred.

---

# 🔗 Combination Pattern

Common modern setup:

```text
key
  ↓
compute scores for nodes
  ↓
pick top-K nodes (replication)
```

* No ring traversal
* Easy replication
* Deterministic placement

---

# 🌍 Real-World Usage

* Apache Kafka — partition assignment
* Cassandra — consistent hashing ring
* ScyllaDB — shard distribution
* Redis — cluster slot hashing
* FoundationDB — locality-aware placement

---

# ❌ Common Misunderstanding

> “Consistent hashing guarantees perfect balance.”

Not without:

* enough virtual nodes
* good hash distribution
* proper weighting

Balance is statistical, not guaranteed.

---

# 🧠 Key Insight

In practice:

* Network latency dominates lookup complexity
* Node count is usually small (10–100)
* Simpler systems are easier to operate

> Simplicity often beats theoretical optimality.

---

# 🔄 Breakthrough Reframe

Treat sharding as a **pure function**:

* no shared mutable state
* deterministic mapping
* easy to test

Rust naturally encourages this model through ownership and immutability.

> Shard selection becomes computation — not coordination.

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [distributed-systems](/tags/distributed-systems/), [hashing](/tags/hashing/), [load-balancing](/tags/load-balancing/), [system-design](/tags/system-design/)
<!-- gh-pages-taxonomy-links:end -->
