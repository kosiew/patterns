---

layout: post
title: "Decorators vs Context Managers in Python"
date: 2026-04-18 00:00:00 +0000
tags: [python, decorators, context-managers, programming-patterns, system-design]
categories: python
---

## ⚙️ Technique A — Decorator (Wrap Behavior at Definition Time)

```python
import time
from functools import wraps

def timing(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        end = time.perf_counter()
        print(f"{func.__name__} took {end - start:.4f}s")
        return result
    return wrapper

@timing
def compute():
    return sum(i * i for i in range(10_000))
```

---

## 🧠 Technique B — Context Manager (Wrap Behavior at Runtime Scope)

```python
import time
from contextlib import contextmanager

@contextmanager
def timing():
    start = time.perf_counter()
    yield
    end = time.perf_counter()
    print(f"Block took {end - start:.4f}s")

def compute():
    with timing():
        return sum(i * i for i in range(10_000))
```

---

## ⚖️ Tradeoff Summary

### Decorators

* Apply behavior at *function definition*
* Reusable across many call sites
* Invisible at call site → cleaner usage, but more implicit
* Harder to parameterize dynamically

### Context Managers

* Apply behavior at *execution scope*
* Explicit and flexible
* Can wrap *any block*, not just functions
* Slightly more verbose, but more precise

---

## 🧠 When to Use What

### Use decorators when:

* Behavior is **intrinsic to the function**
* You want consistency across all calls
* You're building reusable cross-cutting concerns (logging, auth, caching)

### Use context managers when:

* Behavior is **situational or scoped**
* You need dynamic control (only some calls, some branches)
* You're managing resources or temporary state

---

## 🔥 Subtle but Critical Difference

* **Decorators answer:**

  > "What is this function?"

* **Context managers answer:**

  > "What is happening during this execution?"

That’s *identity vs episode*.

---

## ⚠️ Common Misunderstanding (Decorator Trap)

Forgetting that decorators run **once at definition time**, not per call:

```python
def debug(func):
    print("Decorating", func.__name__)  # runs once
    return func
```

This surprises people expecting runtime behavior.

---

## 💡 Elite Secret

You can **combine both** for powerful control:

```python
from contextlib import contextmanager
from functools import wraps

@contextmanager
def timing():
    start = time.perf_counter()
    yield
    print(time.perf_counter() - start)

def timed(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        with timing():
            return func(*args, **kwargs)
    return wrapper
```

Now you get:

* reusable decorator
* internally scoped control

This pattern shows up in serious frameworks.

---

## 🔄 Breakthrough Reframe

Stop thinking:

> "Do I wrap this function?"

Start thinking:

> "Is this behavior part of the function’s identity, or just this execution?"

That distinction prevents:

* over-decorating everything
* hiding critical runtime behavior
* rigid designs