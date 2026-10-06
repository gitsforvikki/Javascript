# Lesson 39 — Closures

Closures are one of the most important JavaScript interview topics.

A closure is not special syntax.

It naturally exists because JavaScript functions are lexically scoped.

The key definition:

> A closure is a function together with continued access to the lexical environment in which that function was created, even after the outer function has finished executing.

---

## 1. Basic Example

```js
function outer() {
  const message =
    "Hello";

  function inner() {
    console.log(
      message
    );
  }

  return inner;
}

const fn =
  outer();

fn();
```

Output:

```text
Hello
```

---

## 2. Why This Matters

By the time:

```js
fn();
```

runs, `outer()` has already returned.

Its execution context is no longer on the call stack.

But `inner()` still accesses:

```js
message
```

That continued lexical access is closure behavior.

---

## 3. Correct Mental Model

Wrong:

```text
closure keeps outer function
on the call stack
```

Correct:

```text
outer returns
↓
outer execution context leaves stack
↓
inner still references needed bindings
↓
those bindings remain reachable
```

---

## 4. Closures Capture Bindings, Not Frozen Copies

```js
function outer() {
  let count = 0;

  function increment() {
    count++;
  }

  function read() {
    return count;
  }

  return {
    increment,
    read,
  };
}

const counter =
  outer();

counter.increment();
counter.increment();

console.log(
  counter.read()
);
```

Output:

```text
2
```

Both functions use the same `count` binding.

---

## 5. Counter Example

```js
function createCounter() {
  let count = 0;

  return function () {
    count++;

    return count;
  };
}

const counter =
  createCounter();

counter();
// 1

counter();
// 2
```

---

## 6. Independent Closures

```js
const a =
  createCounter();

const b =
  createCounter();

a();
// 1

a();
// 2

b();
// 1
```

Each call to `createCounter()` creates a separate lexical environment.

---

## 7. Private State

```js
function createBankAccount() {
  let balance = 0;

  return {
    deposit(amount) {
      balance += amount;
    },

    getBalance() {
      return balance;
    },
  };
}
```

`balance` is not directly accessible from outside.

Closures provide encapsulation.

---

## 8. Function Factory

```js
function multiplyBy(
  factor
) {
  return function (
    value
  ) {
    return (
      value *
      factor
    );
  };
}

const double =
  multiplyBy(2);

const triple =
  multiplyBy(3);
```

Each function remembers a different `factor`.

---

## 9. Event Handler Closure

```js
function setupButton(
  selector
) {
  const button =
    document.querySelector(
      selector
    );

  let clicks = 0;

  button.addEventListener(
    "click",
    () => {
      clicks++;

      console.log(
        clicks
      );
    }
  );
}
```

The callback runs later but keeps access to `clicks`.

---

## 10. setTimeout Closure

```js
function greet(
  name
) {
  setTimeout(
    () => {
      console.log(name);
    },
    1000
  );
}

greet(
  "Vikash"
);
```

The callback still accesses `name` after `greet()` returns.

---

## 11. Classic var Loop Problem

```js
for (
  var i = 0;
  i < 3;
  i++
) {
  setTimeout(
    () => {
      console.log(i);
    },
    0
  );
}
```

Output:

```text
3
3
3
```

All callbacks close over the same function-scoped `i` binding.

---

## 12. let Fix

```js
for (
  let i = 0;
  i < 3;
  i++
) {
  setTimeout(
    () => {
      console.log(i);
    },
    0
  );
}
```

Output:

```text
0
1
2
```

A fresh per-iteration binding is created for `let`.

---

## 13. Old IIFE Fix

```js
for (
  var i = 0;
  i < 3;
  i++
) {
  (
    function (
      value
    ) {
      setTimeout(
        () => {
          console.log(
            value
          );
        },
        0
      );
    }
  )(i);
}
```

Each IIFE call creates a new parameter binding.

---

## 14. Closure Does Not Require return

```js
function outer() {
  const value = 10;

  setTimeout(
    function inner() {
      console.log(value);
    },
    1000
  );
}
```

`inner` is still a closure.

Returning a function is common, but not required.

---

## 15. Closure Sees the Current Binding Value

```js
function outer() {
  let value = 1;

  const read =
    () => value;

  value = 2;

  return read;
}

const fn =
  outer();

console.log(
  fn()
);
```

Output:

```text
2
```

Closures retain bindings, not frozen snapshots.

---

## 16. Memory Connection

```js
function createHandler() {
  const hugeData =
    new Array(
      1_000_000
    ).fill(
      "x"
    );

  return function () {
    return hugeData.length;
  };
}
```

As long as the returned function remains reachable, `hugeData` may remain reachable too.

That is not automatically a leak.

It becomes a problem only when retention is unintended.

---

## 17. React Render Closures

```jsx
function Counter() {
  const [
    count,
    setCount,
  ] =
    useState(0);

  function handleClick() {
    console.log(count);
  }

  return (
    <button
      onClick={
        handleClick
      }
    >
      Click
    </button>
  );
}
```

Each render creates a new `handleClick` that closes over that render's `count`.

---

## 18. Stale Closure Mental Model

```text
Render 1
count = 0
handler A closes over 0

Render 2
count = 1
handler B closes over 1
```

If delayed code still uses handler A, it may observe the older value.

---

## 19. useEffect Connection

```js
useEffect(
  () => {
    console.log(count);
  },
  []
);
```

This effect closes over the `count` value from the render in which that effect callback was created.

That is why dependency arrays matter.

---

## 20. Currying Connection

```js
const multiply =
  (a) =>
  (b) =>
    a * b;
```

The inner function closes over `a`.

---

## 21. Memoization Connection

```js
function memoize(
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

    const result =
      fn(key);

    cache.set(
      key,
      result
    );

    return result;
  };
}
```

The returned function closes over `cache`.

---

## 22. Debounce Connection

```js
function debounce(
  fn,
  delay
) {
  let timerId;

  return function (
    ...args
  ) {
    clearTimeout(
      timerId
    );

    timerId =
      setTimeout(
        () =>
          fn(
            ...args
          ),
        delay
      );
  };
}
```

The wrapper closes over `timerId`.

---

## 23. Common Interview Misconceptions

### Misconception 1

Closure copies outer variables.

Incorrect.

It retains access to bindings.

### Misconception 2

Outer function must still be running.

Incorrect.

The execution context can be gone from the stack.

### Misconception 3

Closures only happen when returning functions.

Incorrect.

Callbacks and event handlers also form closures.

---

## 24. Interview Output Question

```js
function outer() {
  let x = 1;

  return function () {
    x++;

    return x;
  };
}

const a =
  outer();

console.log(a());
console.log(a());
```

Output:

```text
2
3
```

---

## Interview Answer — 30 Seconds

> A closure is a function that retains access to the lexical environment where it was created, even after the outer function has returned. The outer execution context does not remain on the call stack; instead, the required lexical bindings remain reachable. Closures power private state, callbacks, currying, memoization, debounce, and also explain stale state behavior in React.

---

## Key Takeaways

- Closures come from lexical scoping.
- They retain access to bindings, not frozen copies.
- Outer stack frames do not remain active.
- Each outer call can create an independent closure environment.
- Timers and event handlers frequently use closures.
- The `var` loop issue comes from sharing one binding.
- React stale closures are ordinary JavaScript closure behavior.
