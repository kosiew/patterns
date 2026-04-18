---

layout: post
title: "Iterator Patterns: collect(), FromIterator, and Builder Patterns"
date: 2026-04-17 07:14:15 +0000
tags: [rust, iterators, patterns, error-handling]
categories: rust
---

# 🔁 Iterator Patterns: `collect()`, `FromIterator`, and Builders

Rust’s iterator ecosystem is more than loops — it’s a **compositional system for transforming data and errors**.

This post explores three powerful patterns:

* `Iterator<Item = Result<T, E>>`
* `collect::<Result<Vec<T>, E>>()`
* Implementing `FromIterator`

---

# 🧩 Pattern 1: Iterator of `Result`

Instead of failing inside a loop, return a `Result` per item and compose.

```rust
fn parse_all(lines: &[&str]) -> Vec<Result<i32, std::num::ParseIntError>> {
    lines.iter()
        .map(|line| line.parse::<i32>())
        .collect()
}
```

Each element becomes:

* `Ok(value)`
* `Err(error)`

---

## 🌍 Real-World Context

* Parsing files
* Config loading
* Deserialization
* Batch processing

---

## 🧠 Insight

> Keep failures local — compose them later.

---

# ✨ Pattern 2: `collect::<Result<Vec<T>, E>>()`

Transform many small results into one clean result.

```rust
fn parse_all(lines: &[&str]) -> Result<Vec<i32>, std::num::ParseIntError> {
    lines.iter()
        .map(|line| line.parse::<i32>())
        .collect()
}
```

## What happens?

* All `Ok` → `Ok(Vec<T>)`
* First `Err` → return error immediately

---

## 🧠 Why it works

Rust provides:

```rust
impl<T, E, C> FromIterator<Result<T, E>> for Result<C, E>
```

This is what powers the pattern.

---

## ⚠️ Common Misunderstanding

> "collect only builds Vec"

It can build:

* `Vec<T>`
* `HashMap<K, V>`
* `String`
* `Result<Vec<T>, E>`
* `Option<Vec<T>>`

---

## 🧠 Bonus: Works with `Option`

```rust
let nums: Option<Vec<i32>> =
    vec![Some(1), Some(2), Some(3)]
        .into_iter()
        .collect();
```

---

# 🏗 Pattern 3: Implementing `FromIterator`

Make your own types work with `.collect()`.

## Example: Sum type

```rust
struct Sum(i32);

impl std::iter::FromIterator<i32> for Sum {
    fn from_iter<I: IntoIterator<Item = i32>>(iter: I) -> Self {
        let mut total = 0;
        for value in iter {
            total += value;
        }
        Sum(total)
    }
}
```

Now:

```rust
let s: Sum = vec![1, 2, 3].into_iter().collect();
```

---

## 🧠 Insight

> `collect()` turns a stream into a structure.

---

# 🧱 Pattern 4: Builder-as-Collector

Use `FromIterator` to enforce constraints.

```rust
struct NonEmptyVec(Vec<i32>);
```

### Simple version

```rust
impl std::iter::FromIterator<i32> for NonEmptyVec {
    fn from_iter<I: IntoIterator<Item = i32>>(iter: I) -> Self {
        let vec: Vec<i32> = iter.into_iter().collect();
        if vec.is_empty() {
            panic!("cannot be empty");
        }
        NonEmptyVec(vec)
    }
}
```

---

### Better: fallible builder

```rust
impl NonEmptyVec {
    fn try_from_iter<I>(iter: I) -> Result<Self, &'static str>
    where
        I: IntoIterator<Item = i32>,
    {
        let vec: Vec<i32> = iter.into_iter().collect();
        if vec.is_empty() {
            Err("cannot be empty")
        } else {
            Ok(Self(vec))
        }
    }
}
```

---

# ⚖️ `collect()` vs Manual Loop

## Pros (Iterator + collect)

* Concise
* Early exit on error
* Composable with `?`
* Zero-cost abstraction

## Cons

* Harder to understand initially
* Less step-by-step control

---

## When to Use

* Parsing and validation
* Straight-line transformations
* Library APIs

---

## When to Avoid

* Complex control flow
* Partial success handling
* Multiple error accumulation

---

# ⭐ Advanced Combination

```rust
let values: Vec<i32> = lines
    .iter()
    .map(|l| l.parse())
    .collect::<Result<_, _>>()?;
```

---

# 🧠 Final Takeaway

* Iterators express *data flow*
* `collect()` expresses *structure building*
* `FromIterator` lets you define your own semantics

> Rust lets you turn loops into pipelines — without l

<!-- gh-pages-taxonomy-links:start -->
Categories: [rust](/categories/rust/)
Tags: [rust](/tags/rust/), [iterators](/tags/iterators/), [patterns](/tags/patterns/), [error-handling](/tags/error-handling/)
<!-- gh-pages-taxonomy-links:end -->
