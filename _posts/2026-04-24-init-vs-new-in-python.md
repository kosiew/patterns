---

layout: post
title: "**init** vs **new** in Python"
date: 2026-04-24 00:00:00 +0000
tags: [python, oop, internals, design-patterns]
categories: python
---

## 🐍 Topic Pair: `__init__` vs `__new__`

---

## ⚙️ Technique A — `__init__` (Initialize an Existing Instance)

```python
class User:
    def __init__(self, name):
        self.name = name
```

---

## 🧠 Technique B — `__new__` (Control Instance Creation)

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        print("Init runs every call")
```

---

## ⚖️ Tradeoff Summary

### `__init__`

* Runs *after* the object is created
* Used for initialization only
* Clean, predictable, almost always sufficient
* Cannot change what object gets returned

### `__new__`

* Runs *before* the object exists
* Controls *whether and what* gets created
* Enables patterns like singletons, immutables, interning
* Easy to misuse → subtle bugs

---

## 🧠 When to Use What

### Use `__init__` when:

* You're just setting up instance state
* Object identity doesn’t need control
* You want maintainable, idiomatic code

### Use `__new__` when:

* You need to **control instantiation**
* You're subclassing immutable types (`str`, `tuple`, `int`)
* You want object reuse / caching (interning)
* You're implementing patterns like singleton or flyweight

---

## 🔥 Subtle but Critical Difference

* `__init__` answers:

  > "How do I configure this object?"

* `__new__` answers:

  > "Should this object even exist—and if so, which one?"

That’s not initialization—that’s **identity control**.

---

## ⚠️ Common Misunderstanding (`__new__` trap)

Forgetting that `__init__` still runs even if you return an existing instance:

```python
a = Singleton()
b = Singleton()
```

`__init__` runs **twice**, even though `a is b`.

This leads to:

* accidental re-initialization
* state corruption

---

## 💡 Elite Secret

Use `__new__` for immutable subclassing:

```python
class UpperStr(str):
    def __new__(cls, value):
        return super().__new__(cls, value.upper())
```

```python
s = UpperStr("hello")
print(s)  # "HELLO"
```

Why this works:

* `str` is immutable → must be constructed in `__new__`
* `__init__` would be too late

---

## 🔁 Breakthrough Reframe

Stop thinking:

> "`__init__` constructs objects."

It doesn’t.

> "`__new__` constructs, `__init__` configures."

Once you internalize this:

* immutable types make sense
* object caching patterns become obvious
* Python’s data model becomes *predictable*