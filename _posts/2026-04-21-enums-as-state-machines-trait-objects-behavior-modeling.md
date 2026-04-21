---

layout: post
title: "Enums as State Machines + Trait Objects (Behavior Modeling)"
date: 2026-04-21 00:00:00 +0000
tags: [rust, design-patterns, state-machines, trait-objects, systems-design]
categories: rust
---

## 🦀 Today's Patterns: Enums as State Machines + Trait Objects

This is about one core question:

> How do you represent *different behaviors* in Rust?

---

## 🎭 Pattern 1: Enums as State Machines

### 💡 Idea

Use `enum` to represent all possible states *explicitly*, and `match` to define behavior.

```rust
enum Connection {
    Disconnected,
    Connected,
    Authenticated,
}

impl Connection {
    fn send(&self, msg: &str) {
        match self {
            Connection::Authenticated => println!("Sending: {}", msg),
            _ => println!("Cannot send, not authenticated"),
        }
    }
}
```

---

### 🌍 Real-World Context

* Protocol handling (HTTP, TCP states)
* Parsers (token → AST transitions)
* Game state machines
* `Option`, `Result` (the most famous enums!)

---

### ⚠️ Common Misunderstanding

> "Enums don’t scale well"

They *do* — until:

* States become too many
* Behavior becomes too complex

---

### 🧠 Elite Secret

Enums + `match` give you **exhaustiveness checking**:

```rust
match conn {
    Connection::Disconnected => {}
    Connection::Connected => {}
    Connection::Authenticated => {}
}
```

👉 Add a new state → compiler forces updates everywhere

---

### 🔁 Breakthrough Reframe

**Enums turn possibilities into guarantees.**

---

## 🧩 Pattern 2: Trait Objects (`dyn Trait`)

### 💡 Idea

Model behavior via shared interfaces, hiding concrete types.

```rust
trait Handler {
    fn handle(&self, input: &str);
}

struct PrintHandler;

impl Handler for PrintHandler {
    fn handle(&self, input: &str) {
        println!("Print: {}", input);
    }
}

fn run(handler: Box<dyn Handler>) {
    handler.handle("hello");
}
```

---

### 🌍 Real-World Context

* Plugin systems
* Middleware chains (web frameworks)
* GUI components
* Strategy pattern equivalents

Rust uses this when:

* Behavior must be extensible and open-ended

---

### ⚠️ Common Misunderstanding

> "Trait objects are just like inheritance"

Not quite:

* No shared data
* No subclass hierarchy
* Only behavior abstraction

---

### 🧠 Elite Secret

Trait objects enable **open sets**:

👉 You can add new implementations *without modifying existing code*

This is impossible with enums.

---

### 🔁 Breakthrough Reframe

**Trait objects turn closed worlds into open systems.**

---

## ⚖️ Enums vs Trait Objects

### 🟢 Pros

#### Enums

* Exhaustive (compiler-checked)
* No runtime dispatch
* Great for state machines
* Easy to reason about

#### Trait Objects

* Extensible (open for new types)
* Decoupled design
* Useful for plugins and abstractions
* Reduces compile-time coupling

---

### 🔴 Cons

#### Enums

* Closed set (must modify enum to add new variant)
* Can grow large and unwieldy
* Behavior centralized (big `match` blocks)

#### Trait Objects

* Runtime dispatch cost
* No exhaustiveness checking
* Harder to trace behavior
* Requires indirection (`Box`, `Arc`)

---

## 🎯 When to Use Enums

* Known, finite states
* Protocol/state machines
* Performance-critical logic
* You want compiler guarantees

---

## 🎭 When to Use Trait Objects

* Unknown or growing set of behaviors
* Plugin systems
* Decoupled architectures
* Runtime flexibility

---

## 🤝 When to Combine Them (🔥 Powerful)

A common hybrid:

```rust
enum Event {
    Click(Box<dyn Handler>),
    KeyPress(Box<dyn Handler>),
}
```

👉 Enum defines *structure*

👉 Trait object defines *behavior*

---

## ⚔️ When It’s Overkill

* Trait objects for simple branching logic
* Enums with massive variant-specific logic (better split types)
* Premature abstraction

---

*Simple rule of thumb:*

* **Closed world? → Enum**
* **Open world? → Trait objects**