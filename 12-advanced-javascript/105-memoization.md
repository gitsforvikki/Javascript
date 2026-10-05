# Lesson 105 — Memoization

Memoization is an optimization technique that caches the result of a function call so repeated calls with the same inputs can reuse the previous result.

The core idea is:

> Same input → reuse previous output instead of recomputing.

Memoization is useful when:

- computation is expensive
- the function is called repeatedly
- the result is deterministic for the same inputs

---

## 1. Basic Example

Without memoization:

```js
function square(
  number
) {
  console.log(
    "calculating..."
  );

  return number *
    number;
}

square(5);
square(5);
```

The calculation runs twice.

---

## 2. Memoized Version

```js
function memoizedSquare() {
  const cache =
    new Map();

  return function (
    number
  ) {
    if (
      cache.has(number)
    ) {
      return cache.get(
        number
      );
    }

    const result =
      number * number;

    cache.set(
      number,
      result
    );

    return result;
  };
}
```

Usage:

```js
const square =
  memoizedSquare();

square(5);
square(5);
```

The second call uses cached data.

---

## 3. Closure Connection

The returned function remembers:

```js
cache
```

because of a closure.

Mental model:

```text
memoize()
↓
cache created
↓
returned function closes over cache
↓
future calls reuse cache
```

Memoization is a practical closure use case.

---

## 4. Generic Memoize

Simple version:

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

## 5. Why Map Is Useful

`Map` works well because:

- keys can be primitives or objects
- `has()` distinguishes missing entries
- lookup intent is clear
- iteration/size are available if needed

This directly connects to Lesson 88.

---

## 6. Memoization vs Caching

These ideas overlap, but memoization is more specific.

```text
Caching
→ store reusable data

Memoization
→ cache function results
  based on function inputs
```

So memoization is a specialized kind of caching.

---

## 7. Pure Functions Fit Best

Memoization works best with functions where:

```text
same input
→ same output
```

Example:

```js
function double(
  value
) {
  return value * 2;
}
```

This is deterministic.

---

## 8. Non-Deterministic Function Problem

```js
function getRandom() {
  return Math.random();
}
```

Memoizing this changes its intended behavior.

First call stores one value.

Later calls reuse it.

So:

> Do not memoize functions whose result should vary even with the same visible inputs.

---

## 9. Hidden Dependencies Problem

```js
let taxRate =
  0.18;

function total(
  price
) {
  return price *
    (1 + taxRate);
}
```

If you memoize only by `price`, then change:

```js
taxRate = 0.2;
```

the cached result becomes stale.

The function has an external dependency not represented in its key.

---

## 10. Multiple Arguments

Simple approach:

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

This is okay for learning, but not a universal production solution.

---

## 11. Why JSON.stringify Keys Are Limited

Problems include:

- property-order concerns
- circular references
- functions
- Symbols
- special objects
- expensive serialization

So robust memoization needs a key strategy appropriate to the input domain.

---

## 12. Nested Map Strategy

For two primitive/object arguments:

```js
function memoize2(
  fn
) {
  const cache =
    new Map();

  return function (
    a,
    b
  ) {
    let inner =
      cache.get(a);

    if (!inner) {
      inner =
        new Map();

      cache.set(
        a,
        inner
      );
    }

    if (
      inner.has(b)
    ) {
      return inner.get(
        b
      );
    }

    const result =
      fn.call(
        this,
        a,
        b
      );

    inner.set(
      b,
      result
    );

    return result;
  };
}
```

This preserves reference identity without string serialization.

---

## 13. Object Keys and Identity

```js
const memoized =
  memoize(
    (user) =>
      user.name
  );

const a = {
  name: "Vikash",
};

const b = {
  name: "Vikash",
};
```

Even though contents match:

```text
a !== b
```

So a Map-based memoizer treats them as different keys.

This connects to referential equality.

---

## 14. Cache Growth Problem

A memoization cache can grow forever.

```text
new input
↓
new cache entry
↓
more inputs
↓
more entries
```

