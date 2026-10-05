# Lesson 162 — Implement Currying

Currying transforms a multi-argument function into a chain of smaller function calls.

Example:

```text
sum(a, b, c)
↓
sum(a)(b)(c)
```

This interview problem tests:

- closures
- argument accumulation
- function arity
- recursion
- preserving `this`

---

## 1. Original Function

```js
function sum(
  a,
  b,
  c
) {
  return (
    a +
    b +
    c
  );
}
```

Normal call:

```js
sum(
  1,
  2,
  3
);
```

---

## 2. Simple Manual Curry

```js
function curriedSum(
  a
) {
  return function (
    b
  ) {
    return function (
      c
    ) {
      return (
        a +
        b +
        c
      );
    };
  };
}
```

Call:

```js
curriedSum(
  1
)(
  2
)(
  3
);
```

---

## 3. Generic Curry Utility

```js
function curry(
  fn
) {
  return function curried(
    ...args
  ) {
    if (
      args.length >=
      fn.length
    ) {
      return fn.apply(
        this,
        args
      );
    }

    return function (
      ...nextArgs
    ) {
      return curried.apply(
        this,
        [
          ...args,
          ...nextArgs,
        ]
      );
    };
  };
}
```

---

## 4. Usage

```js
function sum(
  a,
  b,
  c
) {
  return (
    a +
    b +
    c
  );
}

const curried =
  curry(sum);

curried(
  1
)(
  2
)(
  3
);
```

Result:

```text
6
```

---

## 5. Flexible Grouping

The same helper can support:

```js
curried(
  1,
  2
)(
  3
);
```

and:

```js
curried(
  1
)(
  2,
  3
);
```

because arguments are accumulated until enough are available.

---

## 6. Closure Mental Model

```text
first call
↓
stores [1]

second call
↓
stores [1, 2]

third call
↓
stores [1, 2, 3]

enough arguments
↓
invoke original function
```

---

## 7. Why fn.length?

```js
fn.length
```

usually represents declared parameter count.

Example:

```js
function sum(
  a,
  b,
  c
) {}

console.log(
  sum.length
);
// 3
```

We use this as the target arity.

---

## 8. fn.length Caveat

Default parameters affect `length`.

```js
function test(
  a,
  b = 10,
  c
) {}
```

`test.length` is not simply 3.

Rest parameters also do not contribute normally.

So generic currying based on `fn.length` has limitations.

---

## 9. Explicit Arity Version

A stronger API:

```js
function curry(
  fn,
  arity =
    fn.length
) {
  return function curried(
    ...args
  ) {
    if (
      args.length >=
      arity
    ) {
      return fn.apply(
        this,
        args
      );
    }

    return function (
      ...nextArgs
    ) {
      return curried.apply(
        this,
        [
          ...args,
          ...nextArgs,
        ]
      );
    };
  };
}
```

Now the caller can override arity.

---

## 10. Preserve this

Using:

```js
fn.apply(
  this,
  args
);
```

lets the eventual call preserve invocation context.

But currying with methods can still become subtle if `this` changes between partial calls.

For pure functions, this problem usually disappears.

---

## 11. Currying vs Partial Application

```text
Currying
→ transform into staged calls

Partial application
→ pre-fill some arguments
```

Example:

```js
const add10 =
  add.bind(
    null,
    10
  );
```

This is partial application, not necessarily currying.

---

## 12. Curried Predicate Example

```js
const greaterThan =
  curry(
    function (
      min,
      value
    ) {
      return (
        value >
        min
      );
    }
  );

const greaterThan18 =
  greaterThan(
    18
  );

[
  10,
  20,
  30,
].filter(
  greaterThan18
);
```

---

## 13. Infinite-Style Currying Variant

Interviewers sometimes ask:

```js
sum(1)(2)(3)(4)()
```

This is a different problem.

Example:

```js
function sum(
  initial
) {
  let total =
    initial;

  function next(
    value
  ) {
    if (
      value ===
      undefined
    ) {
      return total;
    }

    total += value;

    return next;
  }

  return next;
}
```

This is not the same as arity-based generic currying.

---

## 14. Common Wrong Implementation

```js
function curry(
  fn
) {
  return (
    a
  ) => (
    b
  ) => (
    c
  ) =>
    fn(
      a,
      b,
      c
    );
}
```

This only works for exactly three arguments.

It is not generic.

---

## 15. Complexity

For `k` accumulated arguments:

```text
Time:
O(k)
```

for repeated array accumulation in this educational version.

More optimized implementations can reduce copying.

---

## Interview Explanation

> I return a recursive closure that accumulates arguments. Once the accumulated argument count reaches the function's arity, I call the original function. Otherwise, I return another function that collects more arguments. I also mention that `fn.length` has caveats with default and rest parameters, so an explicit arity option is more robust.

---

## Key Takeaways

- Currying relies on closures.
- Arguments are accumulated across calls.
- `fn.length` is useful but imperfect.
- Flexible curry helpers can accept grouped arguments.
- Currying and partial application are related but different.
- Pure functions are easiest to curry.
