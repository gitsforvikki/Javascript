# Lesson 157 — Implement Debounce

Debouncing is an interview-favorite because it tests:

- closures
- timers
- function arguments
- preserving `this`
- cancellation
- leading/trailing execution
- async/event-driven thinking

The core rule is:

> Run the function only after calls have stopped for a given amount of time.

---

## 1. Basic Use Case

Imagine a search input:

```js
input.addEventListener(
  "input",
  () => {
    searchApi();
  }
);
```

If the user types quickly, this may call the API many times.

Debounce reduces unnecessary calls.

---

## 2. Timing Mental Model

Delay:

```text
300ms
```

Calls:

```text
A---B---C-----------

A schedules timer
B resets timer
C resets timer

after 300ms silence
↓
C executes
```

---

## 3. Basic Implementation

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
          fn(...args);
        },
        delay
      );
  };
}
```

This is a simple trailing-edge debounce.

---

## 4. Why Closure Is Required

`timerId` must survive across multiple calls.

```text
debounce()
↓
timerId created once
↓
returned function remembers it
↓
every new call clears previous timer
```

This is a direct closure use case.

---

## 5. Preserve this

This version:

```js
fn(...args);
```

may lose the caller's `this`.

Better:

```js
function debounce(
  fn,
  delay
) {
  let timerId;

  return function (
    ...args
  ) {
    const context =
      this;

    clearTimeout(
      timerId
    );

    timerId =
      setTimeout(
        () => {
          fn.apply(
            context,
            args
          );
        },
        delay
      );
  };
}
```

Now the original invocation context is preserved.

---

## 6. Why Arrow Function Inside setTimeout Is Fine

We use:

```js
() => {
  fn.apply(
    context,
    args
  );
}
```

The arrow itself does not need dynamic `this`.

We already stored the correct `context`.

---

## 7. Cancel Support

A useful implementation exposes:

```js
debounced.cancel();
```

Example:

```js
function debounce(
  fn,
  delay
) {
  let timerId;

  function debounced(
    ...args
  ) {
    const context =
      this;

    clearTimeout(
      timerId
    );

    timerId =
      setTimeout(
        () => {
          timerId =
            undefined;

          fn.apply(
            context,
            args
          );
        },
        delay
      );
  }

  debounced.cancel =
    function () {
      clearTimeout(
        timerId
      );

      timerId =
        undefined;
    };

  return debounced;
}
```

---

## 8. Leading Debounce

Sometimes you want:

```text
first call
↓
run immediately
↓
ignore/reset during delay
```

Example:

```js
function debounceLeading(
  fn,
  delay
) {
  let timerId;

  return function (
    ...args
  ) {
    const shouldRun =
      !timerId;

    clearTimeout(
      timerId
    );

    timerId =
      setTimeout(
        () => {
          timerId =
            undefined;
        },
        delay
      );

    if (
      shouldRun
    ) {
      fn.apply(
        this,
        args
      );
    }
  };
}
```

---

## 9. Leading vs Trailing

```text
Leading
→ run at start

Trailing
→ run after activity stops
```

Many production libraries let you choose:

```js
{
  leading: true,
  trailing: true
}
```

---

## 10. More Configurable Debounce

Educational implementation:

```js
function debounce(
  fn,
  delay,
  {
    leading = false,
    trailing = true,
  } = {}
) {
  let timerId;
  let lastArgs;
  let lastThis;

  function invoke() {
    const args =
      lastArgs;

    const context =
      lastThis;

    lastArgs =
      undefined;

    lastThis =
      undefined;

    return fn.apply(
      context,
      args
    );
  }

  function debounced(
    ...args
  ) {
    const shouldCallNow =
      leading &&
      !timerId;

    lastArgs =
      args;

    lastThis =
      this;

    clearTimeout(
      timerId
    );

    timerId =
      setTimeout(
        () => {
          timerId =
            undefined;

          if (
            trailing &&
            !shouldCallNow
          ) {
            invoke();
          }
        },
        delay
      );

    if (
      shouldCallNow
    ) {
      invoke();
    }
  }

  debounced.cancel =
    function () {
      clearTimeout(
        timerId
      );

      timerId =
        undefined;

      lastArgs =
        undefined;

      lastThis =
        undefined;
    };

  return debounced;
}
```

This is useful for understanding the mechanics, though production libraries handle more edge cases.

---

## 11. Search Input Example

```js
const handleSearch =
  debounce(
    (event) => {
      console.log(
        event.target.value
      );
    },
    300
  );

input.addEventListener(
  "input",
  handleSearch
);
```

---

## 12. Debounce Does Not Cancel Existing Requests

Important:

```text
debounce
→ prevents future function starts

AbortController
→ can cancel an already-started request
```

If an earlier search request is already running, debounce alone will not stop it.

---

## 13. Debounce + AbortController

```js
let controller;

const search =
  debounce(
    async (query) => {
      controller?.abort();

      controller =
        new AbortController();

      const response =
        await fetch(
          `/api/search?q=${encodeURIComponent(
            query
          )}`,
          {
            signal:
              controller.signal,
          }
        );

      return response.json();
    },
    300
  );
```

---

## 14. React Pitfall

Bad:

```js
function Search() {
  const onChange =
    debounce(
      searchApi,
      300
    );

  // new debounce on every render
}
```

Every render creates a new timer closure.

That can break debouncing.

Use stable creation when appropriate.

---

## 15. Stale Closure Problem

A debounced callback runs later.

If it closes over stale React state, it may use old values.

Solutions:

- pass current values as arguments
- use refs
- memoize with correct dependencies

---

## 16. Common Wrong Implementation

```js
function debounce(
  fn,
  delay
) {
  return function () {
    setTimeout(
      fn,
      delay
    );
  };
}
```

Why wrong?

It never cancels previous timers.

So every call still executes.

---

## 17. Complexity

Per invocation:

```text
Time:
O(1)

Extra state:
O(1)
```

excluding argument storage and function work.

---

## Interview Explanation

A strong answer:

> Debounce keeps a timer in closure. Every call clears the previous timer and schedules a new one. Only after calls stop for the delay does the wrapped function run. I preserve `this` and arguments using `apply`, and a stronger version exposes `cancel` plus leading/trailing options.

---

## Key Takeaways

- Debounce waits for inactivity.
- Every new call resets the timer.
- Closure stores timer state.
- Preserve `this` and arguments.
- Trailing execution is most common.
- Leading execution is optional.
- Debounce does not cancel already-running async work.
