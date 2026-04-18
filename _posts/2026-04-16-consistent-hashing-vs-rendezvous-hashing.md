---
layout: post
title: "Consistent Hashing vs Rendezvous Hashing"
date: 2026-04-16 10:00:00 +0000
tags: [distributed-systems, hashing, load-balancing, system-design]
categories: rust
---

# Consistent Hashing vs Rendezvous Hashing (Partitioning Without Coordination)

This is a **core building block** behind:

> “How do we distribute data across nodes… without reshuffling everything?”

* **Consistent Hashing** → ring-based, minimal movement
* **Rendezvous Hashing (HRW)** → score-based, simpler and often better

---

## 🧠 Core Idea

**Consistent hashing** maps both nodes and keys onto a ring. A key belongs to the next node clockwise. When a node is added/removed, only a small portion of keys move.

**Rendezvous hashing** skips the ring entirely. For each key, you compute a score for every node and pick the highest. It sounds expensive—but in practice it’s simple, flexible, and avoids ring quirks.

---

## 🦀 Technique A — Consistent Hashing (Ring-Based)

```rust
use std::collections::BTreeMap;
use std::hash::{Hash, Hasher};
use std::collections::hash_map::DefaultHasher;

fn hash<T: Hash>(t: &T) -> u64 {
    let mut h = DefaultHasher::new();
    t.hash(&mut h);
    h.finish()
}

struct ConsistentHash {
    ring: BTreeMap<u64, String>, // position -> node
}

impl ConsistentHash {
    fn new() -> Self {
        Self {
            ring: BTreeMap::new(),
        }
    }

    fn add_node(&mut self, node: String) {
        let h = hash(&node);
        self.ring.insert(h, node);
    }

    fn get_node(&self, key: &str) -> Option<&String> {
        let h = hash(&key);

        self.ring
            .range(h..)
            .next()
            .or_else(|| self.ring.iter().next())
            .map(|(_, node)| node)
    }
}

fn main() {
    let mut ch = ConsistentHash::new();
    ch.add_node("node-a".into());
    ch.add_node("node-b".into());
    ch.add_node("node-c".into());

    println!("{:?}", ch.get_node("user-123"));
}
```

### 💡 Key Properties

* Only ~**1/N keys move** when cluster changes
* Needs **virtual nodes** for good balance

---

## 🦀 Technique B — Rendezvous Hashing (Highest Random Weight)

```rust
use std::hash::{Hash, Hasher};
use std::collections::hash_map::DefaultHasher;

fn hash<T: Hash>(t: &T) -> u64 {
    let mut h = DefaultHasher::new();
    t.hash(&mut h);
    h.finish()
}

fn rendezvous<'a>(key: &str, nodes: &'a [String]) -> Option<&'a String> {
    nodes.iter().max_by_key(|node| {
        hash(&(key, node))
    })
}

fn main() {
    let nodes = vec![
        "node-a".to_string(),
        "node-b".to_string(),
        "node-c".to_string(),
    ];

    let owner = rendezvous("user-123", &nodes);
    println!("{:?}", owner);
}
```

### 💡 Key Properties

* No ring, no structure
* Just compute → pick max
* Naturally supports **top-K replicas**

---

## ⚖️ Tradeoffs

| Dimension    | Consistent Hashing (A) | Rendezvous (B)     |
| ------------ | ---------------------- | ------------------ |
| Lookup cost  | ✅ O(log N)             | ❌ O(N)             |
| Simplicity   | ❌ complex (ring)       | ✅ very simple      |
| Load balance | ⚠️ needs tuning        | ✅ naturally good   |
| Node changes | ✅ minimal movement     | ✅ minimal movement |
| Memory       | ❌ ring storage         | ✅ none             |
| Replication  | awkward                | ✅ trivial (top-K)  |

---

## 🎯 When to Use

### Pick Consistent Hashing when:

* You need **fast lookup at scale**
* Node set is **large and relatively stable**
* You can afford **ring management**

👉 Example: distributed caches (older designs), memcached clients

---

### Pick Rendezvous Hashing when:

* **Simplicity matters**
* You want **easy replication (top-K nodes)**
* Node set is **moderate**

👉 Example: modern load balancers, sharded services

---

## 🤝 Combination Pattern (Modern systems lean this way 🔥)

```
[Key]
  ↓
[Rendezvous hashing → pick top-K nodes]
  ↓
[Local structures per node (cache / storage)]
```

### Why this wins

* No global structure to maintain
* Easy failover (just pick next highest)
* Clean mental model

---

## 🌍 Real-World Systems

* **Cassandra / Dynamo-style systems**

  * Use variants of **consistent hashing + virtual nodes**

* **Envoy / modern load balancers**

  * Prefer **rendezvous hashing**

* **Kafka**

  * Partitioning is simpler, but similar ideas apply

* **Distributed caches (CDNs, KV stores)**

  * Increasingly moving toward **HRW-style approaches**

---

## ⚠️ Common Misunderstanding

> “The ring automatically balances load”

Not really.

Without **virtual nodes**, you can get:

* hot spots
* uneven distribution

---

## 🧪 Elite Secret

Rendezvous hashing unlocks this pattern:

> “Compute multiple winners, not just one”

```rust
nodes.iter()
    .map(|n| (n, hash(&(key, n))))
    .sorted_by_key(|(_, score)| *score)
    .rev()
    .take(3) // replicas
```

Now replication, failover, and load spreading are all *the same operation*.

---

## 🔁 Breakthrough Reframe (Rust Lens)

Rust makes a key distinction obvious:

* Consistent hashing → **stateful structure** (`BTreeMap`)
* Rendezvous → **pure function** (`Fn(key, nodes) -> node`)

That’s huge.

👉 One requires synchronization
👉 The other is embarrassingly parallel

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [distributed-systems](/tags/distributed-systems/), [hashing](/tags/hashing/), [load-balancing](/tags/load-balancing/), [system-design](/tags/system-design/)
<!-- gh-pages-taxonomy-links:end -->
