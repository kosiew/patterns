---

layout: post
title: "Advanced Python Patterns: Iteration, Data Flow & Object Semantics"
date: 2026-04-17 07:40:42 +0000
tags: [python, iterators, functional, data-structures, performance]
categories: python
---

## Overview

This guide explores advanced Python patterns across:

* Iteration and data flow
* Functional vs Pythonic styles
* Object model internals
* Performance and memory tradeoffs

---

# 🔗 `itertools.chain` vs List Concatenation

## `chain` (Lazy)

```python
from itertools import chain

for x in chain(a, b, c):
    process(x)
```

* zero-copy
* works with any iterable

## Concatenation (Eager)

```python
for x in a + b + c:
    process(x)
```

* builds intermediate list

### 🧠 Insight

> `chain` describes a data stream, not a container

---

# 🔄 `map` / `filter` vs Comprehensions

## Comprehension (Preferred)

```python
[x.strip().lower() for x in data if x]
```

## Functional Style

```python
list(map(normalize, filter(bool, data)))
```

### 🧠 Insight

> Comprehensions show the *result*, `map/filter` show the *pipeline*

---

# 🏗️ `__new__` vs `__init__`

* `__new__` → creates object
* `__init__` → initializes object

### 🧠 Insight

> Identity vs state

---

# ⚖️ EAFP vs LBYL

## EAFP (Pythonic)

```python
try:
    value = d["key"]
except KeyError:
    value = 0
```

## LBYL

```python
if "key" in d:
    value = d["key"]
```

### 🧠 Insight

> React to reality, don’t predict failure

---

# 🧾 `__repr__` vs `__str__`

* `__repr__` → debugging
* `__str__` → user display

### 🧠 Insight

> `__repr__` is your debugging API

---

# 🧮 `__eq__` vs `__hash__`

* equality defines sameness
* hash defines storage location

### Rule

```text
if a == b → hash(a) must equal hash(b)
```

---

# 🔧 `partial` vs `lambda`

* `partial` → configuration
* `lambda` → behavior

---

# 🔢 `enumerate` vs `range(len(...))`

## Preferred

```python
for i, item in enumerate(items):
    ...
```

### 🧠 Insight

> Iterate over data, not indices

---

# 🔗 `zip` vs `zip_longest`

* `zip` → strict alignment
* `zip_longest` → tolerant alignment

---

# 🧱 `__slots__` vs `NamedTuple`

* `__slots__` → mutable, controlled object
* `NamedTuple` → immutable value

---

# 🧬 `super()` vs Direct Calls

### 🧠 Insight

> `super()` means “next in MRO”, not “parent”

---

# 🔁 `__iter__` vs `__getitem__`

* `__iter__` → explicit streaming
* `__getitem__` → index fallback

---

# 📦 `setdefault` vs `defaultdict`

* `setdefault` → explicit
* `defaultdict` → automatic

---

# 🔄 Generators vs Lists

* generators → lazy
* lists → eager

---

# 🔗 `yield` vs `yield from`

* `yield` → manual
* `yield from` → delegation

---

# ✂️ `islice` vs slicing

* `islice` → lazy
* slicing → eager

---

# 🔀 `tee` vs recreating iterators

* `tee` → splits stream
* recreate → simpler, no buffering

---

# 📊 `Counter` vs dict

* `Counter` → multiset operations
* dict → manual logic

---

# 🏆 `heapq` vs `sorted`

* `heapq` → top-K (O(n log k))
* `sorted` → full order (O(n log n))

---

# 📍 `bisect` vs binary search

* `bisect` → insertion-ready
* manual → custom logic

---

# ⚡ `lru_cache` vs memoization

* decorator vs manual cache

---

# 🔁 `reduce` vs loops

* `reduce` → functional
* loop → readable

---

# 🔘 `__call__` vs methods

* `__call__` → function-like object
* method → explicit API

---

# 🔐 Context Managers

## Class-based

* full control

## `@contextmanager`

* concise

---

# 🧠 Final Principles

* Prefer **lazy data flow** over eager allocation
* Prefer **clarity over cleverness**
* Understand **object semantics deeply**
* Use abstractions that match **intent, not habit**

> Great Python

<!-- gh-pages-taxonomy-links:start -->
Categories: [python](/categories/python/)
Tags: [python](/tags/python/), [iterators](/tags/iterators/), [functional](/tags/functional/), [data-structures](/tags/data-structures/), [performance](/tags/performance/)
<!-- gh-pages-taxonomy-links:end -->
