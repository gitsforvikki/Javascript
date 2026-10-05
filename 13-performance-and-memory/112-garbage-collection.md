# Lesson 112 — Garbage Collection

Garbage collection (GC) is the process by which the JavaScript engine automatically reclaims memory that is no longer reachable by the program.

The core idea is:

> Memory is reclaimed based on reachability, not because a variable simply went out of sight.

---

## 1. Allocation, Use, Release

Every program roughly follows:

```text
allocate memory
↓
use values
↓
values become unreachable
↓
garbage collector may reclaim memory
```

JavaScript automates the final step.

---

## 2. Reachability

A value is considered reachable if it can still be accessed from known roots.

Common roots include:

- global objects
- currently executing functions
- active closures
- DOM references held by reachable objects
- runtime-managed references

Example:

```js
let user = {
  name: "Vikash",
};
```

The object is reachable through `user`.

---

## 3. Losing a Reference

```js
let user = {
  name: "Vikash",
};

user = null;
```

If no other reference points to the object, it may become unreachable and eligible for garbage collection.

Important:

> Eligible for collection does not mean collected immediately.

---

## 4. Multiple References

```js
let a = {
  id: 1,
};

let b = a;

a = null;
```

The object is still reachable through `b`.

Only after:

```js
b = null;
```

and no other reachable reference remains can it become collectable.

---

## 5. Mark-and-Sweep Mental Model

A useful conceptual model:

```text
start from roots
↓
mark reachable values
↓
follow references
↓
everything unmarked
is unreachable
↓
memory can be reclaimed
```

Modern engines use more sophisticated techniques, but this mental model is excellent for reasoning.

---

## 6. Circular References Are Not Automatically Leaks

```js
let a = {};
let b = {};

a.b = b;
b.a = a;

a = null;
b = null;
```

The objects reference each other, but if no reachable root points to them, the whole cycle can still be collected.

Modern garbage collectors are not simple reference counters.

---

## 7. Scope and Reachability

```js
function createUser() {
  const user = {
    name: "Vikash",
  };
}

createUser();
```

After the function returns, `user` may become unreachable if nothing else references it.

But closures can intentionally keep data reachable.

---

## 8. Closures and Memory

```js
function createCounter() {
  let count = 0;

  return function () {
    return ++count;
  };
}

const counter =
  createCounter();
```

The outer function has returned, but `count` remains reachable through the returned closure.

This is not a leak by itself.

---

## 9. Closure Leak Pattern

A closure can retain more memory than intended.

Example conceptually:

```js
function createHandler() {
  const hugeData =
    loadHugeData();

  return function () {
    console.log(
      hugeData.length
    );
  };
}
```

As long as the returned function remains reachable, `hugeData` remains reachable too.

---

## 10. Generational Garbage Collection

Many modern engines optimize based on the observation:

> Most newly allocated objects die young.

Conceptually:

```text
young generation
→ checked frequently

surviving objects
→ promoted

older generation
→ checked differently
```

You do not need engine-specific details for everyday coding, but this explains why short-lived allocations can be cheap.

---

## 11. GC Pauses

Garbage collection requires work.

Modern engines minimize pauses using techniques such as:

- incremental collection
- generational collection
- concurrent work
- compaction

But heavy allocation can still contribute to performance pressure.

---

## 12. Allocation Churn

This code creates many temporary objects:

```js
for (
  let i = 0;
  i < 1_000_000;
  i++
) {
  const value = {
    index: i,
  };
}
```

Even if objects die quickly, creating huge numbers of temporary objects increases GC work.

---

## 13. Stack vs Heap Caution

You may hear:

```text
primitives → stack
objects → heap
```

This is an implementation simplification, not a JavaScript language guarantee.

For GC reasoning, focus on:

- reachability
- references
- lifetime

not guessed memory locations.

---

## 14. WeakMap and WeakSet Connection

Weak collections do not keep otherwise unreachable object keys/values alive.

Example:

```js
const metadata =
  new WeakMap();
```

This is useful when metadata lifetime should follow object lifetime.

---

## 15. DOM and GC

A removed DOM node can still remain in memory if JavaScript holds a reference.

```js
const node =
  document.querySelector(
    "#panel"
  );

node.remove();
```

If `node` is still reachable, it may remain in memory.

DOM removal and JavaScript reachability are separate.

---

## 16. React Connection

React components may create:

- closures
- timers
- event listeners
- subscriptions
- cached data

Unmounting a component does not magically clean up everything you created manually.

Cleanup determines whether objects become unreachable.

---

## 17. Memory Profiling Mental Model

When memory usage grows unexpectedly, ask:

```text
What object is growing?
↓
What keeps it reachable?
↓
What root/reference chain leads to it?
```

This is more useful than asking:

> Why didn't GC run?

---

## 18. Heap Snapshot Concept

Browser DevTools can show:

- retained objects
- detached DOM nodes
- retaining paths
- object counts
- memory growth

A retaining path explains why an object is still reachable.

---

## Interview Questions

### What is garbage collection?

Automatic memory reclamation for values that are no longer reachable.

### Does setting a variable to null immediately free memory?

No.

### Can circular references be garbage collected?

Yes, if the cycle is unreachable from GC roots.

### Are closures memory leaks?

No. They only become problematic when they unintentionally retain unnecessary data.

---

## Key Takeaways

- Garbage collection is based on reachability.
- Unreachable does not mean immediately collected.
- Multiple references keep objects alive.
- Circular references are collectable when unreachable.
- Closures can intentionally or accidentally retain memory.
- WeakMap/WeakSet support lifetime-sensitive associations.
- Heavy allocation increases GC pressure.
- To debug memory, find what is retaining the object.