Long-lived memoizers may cause memory pressure.

Possible solutions:

- limit cache size
- evict old entries
- use LRU strategy
- use WeakMap for object-key-only cases

---

## 15. WeakMap Memoization

If keys are objects:

```js
function memoizeObject(
  fn
) {
  const cache =
    new WeakMap();

  return function (
    object
  ) {
    if (
      cache.has(
        object
      )
    ) {
      return cache.get(
        object
      );
    }

    const result =
      fn(object);

    cache.set(
      object,
      result
    );

    return result;
  };
}
```

The cache does not keep otherwise unreachable objects alive.

---

## 16. Memoization and Expensive Recursion

Classic example:

```js
function fibonacci(
  n
) {
  if (n <= 1) {
    return n;
  }

  return (
    fibonacci(n - 1) +
    fibonacci(n - 2)
  );
}
```

This repeats many subproblems.

Memoization can dramatically reduce repeated work.

---

## 17. Memoized Fibonacci

```js
function fibonacciMemo() {
  const cache =
    new Map([
      [0, 0],
      [1, 1],
    ]);

  function fib(n) {
    if (
      cache.has(n)
    ) {
      return cache.get(n);
    }

    const result =
      fib(n - 1) +
      fib(n - 2);

    cache.set(
      n,
      result
    );

    return result;
  }

  return fib;
}
```

Now each `n` is computed once.

---

## 18. Time Complexity Improvement

Naive recursive Fibonacci:

```text
roughly exponential
```

Memoized version:

```text
roughly linear in n
```

because repeated subproblems are reused.

---

## 19. Memoization Is Not Always Faster

For cheap functions:

```js
function add(
  a,
  b
) {
  return a + b;
}
```

memoization can add more overhead than the computation itself.

Costs include:

- cache lookup
- memory
- key creation
- eviction complexity

Use memoization only when it helps.

---

## 20. React.memo Is Different

`React.memo` memoizes rendering decisions for a component based on props.

Conceptually:

```text
same relevant props
↓
skip unnecessary render
```

It is related to memoization, but not exactly the same as caching a pure function result manually.

---

## 21. useMemo

```js
const filtered =
  useMemo(
    () =>
      expensiveFilter(
        items,
        query
      ),
    [
      items,
      query,
    ]
  );
```

React recomputes only when dependencies change.

This is dependency-based memoization.

---

## 22. useCallback

```js
const handleSave =
  useCallback(
    () => {
      save(id);
    },
    [id]
  );
```

`useCallback` memoizes a function reference.

It does not memoize the result of calling that function.

---

## 23. Memoization and Referential Equality

Memoization often depends on stable references.

If every call passes a freshly created object:

```js
memoized({
  page: 1,
});
```

a reference-keyed cache cannot reuse the previous entry.

This is why referential stability matters.

---

## 24. Stale Memoized Data

If a memoized result depends on hidden mutable state, the cache can become stale.

So always ask:

```text
What exactly determines
the output?
```

Those values must be represented in cache invalidation/key logic.

---

## 25. Cache Invalidation

A famous difficulty:

> When should cached data be removed or recomputed?

Strategies include:

- dependency change
- TTL
- manual invalidation
- LRU eviction
- versioned keys

Memoization is simple only when result validity is simple.

---

## 26. Interview Questions

### What is memoization?

Caching function results based on inputs so repeated calls can reuse previous results.

### Memoization vs caching?

Memoization specifically caches function outputs by input; caching is broader.

### When is memoization useful?

For deterministic, expensive, frequently repeated computations.

### What is a downside?

Memory growth, stale results, and cache-management complexity.

---

## Key Takeaways

- Memoization caches function results.
- Closures often hold the cache.
- Pure/deterministic functions fit best.
- Cache keys must represent all output dependencies.
- Object-key memoization uses reference identity.
- WeakMap can help with object-key lifetime.
- Memoization can improve recursive algorithms dramatically.
- Memoization has memory and invalidation costs.
- In React, `useMemo`, `useCallback`, and `React.memo` solve related but different problems.
