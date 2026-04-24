---

layout: post
title: "Cell<T> vs RefCell<T> in Rust"
date: 2026-04-24 00:00:00 +0000
tags: [rust, ownership, borrowing, interior-mutability]
categories: rust
---

## 🧠 `Cell<T>` vs `RefCell<T>`

### 🔹 `Cell<T>` — Copy in / Copy out simplicity

* Works with `Copy` types, like `i32` and `bool`
* No borrowing — just **get/set values**
* No runtime borrow checking
* Super lightweight 🚀

```rust
use std::cell::Cell;

let x = Cell::new(5);
x.set(10);

let val = x.get(); // 10
```

👉 Think: *“just swap values, no references involved”*

---

### 🔸 `RefCell<T>` — Flexible but stricter

* Works with **any type**
* Uses `borrow()` / `borrow_mut()`
* Enforces borrow rules at **runtime**
* Can **panic** if misused ⚠️

```rust
use std::cell::RefCell;

let x = RefCell::new(vec![1, 2]);

x.borrow_mut().push(3);

println!("{:?}", x.borrow());
```

👉 Think: *“real references, but checked at runtime”*

---

## ⚖️ Key Differences

| Feature   | `Cell<T>`     | `RefCell<T>`            |
| --------- | ------------- | ----------------------- |
| Types     | `Copy` only   | Any type                |
| Borrowing | ❌ none        | ✅ runtime checked       |
| Overhead  | very low      | higher, checks + guards |
| Panics    | ❌ never       | ⚠️ possible             |
| Use case  | simple values | complex data structures |

---

## 🧭 Rule of Thumb

*He taps the table twice.*

* Use `Cell` → when you can copy the value
* Use `RefCell` → when you need real mutable references

---

## 🧩 Tiny Intuition Anchor

> `Cell` = “replace the value”
> `RefCell` = “borrow the value”

---

*He chuckles softly.*

If `Cell` is a sticky note you rewrite...

`RefCell` is a shared notebook with rules about who’s holding the pen 😄

---

## Two Kinds of Mutation in Rust

There are two kinds of mutation in Rust:

### 1. External Mutation, Normal Rust

```rust
let mut x = 5;
x = 10;
```

* Requires `mut`
* Checked at **compile time**
* Uses `&mut`

---

### 2. Interior Mutation, `Cell` / `RefCell`

```rust
let x = Cell::new(5);
x.set(10);
```

* No `mut` needed
* Happens **inside the container**
* Uses special types: `Cell`, `RefCell`

---

## 💡 Why This Is Allowed

Because `Cell` says:

> “I will handle mutation safely *internally*, you don’t worry about it.”

It does this using low-level tricks, `UnsafeCell` under the hood, but exposes a **safe API**.

---

## ⚖️ Why `Cell` Is Still Safe

Here’s the important part 👇

* `Cell` **never gives you references**: `&T` or `&mut T`
* You can only:

  * `get()` → copy value out
  * `set()` → replace value

So Rust avoids aliasing problems entirely.

---

## 🧩 Intuition

*He chuckles softly.*

Think of `Cell` like a sealed box:

* You can **put a new value in**
* You can **take a copy out**
* But you can never hold a reference into it

So no one can “see” inconsistent state.

---

## 🔁 The Clean Takeaway

> Yes — it is mutation
> But it’s **interior mutation**, not normal `mut` mutation

---

*He gives a small nod.*

The real mental shift with Rust is this:

**“immutability of the variable” ≠ “immutability of the data inside it.”**

And `Cell` is one of the clearest places where that clicks 😄