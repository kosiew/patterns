---

layout: post
title: "Manual Type Checks vs singledispatch in Python"
date: 2026-04-22 00:00:00 +0000
tags: [python, design-patterns, software-engineering, polymorphism]
categories: python
---

## ⚙️ Technique A — Manual Type Checks

```python
def render(obj):
    if isinstance(obj, str):
        return obj.upper()
    elif isinstance(obj, list):
        return ", ".join(render(x) for x in obj)
    elif isinstance(obj, dict):
        return {k: render(v) for k, v in obj.items()}
    else:
        raise TypeError(f"Unsupported type: {type(obj)}")
```

---

## 🧠 Technique B — `singledispatch` (Generic Function)

```python
from functools import singledispatch

@singledispatch
def render(obj):
    raise TypeError(f"Unsupported type: {type(obj)}")

@render.register
def _(obj: str):
    return obj.upper()

@render.register
def _(obj: list):
    return ", ".join(render(x) for x in obj)

@render.register
def _(obj: dict):
    return {k: render(v) for k, v in obj.items()}
```

---

## ⚖️ Tradeoff Summary

### Manual branching

* Simple, explicit, easy to follow
* All logic in one place
* Becomes unwieldy as types grow
* Violates open/closed principle (you must edit the function)

### `singledispatch`

* Extensible without modifying original function
* Clean separation per type
* Slight indirection → harder to trace
* Dispatch only on *first argument*

---

## 🧠 When to Use What

### Use manual branching when:

* The number of types is small and stable
* Logic is tightly coupled
* You want straightforward control flow

### Use `singledispatch` when:

* You expect **new types over time**
* You want plugin-like extensibility
* Different modules should register their own behavior
* You're modeling operations over *heterogeneous data*

---

## 🔥 Subtle but Critical Difference

* **Manual branching says:**

  > "This function owns all behavior"

* **`singledispatch` says:**

  > "Behavior can be extended externally"

That’s the difference between:

* **closed function**
* **open system**

---

## ⚠️ Common Misunderstanding (`singledispatch` trap)

Assuming it works like full multiple dispatch:

```python
@render.register
def _(obj: list[str]):  # ❌ ignored at runtime
    ...
```

Type hints don’t affect dispatch—only the **concrete runtime type** does.

---

## 💡 Elite Secret

You can dispatch on abstract base classes:

```python
from collections.abc import Sequence

@render.register
def _(obj: Sequence):
    return [render(x) for x in obj]
```

Now your function supports:

* lists
* tuples
* custom sequence types

...without knowing them explicitly.

---

## 🔄 Breakthrough Reframe

Stop thinking:

> "How do I handle different types?"

Start thinking:

> "Who *owns* the behavior for this type?"

* Manual branching → centralized ownership
* `singledispatch` → distributed ownership

That’s a **system design decision**, not just syntax.