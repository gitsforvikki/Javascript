# Lesson 40 — Practical Closure Patterns and Common Mistakes

Closures are not only an interview theory topic.

They appear constantly in real JavaScript code:

- private state
- function factories
- callbacks
- event handlers
- memoization
- debounce and throttle
- module patterns
- React hooks

This lesson focuses on practical closure usage and the mistakes developers commonly make.

---

## 1. Closure Recap

A closure is created when a function retains access to variables from the lexical environment where it was defined.

Example:

```js
function outer() {
  let count = 0;

  return function () {
    count++;

    return count;
  };
}
```

The returned function keeps access to `count`.

---

## 2. Pattern — Private State

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
    },

    decrement() {
      count--;
    },

    getValue() {
      return count;
    },
  };
}
```

Usage:

```js
const counter =
  createCounter();

counter.increment();

console.log(
  counter.getValue()
);
// 1
```

`count` cannot be changed directly from outside.

---

## 3. Pattern — Function Factory

```js
function createMultiplier(
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
  createMultiplier(2);

const triple =
  createMultiplier(3);
```

Each returned function keeps a different `factor`.

---

## 4. Pattern — Configuration Once, Use Many Times

```js
function createApiClient(
  baseUrl
) {
  return async function (
    path
  ) {
    return fetch(
      baseUrl +
      path
    );
  };
}

const api =
  createApiClient(
    "/api"
  );
```

Now every call automatically uses the same base URL.

---

## 5. Pattern — Memoization

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

## 6. Pattern — Debounce

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
        () => {
          fn.apply(
            this,
            args
          );
        },
        delay
      );
  };
}
```

The wrapper closes over `timerId`.

That state survives between calls.

---

## 7. Pattern — Event Handlers

```js
function setupButton(
  button
) {
  let clicks = 0;

  function handleClick() {
    clicks++;

    console.log(
      clicks
    );
  }

  button.addEventListener(
    "click",
    handleClick
  );
}
```

The handler executes later but still has access to `clicks`.

---

## 8. Pattern — Module Style Encapsulation

Before ES modules became standard, closures were often used to create module-like APIs.

```js
const userModule =
  (
    function () {
      let currentUser =
        null;

      function login(
        user
      ) {
        currentUser =
          user;
      }

      function logout() {
        currentUser =
          null;
      }

      function getUser() {
        return currentUser;
      }

      return {
        login,
        logout,
        getUser,
      };
    }
  )();
```

`currentUser` remains private.

---

## 9. Mistake — var in Loops

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

Why?

All callbacks share the same `i` binding.

---

## 10. Fix with let

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

Each iteration gets a separate binding.

---

## 11. Mistake — Thinking Closures Capture Values

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

The closure accesses the binding, not a frozen copy of `1`.

---

## 12. Mistake — Keeping Large Objects Alive

```js
function createHandler() {
  const hugeData =
    new Array(
      1_000_000
    ).fill(
      "data"
    );

  return function () {
    return hugeData.length;
  };
}
```

As long as the returned function remains reachable, `hugeData` may remain reachable.

This can be intentional or wasteful.

---

## 13. Mistake — Forgotten Event Listener

```js
function mount() {
  const largeState =
    loadLargeState();

  function handleResize() {
    console.log(
      largeState.length
    );
  }

  window.addEventListener(
    "resize",
    handleResize
  );
}
```

If the owner is destroyed but the listener is never removed, the listener may keep `largeState` reachable.

Cleanup matters.

---

## 14. Fix — Explicit Cleanup

```js
function mount() {
  const largeState =
    loadLargeState();

  function handleResize() {
    console.log(
      largeState.length
    );
  }

  window.addEventListener(
    "resize",
    handleResize
  );

  return function cleanup() {
    window.removeEventListener(
      "resize",
      handleResize
    );
  };
}
```

---

## 15. Mistake — Stale Closure in React

```jsx
function Counter() {
  const [
    count,
    setCount,
  ] =
    useState(0);

  function logLater() {
    setTimeout(
      () => {
        console.log(
          count
        );
      },
      3000
    );
  }
}
```

The timeout callback closes over the `count` from the render in which `logLater` was created.

If `count` changes later, the callback may still print the older value.

---

## 16. React Mental Model

```text
Render 1
count = 0
closure A remembers render 1 bindings

Render 2
count = 1
closure B remembers render 2 bindings
```

React does not mutate the old render's local variable.

It invokes the component again and creates a new lexical environment.

---

## 17. Fix — Pass Current Value

One strategy is to pass the current value when scheduling:

```js
function logLater(
  value
) {
  setTimeout(
    () => {
      console.log(
        value
      );
    },
    3000
  );
}
```

Now the captured value is explicit.

---

## 18. Fix — Functional State Update

For state updates based on previous state:

```js
setCount(
  (previous) =>
    previous + 1
);
```

This avoids relying on a possibly stale closed-over `count`.

---

## 19. Fix — Ref for Latest Mutable Value

Sometimes you need access to the latest value from delayed callbacks.

Conceptually:

```js
const latest =
  useRef(value);

latest.current =
  value;
```

Then delayed code can read:

```js
latest.current
```

This is a React-specific escape hatch.

---

## 20. Mistake — Assuming Closure Means Memory Leak

Closures are not memory leaks.

A leak happens when data remains reachable longer than necessary.

Closure is simply one possible reason something remains reachable.

---

## 21. Mistake — Returning Too Much State

Bad:

```js
function createThing() {
  const hugeConfig =
    loadHugeConfig();

  const smallValue =
    hugeConfig.id;

  return function () {
    return hugeConfig.id;
  };
}
```

If you only need `id`, avoid retaining the entire object.

Better:

```js
function createThing() {
  const hugeConfig =
    loadHugeConfig();

  const id =
    hugeConfig.id;

  return function () {
    return id;
  };
}
```

Now the closure need not conceptually retain the full config just to access the id.

---

## 22. Pattern — Once Function

```js
function once(
  fn
) {
  let called =
    false;

  let result;

  return function (
    ...args
  ) {
    if (
      !called
    ) {
      result =
        fn.apply(
          this,
          args
        );

      called =
        true;
    }

    return result;
  };
}
```

The closure stores:

- whether the function already ran
- the cached result

---

## 23. Pattern — ID Generator

```js
function createIdGenerator() {
  let id = 0;

  return function () {
    id++;

    return id;
  };
}

const nextId =
  createIdGenerator();
```

Each call updates private state.

---

## 24. Pattern — Access Control

```js
function createSecret(
  secret
) {
  return {
    verify(
      value
    ) {
      return (
        value ===
        secret
      );
    },
  };
}
```

The secret is kept inside the closure.

---

## 25. Closures and Garbage Collection

If no reachable function/object references a captured environment anymore, the environment can become unreachable.

Conceptually:

```text
closure reachable
→ captured bindings reachable

closure unreachable
→ captured bindings may become collectable
```

Garbage collection is based on reachability.

---

## 26. Interview Trap — Does Every Closure Keep Every Outer Variable?

Do not claim:

> Every closure stores every variable from every outer scope.

The safe conceptual statement is:

> A closure retains access to the lexical environment it needs. Engines may optimize the exact internal storage.

---

## 27. Interview Trap — Does Returning a Function Create Closure?

Returning a function often makes closure behavior visible, but closures are not limited to returned functions.

Callbacks, timers, promises, and event handlers can also close over variables.

---

## 28. Interview Problem

What is the output?

```js
function create() {
  let value = 0;

  return {
    increment() {
      value++;
    },

    read() {
      return value;
    },
  };
}

const a =
  create();

const b =
  create();

a.increment();
a.increment();
b.increment();

console.log(
  a.read(),
  b.read()
);
```

Output:

```text
2 1
```

Each `create()` call has a separate lexical environment.

---

## Interview Answer — 30 Seconds

> Closures are used for private state, function factories, memoization, debounce, callbacks, and modules. The main mistakes are assuming they capture frozen values, forgetting that long-lived callbacks can retain large objects, and running into stale closures in React. Closures themselves are not leaks; unwanted reachability is the real problem.

---

## Key Takeaways

- Closures are practical, not just theoretical.
- They enable private state and reusable configured functions.
- Event handlers, timers, memoization, and debounce rely on them.
- `var` loop bugs come from sharing one binding.
- React stale closures come from render-specific lexical environments.
- Long-lived closures can retain unnecessary data.
- Closures are not automatically memory leaks.
