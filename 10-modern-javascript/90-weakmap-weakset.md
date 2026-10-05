# Lesson 90 — WeakMap and WeakSet

`WeakMap` and `WeakSet` are specialized collections designed to hold **weak references** to objects.

Their purpose is mainly connected to garbage collection.

The most important mental model:

> A weak collection does not keep an otherwise unreachable object alive just because that object is stored in the collection.

---

## 1. Why "Weak"?

Normal Map:

```js
const map =
  new Map();

let user = {
  id: 1,
};

map.set(
  user,
  "metadata"
);

user = null;
```

The Map still strongly references the original object key.

So the object remains reachable through the Map.

---

## 2. WeakMap

```js
const metadata =
  new WeakMap();

let user = {
  id: 1,
};

metadata.set(
  user,
  {
    lastSeen:
      Date.now(),
  }
);
```

If all other references to `user` disappear:

```js
user = null;
```

the object may become eligible for garbage collection.

The WeakMap does not force it to stay alive.

---

## 3. WeakMap Keys

WeakMap keys must be appropriate garbage-collectable identity values, normally objects or non-registered symbols in modern JavaScript.

For everyday usage, remember:

> WeakMap keys are typically objects.

This is invalid in common use:

```js
weakMap.set(
  "name",
  "Vikash"
);
```

because primitive strings are not valid weak keys.

---

## 4. WeakMap Methods

Core methods:

```js
weakMap.set(
  object,
  value
);

weakMap.get(
  object
);

weakMap.has(
  object
);

weakMap.delete(
  object
);
```

There is no:

```js
weakMap.size
weakMap.keys()
weakMap.values()
weakMap.entries()
```

---

## 5. Why WeakMap Is Not Iterable

If JavaScript allowed enumeration, code could observe garbage-collection timing.

But garbage collection is intentionally nondeterministic.

Therefore WeakMap does not provide iteration APIs.

This protects the weak-reference semantics.

---

## 6. WeakMap Use Case — Metadata

```js
const metadata =
  new WeakMap();

function track(element) {
  metadata.set(
    element,
    {
      createdAt:
        Date.now(),
    }
  );
}
```

If a DOM element later becomes unreachable elsewhere, metadata associated with it does not force the element to remain alive.

This is a classic weak-reference use case.

---

## 7. WeakMap Use Case — Private Data

Before `#private` fields, WeakMap was often used for instance-private data.

```js
const privateData =
  new WeakMap();

class User {
  constructor(name) {
    privateData.set(
      this,
      {
        name,
      }
    );
  }

  getName() {
    return privateData
      .get(this)
      .name;
  }
}
```

The metadata is keyed by the instance.

Today, `#privateField` is often clearer when class-private state is the goal.

---

## 8. WeakSet

```js
const visited =
  new WeakSet();
```

Add an object:

```js
const user = {
  id: 1,
};

visited.add(user);
```

Check:

```js
visited.has(user);
```

Delete:

```js
visited.delete(user);
```

---

## 9. WeakSet Values

WeakSet stores garbage-collectable identity values, typically objects.

It does not store arbitrary primitive values like normal Set.

---

## 10. WeakSet Is Not Iterable

There is no:

```js
weakSet.size
weakSet.values()
weakSet.entries()
```

Same reason:

Garbage collection must remain unobservable and nondeterministic.

---

## 11. WeakSet Use Case — Marking Objects

```js
const processed =
  new WeakSet();

function process(user) {
  if (
    processed.has(user)
  ) {
    return;
  }

  processed.add(user);

  // process once
}
```

The marker does not force unused objects to stay alive.

---

## 12. Map vs WeakMap

```text
Map
→ strong references
→ iterable
→ any key type
→ has size

WeakMap
→ weak object-like keys
→ not iterable
→ no size
→ GC-sensitive association
```

---

## 13. Set vs WeakSet

```text
Set
→ strong references
→ any value
→ iterable
→ has size

WeakSet
→ weak object-like values
→ not iterable
→ no size
```

---

## 14. Garbage Collection Is Not Immediate

Do not think:

```js
user = null;
```

means:

```text
object deleted immediately
```

It only means the object may become unreachable.

The JavaScript engine decides when garbage collection happens.

Weak collections do not let you observe exactly when.

---

## 15. Weak Does Not Mean "Memory Leak Impossible"

Weak collections can help avoid retaining objects accidentally.

But poor application architecture can still leak memory through:

- global references
- event listeners
- timers
- caches
- closures

WeakMap/WeakSet solve specific reference-retention problems.

---

## 16. DOM Metadata Example

```js
const elementState =
  new WeakMap();

function init(element) {
  elementState.set(
    element,
    {
      mounted: true,
    }
  );
}
```

This is better than placing metadata on the DOM node when you want external bookkeeping without affecting lifetime.

---

## 17. Common Mistake — Expecting Enumeration

This is invalid:

```js
for (
  const item
  of weakMap
) {
}
```

WeakMap is not iterable.

---

## 18. Common Mistake — Using WeakMap for Cache You Need to Inspect

If you need:

- entry count
- iteration
- eviction policies
- reporting

use `Map`, not `WeakMap`.

WeakMap is for object-associated metadata where object lifetime should control entry lifetime.

---

## 19. React/Application Connection

WeakMap can be useful in library/framework internals for associating metadata with:

- component-related objects
- DOM nodes
- external objects
- cache keys

Most normal React application state should not use WeakMap unless there is a clear object-lifetime reason.

---

## Interview Questions

### Why is WeakMap called weak?

Because its key reference does not prevent an otherwise unreachable key object from being garbage collected.

### Why is WeakMap not iterable?

Enumeration would expose garbage-collection-dependent state.

### WeakMap vs Map?

Use WeakMap for object-associated metadata whose lifetime should follow the key object; use Map when you need normal collection behavior and iteration.

---

## Key Takeaways

- WeakMap and WeakSet are GC-sensitive collections.
- WeakMap keys are typically objects.
- WeakSet stores weakly held object-like values.
- Weak collections are not iterable.
- They do not expose size.
- They do not guarantee immediate garbage collection.
- They are useful for metadata, private associations, and object-lifetime-aware bookkeeping.
