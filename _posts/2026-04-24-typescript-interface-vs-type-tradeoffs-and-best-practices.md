---

layout: post
title: "TypeScript: interface vs type — Tradeoffs and Best Practices"
date: 2026-04-24 00:00:00 +0000
tags: [typescript, type-system, interfaces, software-design]
categories: typescript
---

## Technique A — `interface` (Extensible Object Contracts)

```ts
interface User {
  id: string;
  name: string;
}

interface Admin extends User {
  role: 'admin';
}

function printUser(user: User) {
  console.log(user.name);
}
```

---

## Technique B — `type` (Composable Type System Primitives)

```ts
type User = {
  id: string;
  name: string;
};

type Admin = User & {
  role: 'admin';
};

type ApiResponse<T> =
  | { status: 'ok'; data: T }
  | { status: 'error'; error: string };
```

---

## Tradeoff Summary 🧠

This isn’t just syntax — it’s about **how you evolve and compose types over time**.

* `interface` is optimized for **declaration and extension** especially in OOP-style or public APIs.
* `type` is optimized for **composition and expressiveness** such as unions, intersections, mapped types, and conditionals.

### Tradeoffs

* **Extensibility vs Expressiveness**
  Interfaces can be **merged and extended**.
  Types can express **much more complex relationships**.

* **Stability vs Power**
  Interfaces are predictable and stable for public contracts.
  Types unlock the full power of the TypeScript type system.

* **Tooling ergonomics**
  Interfaces often produce cleaner error messages.
  Complex `type` compositions can become harder to debug.

---

## When to Use Which

* Use `interface` when:

  * Designing **public APIs or library surfaces**
  * You expect **declaration merging**, such as augmenting Express `Request`
  * Modeling **object shapes that evolve over time**

* Use `type` when:

  * You need **unions, intersections, tuples, or conditionals**
  * You’re building **utility types or abstractions**
  * You want to **compose types like functions**

* Combine them when:

  * Use `interface` for base contracts, `type` for transformations

---

## Common Misunderstanding ⚠️

> “`interface` and `type` are interchangeable.”

They overlap — but only up to a point.

You **cannot** do this with `interface`:

```ts
type ID = string | number; // union ❌ not possible with interface
```

And you **cannot** declaration-merge a `type`:

```ts
type User = { name: string };
// later...
type User = { age: number }; // ❌ error
```

But with `interface`, this *does* merge:

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}

// ✅ becomes { name: string; age: number }
```

---

## Elite Secret ⚙️

Use `interface` to **anchor domain models**, and `type` to **transform them**:

```ts
interface User {
  id: string;
  name: string;
}

type PartialUser = Partial<User>;
type UserWithRole = User & { role: string };
```

This separation keeps:

* your **core models readable**
* your **type logic powerful but isolated**

---

## Breakthrough Reframe 💡

Think of:

* `interface` = **“open contracts”** that can evolve, be extended, and merged
* `type` = **“type functions”** that can compute, transform, and compose

Once you see `type` as a **language for type-level programming**, you stop using it as just a prettier `interface`.

---

## Real Node.js Context ⚙️

This distinction shows up heavily in:

* Express / Fastify typings, where interfaces are used for request augmentation
* Schema transformations, where types are used for DTOs and validation shapes
* Complex backend logic, where types are used for conditional API responses

### Example

```ts
interface Request {
  user?: User;
}
```

Augmented across middleware layers via declaration merging — something only `interface` can do cleanly.