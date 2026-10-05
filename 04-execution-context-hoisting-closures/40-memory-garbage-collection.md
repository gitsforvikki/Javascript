# Lesson 40 — Memory and Garbage Collection

JavaScript automatically manages memory for us, but developers still need to understand **references, reachability, garbage collection, and memory leaks**.

This lesson focuses on the conceptual model needed for application development and interviews.

---

## 1. Memory Lifecycle

Programs generally go through three memory-related steps:

```text
Allocate memory
      ↓
Use memory
      ↓
Release memory when no longer needed
```

In JavaScript, memory release is primarily automatic.

The JavaScript engine uses a **garbage collector (GC)** to reclaim memory that is no longer reachable.

---

## 2. Values and References

Primitive values behave as values:

```js
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
console.log(b); // 20
```

Objects behave through references:

```js
const user1 = {
  name: "Vikash",
};

const user2 = user1;

user2.name = "Kumar";

console.log(user1.name);
// Kumar
```

Conceptually:

```text
user1 ──┐
        ├──→ Object
user2 ──┘
```

The object remains reachable through both variables.

---

## 3. Reachability

The central idea behind garbage collection is **reachability**.

An object is generally considered reachable when the running program can still reach it through references starting from live roots.

Simplified roots can include things such as:

- active execution contexts
- global references
- currently reachable closures
- runtime-held references

Example:

```js
let user = {
  name: "Vikash",
};
```

```text
user ──→ Object
```

The object is reachable.

Now:

```js
user = null;
```

If nothing else references that object:

```text
user ──→ null

Object
  ↑
no reachable reference
```

The object becomes eligible for garbage collection.

Important:

> Eligible for garbage collection does not mean JavaScript immediately frees it at that exact line.

The engine decides when garbage collection runs.

---

## 4. Multiple References

```js
let user = {
  name: "Vikash",
};

let admin = user;

user = null;
```

Is the object collectible?

No.

```text
user ──→ null

admin ──→ Object
```

The object is still reachable through `admin`.

Only when no reachable reference remains can it become eligible for collection.

---

## 5. Circular References

Consider:

```js
let user = {};
let profile = {};

user.profile = profile;
profile.user = user;
```

Now:

```text
user object ─────→ profile object
     ↑                  │
     └──────────────────┘
```

Older simplistic explanations sometimes suggest that circular references automatically cause memory leaks.

Modern garbage collectors are generally based on **reachability**, so a cycle can still be collected if the entire cycle becomes unreachable from live roots.

Example:

```js
user = null;
profile = null;
```

If nothing else reaches those objects, the cycle can become garbage.

---

## 6. Mark-and-Sweep — Conceptual Model

A common conceptual garbage-collection strategy is **mark-and-sweep**.

Simplified:

```text
Start from roots
      ↓
Mark reachable values
      ↓
Follow their references
      ↓
Mark everything reachable
      ↓
Unmarked unreachable memory
      ↓
can be reclaimed
```

Example:

```text
Root
 │
 └──→ A ──→ B

C ──→ D
↑
no path from root
```

A and B are reachable.

C and D are unreachable and can be reclaimed.

Modern engines use much more sophisticated garbage-collection strategies and optimizations, but reachability is the important developer mental model.

---

## 7. Stack vs Heap — Be Careful

You will often hear:

> primitives are stored on the stack and objects on the heap.

This is a useful beginner simplification, but it is not a reliable JavaScript language guarantee.

Engines can optimize storage internally.

A safer conceptual model is:

```text
Call Stack
→ tracks active execution

Managed Memory / Heap
→ engine-managed storage for objects and other runtime data
```

For JavaScript programming, focus more on:

- value semantics
- reference identity
- reachability
- lifetime

than on assuming an exact physical memory location.

---

## 8. What Is a Memory Leak?

A memory leak occurs when memory remains reachable and therefore cannot be reclaimed even though the application no longer needs that data.

Conceptually:

```text
Application no longer needs object
          ↓
but some reference still exists
          ↓
object remains reachable
          ↓
GC cannot reclaim it
          ↓
memory usage can grow
```

---

## 9. Common Memory-Leak Sources

### Unremoved Event Listeners

```js
const button = document.querySelector(
  "#button"
);

function handleClick() {
  console.log("clicked");
}

button.addEventListener(
  "click",
  handleClick
);
```

If application code repeatedly creates listeners or retains removed DOM structures through references without proper cleanup, memory can remain reachable unnecessarily.

### Timers

```js
const timer = setInterval(() => {
  // repeated work
}, 1000);
```

If no longer needed:

```js
clearInterval(timer);
```

### Large Caches

```js
const cache = new Map();

function save(key, value) {
  cache.set(key, value);
}
```

If entries are continually added and never removed, the cache itself intentionally keeps those values reachable.

### Closures

Closures are not inherently memory leaks.

But a long-lived closure can retain access to data from its lexical environment.

```js
function createHandler() {
  const largeData = new Array(
    1_000_000
  ).fill("data");

  return function handler() {
    console.log(largeData.length);
  };
}

const handler = createHandler();
```

As long as `handler` remains reachable and needs `largeData`, that data must remain available.

That is correct behavior, not automatically a leak.

It becomes problematic only when the program unintentionally retains data it no longer needs.

---

## 10. Garbage Collection and Closures

This connects directly to Lesson 41.

```js
function createCounter() {
  let count = 0;

  return () => ++count;
}

const counter = createCounter();
```

The `createCounter` call has finished, but the returned function still needs the `count` binding.

Therefore the required lexical environment remains reachable.

```text
counter
  ↓
returned function
  ↓
closure
  ↓
count binding
```

Garbage collection cannot remove data that is still reachable and needed.

---

## 11. React Connection

Effects commonly create external subscriptions or timers.

```jsx
useEffect(() => {
  const timer = setInterval(() => {
    console.log("running");
  }, 1000);

  return () => {
    clearInterval(timer);
  };
}, []);
```

Cleanup matters because the external resource may otherwise continue running or retain references longer than intended.

Other examples include:

- event listeners
- subscriptions
- observers
- timers
- some ongoing asynchronous resources

Understanding memory helps explain **why cleanup exists**, not just how to memorize `useEffect` syntax.

---

## 12. Can We Force Garbage Collection?

Normal application JavaScript should not depend on manually forcing garbage collection.

The engine decides:

- when GC runs
- which strategy it uses
- how memory is organized internally

Your responsibility is primarily to stop retaining references/resources that are no longer needed.

---

## Interview Perspective

**What is garbage collection?**

Garbage collection is automatic memory management that identifies memory no longer reachable by the running program and makes it available for reclamation.

**What is a memory leak in JavaScript?**

It occurs when data remains reachable through references even though the application no longer needs it, preventing garbage collection.

**Do circular references always cause leaks?**

No. Modern reachability-based garbage collection can collect an unreachable cycle.

**Are closures memory leaks?**

No. Closures intentionally retain access to required lexical bindings. They become a concern only when they unintentionally keep unnecessary data reachable.

---

## Key Takeaways

- JavaScript uses automatic memory management.
- Reachability is the key mental model for garbage collection.
- Losing one reference does not make an object collectible if another reachable reference remains.
- Circular references are not automatically leaks.
- Memory leaks usually involve unwanted retained references/resources.
- Event listeners, timers, caches, and closures can retain memory.
- Closures intentionally preserve required lexical data.
- Do not rely on exact “primitive stack/object heap” rules as language guarantees.
