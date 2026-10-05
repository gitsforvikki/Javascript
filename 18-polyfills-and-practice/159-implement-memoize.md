# Lesson 159 — Implement Memoize

Memoization caches a function's result so repeated calls with the same inputs can reuse previous work.

The core rule is:

> same cache key → return previous result

This interview problem tests:

- closures
- Map / WeakMap
- cache-key design
- referential equality
- multiple arguments
- `this`
- memory tradeoffs

---

## 1. Basic Example

```js
function slowSquare(
  number
) {
  console.log(
    "calculating"
  );

  return number *
    number;
}
```

Without memoization:

```js
slowSquare(5);
slowSquare(5);
```

calculation runs twice.

---

## 2. Basic Memoize

```js
function memoize(
  fn
) {
  const cache =
    new Map();

  return function (
    arg
  ) {
    if (
      cache.has(arg)
    ) {
      return cache.get(
        arg
      );
    }

    const result =
      fn.call(
        this,
        arg
      );

    cache.set(
      arg,
      result
    );

    return result;
  };
}
```

---

## 3. Closure Connection

The returned function remembers:

```js
cache
```

through closure.

```text
memoize()
↓
cache created once
↓
returned wrapper keeps access
↓
future calls reuse entries
```

---

## 4. Why Map?

Map supports:

- primitive keys
- object keys
- clear `has/get/set` semantics
- reference identity for object keys

---

## 5. Object Identity Matters

```js
const memoized =
  memoize(
    (user) =>
      user.name
  );

memoized({
  name: "Vikash",
});

memoized({
  name: "Vikash",
});
```

These are two different object references.

So they produce two cache entries.

---

## 6. Multiple Arguments Problem

Simple memoize only handles one argument.

Naive approach:

```js
const key =
  JSON.stringify(
    args
  );
```

Example:

```js
function memoize(
  fn
) {
  const cache =
    new Map();

  return function (
    ...args
  ) {
    const key =
      JSON.stringify(
        args
      );

    if (
      cache.has(key)
    ) {
      return cache.get(
        key
      );
    }

    const result =
      fn.apply(
        this,
        args
      );

    cache.set(
      key,
      result
    );

    return result;
  };
}
```

Good for learning, but not universally safe.

---

## 7. JSON.stringify Limitations

Problems:

- circular references
- functions
- Symbols
- property ordering assumptions
- special values
- expensive serialization

So production memoization needs a deliberate cache-key strategy.

---

## 8. Nested Map Strategy

A better identity-preserving strategy for multiple arguments:

```js
function memoize(
  fn
) {
  const root =
    new Map();

  return function (
    ...args
  ) {
    let node =
      root;

    for (
      let i = 0;
      i < args.length;
      i++
    ) {
      const arg =
        args[i];

      if (
        !node.has(arg)
      ) {
        node.set(
          arg,
          new Map()
        );
      }

      node =
        node.get(arg);
    }

    const RESULT =
      Symbol.for(
        "memoized-result"
      );

    if (
      node.has(RESULT)
    ) {
      return node.get(
        RESULT
      );
    }

    const result =
      fn.apply(
        this,
        args
      );

    node.set(
      RESULT,
      result
    );

    return result;
  };
}
```

This avoids string serialization, though it can grow indefinitely.

---

## 9. Better Marker Without Global Symbol

Create marker once outside calls:

```js
function memoize(
  fn
) {
  const root =
    new Map();

  const RESULT =
    Symbol(
      "result"
    );

  return function (
    ...args
  ) {
    // same traversal
  };
}
```

A local unique Symbol is safer than a shared registry Symbol for internal bookkeeping.

---

## 10. Preserve this

Use:

```js
fn.apply(
  this,
  args
);
```

not:

```js
fn(...args);
```

if the original function may depend on dynamic `this`.

---

## 11. But this Can Affect Cache Correctness

Example:

```js
function getValue(
  x
) {
  return (
    this.base +
    x
  );
}
```

If cache key uses only `x`, then calls with different `this` objects can incorrectly share results.

So the cache key may need to include:

```text
this
+
arguments
```

---

## 12. Memoize Including this

Conceptually:

```js
function memoize(
  fn
) {
  const root =
    new Map();

  return function (
    ...args
  ) {
    const allKeys =
      [
        this,
        ...args,
      ];

    // traverse nested Maps
  };
}
```

Whether `this` belongs in the cache key depends on the function's semantics.

---

## 13. WeakMap for Object Keys

If the first key is always an object:

```js
const cache =
  new WeakMap();
```

can help avoid keeping otherwise unreachable keys alive.

---

## 14. Cache Growth Problem

Memoization can leak memory if:

```text
new unique input
↓
new cache entry
↓
never evicted
```

Long-lived memoizers may need:

- max size
- TTL
- LRU
- manual clear
- WeakMap where appropriate

---

## 15. Cache Invalidation

The hardest part of caching is often:

```text
When is cached data no longer valid?
```

Strategies:

- dependencies in key
- versioned key
- time-based expiry
- explicit invalidation

---

## 16. Pure Functions Fit Best

Memoization is easiest when:

```text
same inputs
→ same output
```

Avoid memoizing functions that depend on hidden mutable state unless that state is represented in the key.

---

## 17. Async Memoization

You can memoize Promises too.

```js
function memoizeAsync(
  fn
) {
  const cache =
    new Map();

  return function (
    key
  ) {
    if (
      cache.has(key)
    ) {
      return cache.get(
        key
      );
    }

    const promise =
      Promise.resolve(
        fn.call(
          this,
          key
        )
      );

    cache.set(
      key,
      promise
    );

    return promise;
  };
}
```

Now concurrent calls for the same key can share one in-flight Promise.

---

## 18. Rejected Promise Problem

If the cached Promise rejects, you may not want to cache that failure forever.

Better:

```js
const promise =
  Promise.resolve(
    fn(key)
  ).catch(
    (error) => {
      cache.delete(
        key
      );

      throw error;
    }
  );
```

---

## 19. React Connection

`useMemo` is not the same as a global memoize utility.

```text
memoize(fn)
→ cache by function inputs

useMemo
→ cache one computed value
  between React renders
  based on dependencies
```

---

## 20. Common Wrong Implementation

```js
function memoize(
  fn
) {
  let lastArg;
  let lastResult;

  return function (
    arg
  ) {
    if (
      arg === lastArg
    ) {
      return lastResult;
    }

    lastArg =
      arg;

    lastResult =
      fn(arg);

    return lastResult;
  };
}
```

This only remembers one previous call.

That can be valid as a tiny optimization, but it is not a general memoization cache.

---

## 21. Complexity

Lookup:

```text
Map-based single key
→ average O(1)
```

Nested multiple keys:

```text
O(k)
```

where `k` is argument count.

Memory depends on unique key count.

---

## Interview Explanation

A strong answer:

> Memoization stores results in closure. For a single argument, a Map is enough. For multiple arguments, I either need a serializer with known limitations or nested Maps so identity is preserved. I also preserve `this` if needed and consider whether `this` itself belongs in the cache key. A production version also needs an eviction/invalidation strategy.

---

## Key Takeaways

- Memoization caches function results.
- Closure stores the cache.
- Map preserves primitive/object identity.
- Multiple arguments require a deliberate key strategy.
- JSON.stringify keys are limited.
- `this` may affect correctness.
- Async memoization can cache in-flight Promises.
- Cache growth and invalidation matter.
