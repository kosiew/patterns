---

layout: post
title: "Rust: impl Trait vs dyn Trait"
date: 2026-04-18 00:00:00 +0000
tags: [rust, traits, generics, performance, system-design]
categories: rust
---

## ⚡ Pattern 1: `impl Trait` (Static Dispatch)

### 💡 Idea

Use generics so the compiler knows the exact type at compile time.

```rust
fn print_all(items: impl Iterator<Item = i32>) {
    for item in items {
        println!("{}", item);
    }
}
```

Or returning:

```rust
fn evens() -> impl Iterator<Item = i32> {
    (0..10).filter(|x| x % 2 == 0)
}
```

---

### 🌍 Real-World Context

* Most of the standard library
* Iterator-heavy APIs
* High-performance systems code
* `async fn` (returns `impl Future` under the hood)

Rust prefers this because:

> it enables zero-cost abstractions via monomorphization

---

### ⚠️ Common Misunderstanding

> "`impl Trait` is just nicer syntax for generics"

Not quite:

* In arguments → sugar for generics
* In returns → **opaque type** (caller does not know the concrete type)

---

### 🧠 Elite Secret

Each use of `impl Trait` creates a separate monomorphized version:

```rust
print_all(vec![1, 2, 3].into_iter());
print_all([1, 2, 3].into_iter());
```

👉 These compile into **different functions**.

This is why:

* it's fast
* but increases binary size

---

### 🔁 Breakthrough Reframe

> `impl Trait` = "compile-time polymorphism with zero runtime cost"

---

## 🎭 Pattern 2: `dyn Trait` (Dynamic Dispatch)

### 💡 Idea

Use trait objects when the concrete type is unknown at compile time.

```rust
fn print_all(items: Box<dyn Iterator<Item = i32>>) {
    for item in items {
        println!("{}", item);
    }
}
```

---

### 🌍 Real-World Context

* Plugin systems
* Heterogeneous collections
* GUI frameworks
* Dependency injection patterns

Rust uses this when:

> flexibility matters more than raw performance

---

### ⚠️ Common Misunderstanding

> "`dyn Trait` is slow"

Reality:

* small indirection cost (vtable lookup)
* often negligible unless in tight loops

---

### 🧠 Elite Secret

`dyn Trait` is a **fat pointer**:

```rust
// Conceptually:
(data_ptr, vtable_ptr)
```

👉 The vtable contains function pointers for method calls.

---

### 🔁 Breakthrough Reframe

> `dyn Trait` = "runtime polymorphism with controlled cost"

---

## ⚖️ `impl Trait` vs `dyn Trait`

### 🟢 Pros

#### `impl Trait`

* Zero runtime overhead
* Inlinable, optimizable
* Best performance
* Works great with iterators, async

#### `dyn Trait`

* Type erasure
* Smaller binaries (no monomorphization explosion)
* Can store mixed types together
* More flexible APIs

---

### 🔴 Cons

#### `impl Trait`

* Code bloat (many monomorphized versions)
* Cannot easily store heterogeneous types
* Return type must be a single concrete type

#### `dyn Trait`

* Runtime dispatch cost
* No inlining
* Requires heap allocation (`Box`, `Arc`, etc.) in many cases

---

## 🎯 When to Use `impl Trait`

* Performance-critical paths
* Iterator chains
* `async` code
* Library APIs where types can stay concrete

---

## 🎭 When to Use `dyn Trait`

* Plugin architectures
* Collections of mixed types
* Reducing compile time / binary size
* Stable ABI boundaries

---

## 🤝 When to Combine Them (🔥 Real-World)

Very common pattern:

```rust
fn make_iter(flag: bool) -> Box<dyn Iterator<Item = i32>> {
    if flag {
        Box::new((0..5).into_iter())
    } else {
        Box::new((5..10).into_iter())
    }
}
```

👉 Why not `impl Iterator`?

Because:

> `impl Trait` must return **one concrete type**

---

## ⚔️ When It's Overkill

* Using `dyn Trait` inside tight loops
* Using `impl Trait` when you actually need heterogeneity
* Premature abstraction before requirements are clear