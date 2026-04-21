---

layout: post
title: "Mapped Types vs Conditional Types in TypeScript"
date: 2026-04-21 00:00:00 +0000
tags: [typescript, generics, type-system, programming]
categories: typescript
---

## Topic Pair: Mapped Types vs Conditional Types

### Mapped Types (TypeScript)

```ts
// Mapped Type: transform every property in a type
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type User = {
  id: string;
  age: number;
};

type NullableUser = Nullable<User>;
// { id: string | null; age: number | null }
```

### Conditional Types (TypeScript)

```ts
// Conditional Type: branch logic based on type relationships
type ApiResponse<T> = T extends Error
  ? { success: false; error: string }
  : { success: true; data: T };

type Ok = ApiResponse<{ id: string }>;
// { success: true; data: { id: string } }

type Fail = ApiResponse<Error>;
// { success: false; error: string }
```

## Tradeoff Summary

Mapped types are about **systematic transformation** — you take an existing shape and apply a rule uniformly across its properties. They shine in scenarios like DTO transformations, form states, or enforcing consistency (e.g., making everything readonly or optional).

Conditional types, on the other hand, are about **type-level branching logic**. They let you encode decisions and adapt types dynamically, which is incredibly powerful for libraries and reusable abstractions — but can quickly become hard to read and debug.

* **Mapped Types →** predictable, readable, structure-preserving
* **Conditional Types →** flexible, expressive, but mentally heavier

Mapped types win when your transformation is **uniform and structural**.
Conditional types win when your logic depends on **relationships between types**.

Combine them when you need both: e.g., *"map over properties, but conditionally transform each one."*

## Where Each Breaks Down

* Mapped types struggle when logic depends on **value types** ("if this property is a function, do X...")
* Conditional types become unwieldy when nested — readability tanks fast ⚠️
* Both can increase **compile-time complexity** in large codebases

## Common Misunderstanding (Conditional Types)

Many developers forget that conditional types are **distributive over unions**:

```ts
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>;
// string[] | number[]   (NOT (string | number)[])
```

This surprises people — but it’s often exactly what you want.

## Elite Secret 🧠

You can **combine mapped + conditional types** to selectively transform properties:

```ts
type Serialize<T> = {
  [K in keyof T]: T[K] extends Function ? never : T[K];
};
```

This removes all function properties from a type — a pattern used in API serialization layers.

## Breakthrough Reframe

Think of:

* **Mapped Types =** "loops over properties"
* **Conditional Types =** "if statements for types"

Once you see that, you realize you're basically writing a tiny **compile-time program**.