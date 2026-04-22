---

layout: post
title: "TypeScript Unions: Loose vs Discriminated"
date: 2026-04-22 00:00:00 +0000
tags: [typescript, type-systems, unions, state-machines, programming]
categories: typescript
---

## Technique A — Loose Union Types (Ambiguous Shape)

```ts
type Result =
  | { data: string }
  | { error: Error };

function handle(result: Result) {
  if ('data' in result) {
    console.log(result.data.toUpperCase());
  } else {
    console.error(result.error.message);
  }
}
```

---

## Technique B — Discriminated Unions (Explicit State Machine)

```ts
type Result =
  | { status: 'ok'; data: string }
  | { status: 'error'; error: Error };

function handle(result: Result) {
  switch (result.status) {
    case 'ok':
      console.log(result.data.toUpperCase());
      break;
    case 'error':
      console.error(result.error.message);
      break;
  }
}
```

---

## Tradeoff Summary 🧠

Both model “one of several possible shapes,” but the difference is **how explicit the state is**:

* Loose unions rely on **property existence checks** (`'data' in result`)
* Discriminated unions introduce a **single canonical tag** (`status`) that drives narrowing

---

## Tradeoffs

### Flexibility vs Safety

* Loose unions are quick and flexible, but can become ambiguous as shapes grow
* Discriminated unions enforce **clear, mutually exclusive states**

### Ergonomics vs Exhaustiveness

* Loose unions feel lightweight
* Discriminated unions enable **exhaustive checking** (the compiler helps you not miss cases)

### Local Reasoning vs System-wide Clarity

* Loose unions work fine in small scopes
* Discriminated unions scale better across teams and services

---

## When to Use Which

* Use **loose unions** when:

  * You’re modeling **simple, short-lived data**
  * The shapes are **obviously distinct**
  * You don’t need strict guarantees

* Use **discriminated unions** when:

  * Modeling **state machines** (loading, success, error, etc.)
  * Building **APIs or shared contracts**
  * You want **compiler-enforced exhaustiveness**

* Combine them when:

  * You gradually evolve a loose union into a discriminated one as complexity grows

---

## Common Misunderstanding ⚠️

> “Checking `'property' in obj` is just as safe as discriminated unions.”

Not quite.

This breaks down when shapes overlap:

```ts
type Result =
  | { data: string; error?: undefined }
  | { error: Error; data?: undefined };
```

Now both properties technically exist in both types — your narrowing becomes **fragile and misleading**.

---

## Elite Secret ⚙️

Leverage `never` for exhaustive checking:

```ts
function assertNever(x: never): never {
  throw new Error(`Unexpected value: ${x}`);
}

function handle(result: Result) {
  switch (result.status) {
    case 'ok':
      return result.data;
    case 'error':
      return result.error.message;
    default:
      return assertNever(result); // compiler enforces completeness
  }
}
```

Now if you add a new variant, TypeScript forces you to handle it.

---

## Breakthrough Reframe 💡

Discriminated unions are not just “better unions.”

They are **compile-time state machines**.

Instead of thinking:

> “This object might look like X or Y”

Think:

> “This system is in exactly one valid state at a time”

That shift is what enables:

* safer async flows
* predictable reducers
* robust API contracts

---

## Real Node.js Context ⚙️

This pattern shines in:

* HTTP responses (`success | error | validation_error`)
* Job processing (`pending | running | failed | completed`)
* Event-driven systems (Kafka, queues, workers)

Loose unions tend to leak bugs at runtime.

Discriminated unions **fail at compile time instead** — much cheaper.