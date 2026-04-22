---

layout: post
title: "Promise.all vs Promise.allSettled"
date: 2026-04-22 00:00:00 +0000
tags: [javascript, typescript, async, concurrency, system-design]
categories: typescript
---

# Promise.all vs Promise.allSettled

## Failure Semantics in Concurrency

## Topic Pair: `Promise.all` vs `Promise.allSettled`

---

## Technique A — `Promise.all` (Fail Fast)

```ts
async function fetchUserBundle(userId: string) {
  const [profile, posts, friends] = await Promise.all([
    getProfile(userId),
    getPosts(userId),
    getFriends(userId),
  ]);

  return { profile, posts, friends };
}
```

---

## Technique B — `Promise.allSettled` (Fail Soft / Observe Everything)

```ts
async function fetchUserBundle(userId: string) {
  const results = await Promise.allSettled([
    getProfile(userId),
    getPosts(userId),
    getFriends(userId),
  ]);

  const [profile, posts, friends] = results.map((r) =>
    r.status === 'fulfilled' ? r.value : null
  );

  return { profile, posts, friends };
}
```

---

## Tradeoff Summary 🧠

This is fundamentally about **failure strategy under concurrency**.

* `Promise.all` is **fail-fast**: one rejection short-circuits everything. Great when **all results are required** for correctness. It preserves a strong invariant: *either everything succeeded or nothing did*.
* `Promise.allSettled` is **fail-soft**: you always get a full report. Ideal when **partial data is acceptable** or when you need **observability into multiple failures** (e.g., aggregating APIs, batch jobs).

### Tradeoffs

* **Correctness vs Resilience:**
  `Promise.all` enforces strict correctness; `allSettled` enables graceful degradation.
* **Simplicity vs Control:**
  `Promise.all` is clean and ergonomic; `allSettled` requires manual result handling.
* **Latency vs Waste:**
  `Promise.all` may reject early, but underlying async ops still run — you just ignore them.
  `allSettled` embraces that reality and lets you inspect everything.

---

## When to Use Which

* Use `Promise.all` when:

  * Missing any piece invalidates the whole result (e.g., transactional workflows)
  * You want **early failure signaling**
  * You rely on **strong invariants downstream**

* Use `Promise.allSettled` when:

  * You’re aggregating **independent data sources**
  * **Partial success** is acceptable (e.g., dashboards, recommendations)
  * You need **error reporting per task**

* Combine them when:

  * You group *critical* vs *optional* tasks separately
  * Example: `Promise.all` for core data, `allSettled` for enrichments

---

## Common Misunderstanding ⚠️

> “`Promise.all` cancels other promises when one fails.”

It doesn’t. JavaScript promises are **not cancellable by default**.

The other operations keep running — you just stop awaiting them.

This becomes dangerous in Node.js when:

* those promises involve I/O (**DB calls, HTTP requests**)
* or **side effects** (**writes, mutations**)

You can end up with **partial writes + thrown error = inconsistent system state**.

---

## Elite Secret ⚙️

Wrap `Promise.allSettled` into a typed helper to recover type safety:

```ts
type Settled<T> =
  | { ok: true; value: T }
  | { ok: false; error: unknown };

async function settle<T>(promises: Promise<T>[]): Promise<Settled<T>[]> {
  const results = await Promise.allSettled(promises);
  return results.map((r) =>
    r.status === 'fulfilled'
      ? { ok: true, value: r.value }
      : { ok: false, error: r.reason }
  );
}
```

Now you get **discriminated unions** instead of stringly-typed status checks — much cleaner and safer at scale.

---

## Breakthrough Reframe 💡

Stop thinking of these as just “promise utilities.”

Think of them as **distributed system primitives in disguise**:

* `Promise.all` = *strong consistency requirement*
* `Promise.allSettled` = *eventual consistency / partial availability*

Once your Node.js service calls multiple external systems, you're no longer just writing async code — you're designing **failure boundaries**.

---

## Reflective Questions

* If one of these calls fails in production, should your endpoint **fail or degrade**?
* What happens if these promises include **writes instead of reads**?
* Where does TypeScript stop helping, and runtime behavior take over?

````



---

## Technique B — AbortController (Cooperative Cancellation)

```ts
async function handler(req: Request) {
  const controller = new AbortController();
  const { signal } = controller;

  const tasks = [
    fetchUser({ signal }),
    fetchOrders({ signal }),
    fetchRecommendations({ signal }),
  ];

  try {
    return await Promise.all(tasks);
  } catch (err) {
    controller.abort(); // actively cancel others
    throw err;
  }
}
````

*(Assumes your `fetch*` functions pass `signal` to `fetch()` or respect it internally.)*

---

## Tradeoff Summary 🧠

This is about **control over in-flight work**.

* Without cancellation:

  * Simpler
  * But wastes resources and risks side effects continuing after failure
* With `AbortController`:

  * You gain **explicit control over concurrency lifecycle**
  * But only works if your async functions are **cooperative**

### Tradeoffs

* **Simplicity vs Control**

  * Fire-and-forget is clean; cancellation introduces plumbing everywhere
* **Correctness vs Reality**

  * Without cancellation, your system *looks* correct but may still execute unwanted work
* **Runtime Cost vs Resource Efficiency**

  * Cancellation adds overhead, but prevents wasted I/O, CPU, and DB load

---

## Where This Really Matters (Node.js Reality)

This becomes critical when your promises involve:

* 🌐 HTTP calls (e.g., `fetch`, axios)
* 🗄️ Database queries
* 📦 Message queues / background jobs

Without cancellation:

* You might **DDOS your own dependencies under failure**
* You can trigger **writes after a request already failed**
* You lose **backpressure control**

---

## Common Misunderstanding ⚠️

> “Using `AbortController` automatically cancels everything.”

Nope. Cancellation in JS is **cooperative, not enforced**.

If your function ignores the signal:

```ts
async function fetchUser() {
  return db.query('SELECT * FROM users'); // ignores AbortSignal
}
```

👉 It will **not stop**, even if you call `abort()`.

---

## Elite Secret ⚙️

Build your APIs to be **cancellation-aware by design**:

```ts
async function fetchUser({ signal }: { signal?: AbortSignal }) {
  const res = await fetch('/user', { signal });
  return res.json();
}
```

Now cancellation becomes **composable across your entire stack**.

Even better: propagate it through service layers:

```ts
function createUserService(signal: AbortSignal) {
  return {
    getUser: () => fetchUser({ signal }),
  };
}
```

You’ve now turned cancellation into a **cross-cutting concern**, like logging or tracing.

---

## Breakthrough Reframe 💡

> Promises don’t represent *tasks*. They represent *results*.

That’s why they aren’t cancellable.

If you need cancellation, you’re no longer just dealing with values —

you’re managing **lifecycles of side effects**.

That’s a different level of system des