---

layout: post
title: "Advanced Python Design Patterns & Internals"
date: 2026-04-17 07:43:36 +0000
tags: [python, design-patterns, object-model, concurrency, memory, advanced]
categories: python
---

## Overview

A curated collection of advanced Python concepts, focusing on tradeoffs, mental models, and practical usage patterns across object design, memory, and concurrency.

---

# `@staticmethod` vs `@classmethod`

*(namespace utility vs class-aware behavior)*

## `@staticmethod`

```python
class MathUtils:
    @staticmethod
    def add(x, y):
        return x + y
```

* No `self`, no `cls`
* Pure utility function inside a class namespace

## `@classmethod`

```python
class User:
    def __init__(self, name):
        self.name = name

    @classmethod
    def from_email(cls, email):
        name = email.split("@")[0]
        return cls(name)
```

* Receives `cls`
* Supports alternative constructors
* Inheritance-aware

### Insight

> `staticmethod` = namespacing
> `classmethod` = class-aware behavior

---

# Decorators vs Descriptors

*(function transformation vs attribute control)*

## Decorators

```python
def log_calls(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper
```

* Modify functions at **definition time**

## Descriptors

```python
class UpperCase:
    def __set_name__(self, owner, name):
        self.name = "_" + name

    def __get__(self, obj, objtype=None):
        return getattr(obj, self.name)

    def __set__(self, obj, value):
        setattr(obj, self.name, value.upper())
```

* Control attribute access at **runtime**

### Insight

> Decorators wrap execution
> Descriptors redefine access

---

# `__getattr__` vs `__getattribute__`

*(fallback vs full control)*

## `__getattr__`

* Called only when attribute is missing
* Safe fallback mechanism

## `__getattribute__`

* Called on **every access**
* Full control, but dangerous

### Insight

> `__getattr__` = fallback
> `__getattribute__` = total control

---

# Metaclasses vs Class Decorators

*(creation vs transformation)*

## Class Decorator

* Modifies class **after creation**

## Metaclass

* Controls class **during creation**

### Insight

> Decorators modify classes
> Metaclasses define how classes exist

---

# `typing.Protocol` vs Inheritance

*(structural vs nominal typing)*

## Inheritance (ABC)

* Explicit hierarchy
* Runtime enforcement

## Protocol

* Behavior-based typing
* No inheritance required

### Insight

> Inheritance: “you are this”
> Protocol: “you behave like this”

---

# `dataclasses` vs `attrs`

*(simple vs powerful models)*

## `dataclasses`

* Standard library
* Simple and readable

## `attrs`

* Rich validation
* Field-level customization

### Insight

> dataclasses reduce boilerplate
> attrs builds full data models

---

# `__dict__` vs `__slots__`

*(flexibility vs memory efficiency)*

## `__dict__`

* Dynamic attributes
* Flexible

## `__slots__`

* Fixed layout
* Memory efficient

### Insight

> `__dict__` = flexible objects
> `__slots__` = structured objects

---

# Reference Counting vs Garbage Collection

*(deterministic vs cyclic cleanup)*

## Reference Counting

* Immediate cleanup
* Cannot handle cycles

## Garbage Collection

* Cleans cycles
* Non-deterministic

### Insight

> Python = reference counting first, GC second

---

# Shallow Copy vs Deep Copy

*(structure vs full duplication)*

## Shallow Copy

* Copies outer container
* Shares inner objects

## Deep Copy

* Fully independent copy

### Insight

> Shallow = shared state
> Deep = independent state

---

# Inheritance vs Composition

*(identity vs capability)*

## Inheritance

* “is-a” relationship
* Tight coupling

## Composition

* “has-a” relationship
* Flexible design

### Insight

> Inheritance defines identity
> Composition defines capability

---

# Strategy Pattern vs `if/elif`

*(modular behavior vs inline logic)*

## `if/elif`

* Simple, but scales poorly

## Strategy Pattern

* Encapsulated behaviors
* Easily extensible

### Insight

> Strategy distributes decisions into behavior units

---

# Factory Pattern vs Direct Instantiation

*(decoupled creation vs explicit construction)*

## Direct Instantiation

* Simple
* Tightly coupled

## Factory

* Centralized creation logic
* Extensible

### Insight

> Factories decouple code from concrete types

---

# `threading` vs `asyncio`

*(preemptive vs cooperative concurrency)*

## `threading`

* OS-managed
* Good for blocking I/O

## `asyncio`

* Event loop
* Scales for many tasks

### Insight

> Threads = multiple workers
> Async = one worker juggling tasks

---

# `@property` vs Descriptor Protocol

*(localized vs reusable attribute logic)*

## `@property`

* Simple
* Class-specific

## Descriptor

* Reusable
* Framework-level power

### Insight

> `@property` modifies access
> Descriptors define attributes

---

## Final Thoughts

* Prefer **clarity over cleverness**
* Prefer **composition over inheritance**
* Use **abstractions that match intent**
* Understand Python’s **object model deeply**

> Great Python code reads like a story — and scales

<!-- gh-pages-taxonomy-links:start -->
Categories: [python](/categories/python/)
Tags: [python](/tags/python/), [design-patterns](/tags/design-patterns/), [object-model](/tags/object-model/), [concurrency](/tags/concurrency/), [memory](/tags/memory/), [advanced](/tags/advanced/)
<!-- gh-pages-taxonomy-links:end -->
