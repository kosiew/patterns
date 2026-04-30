---

layout: post
title: "Arc<Mutex<T>> vs RwLock<T> in Rust"
date: 2026-04-30 00:00:00 +0000
tags: [rust, concurrency, synchronization, systems-programming]
categories: rust
---

## 🦀 Today’s Patterns: `Arc<Mutex<T>>` vs `RwLock<T>`

These are the backbone of shared state in concurrent Rust.

---

## 🔒 Pattern 1: `Arc<Mutex<T>>`

### 💡 Idea

Multiple threads share ownership (`Arc`) and synchronize access with a **mutual exclusion lock**.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

let counter = Arc::new(Mutex::new(0));

let handles: Vec<_> = (0..3).map(|_| {
    let counter = Arc::clone(&counter);
    thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    })
}).collect();

for h in handles {
    h.join().unwrap();
}
```

### 🌍 Real-World Context

* Shared counters
* Task queues
* Global state in servers
* `tokio::sync::Mutex` (async version)

> Rust uses this because:
> it guarantees **only one mutable access at a time across threads**

### ⚠️ Common Misunderstanding

> “Mutex is slow”

Not inherently:

* uncontended → very fast
* contention → can become a bottleneck

### 🧠 Elite Secret

The real power:

```rust
let guard = counter.lock().unwrap();
```

👉 This returns a **RAII guard**

* lock acquired here
* automatically released on drop

Even on panic 🔥

### 🔁 Breakthrough Reframe

> `Mutex<T>` is just `RefCell<T>` — but for threads.

---

## 📖 Pattern 2: `RwLock<T>` (Read-Write Lock)

### 💡 Idea

Allow:

* many readers (`&T`)
* **OR** one writer (`&mut T`)

```rust
use std::sync::{Arc, RwLock};
use std::thread;

let data = Arc::new(RwLock::new(5));

let reader = {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        let val = data.read().unwrap();
        println!("read: {}", *val);
    })
};

let writer = {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        let mut val = data.write().unwrap();
        *val += 1;
    })
};

reader.join().unwrap();
writer.join().unwrap();
```

### 🌍 Real-World Context

* Caches
* Configuration data
* Read-heavy workloads
* Databases, in-memory stores

> Rust uses this because:
> reads can scale without blocking each other

### ⚠️ Common Misunderstanding

> “RwLock is always better than Mutex”

Nope:

* write-heavy workloads → worse performance
* more complex locking behavior

### 🧠 Elite Secret

`RwLock` can suffer from **writer starvation**:

👉 If readers keep coming, writers may wait indefinitely (depending on implementation)

### 🔁 Breakthrough Reframe

> `RwLock` is about *throughput optimization*, not just correctness.

---

## ⚖️ `Arc<Mutex<T>>` vs `RwLock<T>`

### 🟢 Pros

#### `Arc<Mutex<T>>`

* Simple mental model
* Predictable behavior
* No reader/writer complexity
* Works well in most cases

#### `RwLock<T>`

* Parallel reads
* Better for read-heavy systems
* Higher throughput when contention is low

### 🔴 Cons

#### `Arc<Mutex<T>>`

* Blocks all access (even reads)
* Can bottleneck under contention

#### `RwLock<T>`

* More complex
* Potential starvation
* Worse under write-heavy load

---

## 🎯 When to Use `Arc<Mutex<T>>`

* Mixed read/write workloads
* Simpler systems
* When correctness > optimization
* Default choice

---

## 📘 When to Use `RwLock<T>`

* Many reads, few writes
* Performance-sensitive read paths
* Shared configuration/state

---

## 🤝 When to Combine Them (🔥 Real Systems)

You’ll often see:

```rust
struct AppState {
    cache: RwLock<HashMap<String, String>>,
    stats: Mutex<u64>,
}
```

👉 Use:

* `RwLock` for read-heavy data
* `Mutex` for write-heavy or simple counters

---

## ⚔️ When It’s Overkill

* Single-threaded code (`RefCell` instead)
* Low contention scenarios
* Premature optimization