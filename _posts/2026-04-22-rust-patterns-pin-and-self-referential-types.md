---

layout: post
title: "Rust Patterns: Pin and Self-Referential Types"
date: 2026-04-22 00:00:00 +0000
tags: [rust, memory-safety, async, systems-programming]
categories: rust
---

# 🦀 Today’s Patterns: `Pin` + Self-Referential Types

This is where Rust enforces **memory stability over time**.

---

## 📌 Pattern 1: `Pin` (Preventing Moves)

### 💡 Idea

`Pin` ensures a value **cannot be moved in memory after being pinned**.

```rust
use std::pin::Pin;

struct Data {
    value: String,
}

fn use_pinned(data: Pin<&mut Data>) {
    // cannot move `data` out of its location
}
```

---

## 🌍 Real-World Context

* `Future` in async Rust (`.await`)
* Generators
* `tokio`, `async-std`
* Low-level systems code

Rust uses `Pin` because:

> some values must stay at a fixed memory address

---

## ⚠️ Common Misunderstanding

> "Pin prevents all mutation"

**No — it prevents movement, not mutation.**

You can still mutate safely:

```rust
fn mutate(data: Pin<&mut Data>) {
    let data = unsafe { data.get_unchecked_mut() };
    data.value.push_str("!");
}
```

---

## 🧠 Elite Insight

Moving a value = **bitwise copy to a new location**

* For most types, that’s fine
* For some types, it breaks internal references

---

## 🔄 Breakthrough Reframe

> `Pin` is about *where* a value lives, not *who* owns it.

---

## 🧵 Pattern 2: Self-Referential Types

### 💡 Idea

A struct that contains a reference to **itself**.

```rust
struct Bad {
    data: String,
    slice: *const str, // points into `data`
}
```

### ❓ Why is this dangerous?

```rust
let mut x = Bad { /* ... */ };
let y = x; // move!

// ❌ pointer inside `slice` is now invalid
```

---

## 🌍 Real-World Context

* `Future` state machines (store references into themselves)
* Parsers with internal buffers
* Generators

Rust avoids these directly because:

> moving them would invalidate internal references

---

## ⚠️ Common Misunderstanding

> "Rust doesn’t allow self-referential structs"

It **does**, but:

* only safely with `Pin`
* or via crates (`ouroboros`, etc.)

---

## 🧠 Elite Insight

`async fn` secretly creates a **self-referential state machine**:

```rust
async fn example() {
    let s = String::from("hello");
    let r = &s;
    await_something().await;
    println!("{}", r);
}
```

👉 The future stores:

* `s`
* reference `r` → pointing into `s`

This requires:

> the future must not move after polling begins

---

## 🔄 Breakthrough Reframe

> Self-referential types are **time bombs** unless memory is stable.

---

## ⚖️ `Pin` vs Self-Referential Types

### 🟢 Pros

#### `Pin`

* Guarantees memory stability
* Enables async/await
* Works with safe abstractions

#### Self-Referential Types

* Powerful internal optimizations
* Avoid allocations
* Enable complex state machines

---

### 🔴 Cons

#### `Pin`

* Complex API (`Pin<&mut T>`)
* Requires `unsafe` to implement correctly
* Hard mental model

#### Self-Referential Types

* Unsafe without `Pin`
* Very easy to get wrong
* Not ergonomic

---

## 🎯 When to Use `Pin`

* Implementing `Future`
* Writing async runtimes
* Low-level libraries

---

## 🧵 When to Use Self-Referential Types

* Rarely directly
* Mostly via:

  * async/await
  * libraries that hide complexity

---

## 🤝 When to Combine Them (🔥 Essential)

This is exactly how async works:

* self-referential future
* pinned in memory
* safely polled

Conceptually:

```rust
struct FutureStateMachine {
    buffer: String,
    pointer_into_buffer: *const str,
}
```

👉 `Pin` ensures:

> this struct never moves → pointer stays valid

---

## ⚔️ When It’s Overkill

* Most application code
* Simple data structures
* Anything not dealing with internal references