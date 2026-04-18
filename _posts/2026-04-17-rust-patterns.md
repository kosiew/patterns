---

layout: post
title: "Rust patterns"
date: 2026-04-17 07:01:08 +0000
tags: [rust, systems-programming, design-patterns, ownership, safety]
categories: rust
---

# ♻️ Pattern 1: RAII (Resource Acquisition Is Initialization)

## 💡 Idea

Tie resource lifetime (file, lock, socket) directly to scope using **ownership + `Drop`**.

```rust
use std::fs::File;

fn read_file() {
    let file = File::open("data.txt").unwrap();

    // use file...

} // 🔥 file is automatically closed here (Drop runs)
```

Or more explicit:

```rust
struct Connection;

impl Drop for Connection {
    fn drop(&mut self) {
        println!("Connection closed");
    }
}
```

---

## 🌍 Real-World Context

* `MutexGuard` → unlocks automatically
* `File` → closes on drop
* `Vec` → frees memory
* `tokio::sync::MutexGuard`

---

## 🤔 Why Rust Prefers RAII

> Cleanup is guaranteed even in early returns or panics.

---

## ⚠️ Common Misunderstanding

> “Drop is for business logic”

**No — `Drop` is for cleanup, not:**

* retry logic
* network calls
* anything fallible

---

## 🧠 Elite Insight

RAII + borrowing gives you **temporal safety**:

```rust
let guard = mutex.lock();
// 🔒 locked

// cannot access mutex mutably elsewhere

// unlock happens automatically
```

> This is stronger than most languages:
>
> You cannot forget to release the lock — and you cannot misuse it while held.

---

## 🔁 Breakthrough Reframe

> Ownership doesn’t just model *who owns data* — it models **when things are valid**.

---

# 🔐 Pattern 2: Typestate (Lifecycle Encoding)

## 💡 Idea

Encode **valid states and transitions** explicitly in types.

```rust
struct Disconnected;
struct Connected;

struct Socket<State> {
    _state: std::marker::PhantomData<State>,
}

impl Socket<Disconnected> {
    fn connect(self) -> Socket<Connected> {
        Socket { _state: std::marker::PhantomData }
    }
}

impl Socket<Connected> {
    fn send(&self, data: &[u8]) {
        println!("Sending: {:?}", data);
    }
}
```

---

## 🌍 Real-World Context

* Network protocols (handshake → authenticated → streaming)
* Embedded drivers (uninitialized → configured → running)
* Unsafe abstractions (ensuring correct usage order)

### Rust favors this because:

> Illegal transitions become unrepresentable.

---

## ⚠️ Common Misunderstanding

> “Typestate replaces runtime checks entirely”

**Not always:**

* external systems can still fail
* you often combine typestate + `Result`

---

## 🧠 Elite Insight

Typestate + ownership enables **linear state transitions**:

```rust
let socket = Socket::<Disconnected> {};
let socket = socket.connect(); // old state consumed
```

👉 You literally cannot reuse the old state.

---

## 🔁 Breakthrough Reframe

> Typestate turns *time* into **types**.

---

# ⚖️ RAII vs Typestate

## 🟢 Pros

### RAII

* Automatic cleanup
* Minimal boilerplate
* Works everywhere
* Idiomatic Rust

### Typestate

* Enforces correct order of operations
* Eliminates misuse entirely
* Great for protocols & APIs

---

## 🔴 Cons

### RAII

* Doesn’t prevent misuse *before* drop
* Cannot enforce call order

### Typestate

* More complex API
* More types/generics
* Harder to evolve

---

## 🎯 When to Use RAII

* Resource cleanup (files, locks, memory)
* Scope-based safety
* Almost always — this is foundational

---

## 🔐 When to Use Typestate

* Order matters:

  * connect → authenticate → send
* Preventing invalid sequences
* Wrapping unsafe or low-level APIs

---

## 🤝 When to Combine Them (🔥 Powerful)

This is real-world Rust:

👉 Typestate ensures **correct usage**
👉 RAII ensures **correct cleanup**

### Example

```rust
struct Open;
struct Closed;

struct FileHandle<State> {
    file: Option<std::fs::File>,
    _state: std::marker::PhantomData<State>,
}

impl FileHandle<Open> {
    fn close(mut self) -> FileHandle<Closed> {
        self.file.take(); // drop happens
        FileHandle { file: None, _state: std::marker::PhantomData }
    }
}

impl Drop for FileHandle<Open> {
    fn drop(&mut self) {
        println!("File auto-closed!");
    }
}
```

👉 Even if you forget `.close()`:

* RAII saves you
* Typestate prevents misuse before that

---

# ⚔️ When It’s Overkill

* Simple file usage (`std::fs::File` already handles it)
* Short-lived resources
* Internal code with obvious flow

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [systems-programming](/tags/systems-programming/), [design-patterns](/tags/design-patterns/), [ownership](/tags/ownership/), [safety](/tags/safety/)
<!-- gh-pages-taxonomy-links:end -->
