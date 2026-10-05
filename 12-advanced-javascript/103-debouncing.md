# Lesson 103 — Debouncing

Debouncing is a technique used to control how often a function executes when an event fires repeatedly.

The core idea is:

> Wait until calls stop for a specified delay, then run the function once.

This is especially useful for:

- search inputs
- resize handlers
- validation
- autosave
- expensive calculations

---

## 1. The Problem

Imagine:

```js
input.addEventListener(
  "input",
  () => {
    searchApi();
  }
);
```

If the user types:

```text
j
ja
jav
java
```

you may send four requests.

That can be wasteful.

---

## 2. Debounce Mental Model

Suppose delay is:

```text
300ms
```

Calls happen:

```text
call ──100ms── call ──100ms── call
                         │
                         └── wait 300ms
                              ↓
                           execute
```

Every new call resets the timer.

---

## 3. Basic Debounce Implementation

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

Usage:

```js
const debouncedSearch =
  debounce(
    searchApi,
    300
  );

input.addEventListener(
  "input",
  debouncedSearch
);
```

---

## 4. Why clearTimeout() Matters

Without:

```js
clearTimeout(
  timerId
);
```

every scheduled call would still execute later.

Debouncing works because each new call cancels the previous pending timer.

---

## 5. Closure Connection

`timerId` remains available because the returned function closes over it.

```text
debounce()
↓
timerId created
↓
returned function keeps access
↓
each call updates same timerId
```

This is a direct real-world use of closures.

---

## 6. Preserve this

The basic version:

```js
fn(...args);
```

may lose the caller's intended `this`.

A more complete implementation:

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

Now the wrapped function receives the original call-site context.

---

## 7. Event Example

```js
const searchInput =
  document.querySelector(
    "#search"
  );

const handleSearch =
  debounce(
    (event) => {
      console.log(
        event.target.value
      );
    },
    300
  );

searchInput.addEventListener(
  "input",
  handleSearch
);
```

The callback runs only after input activity pauses.

---

## 8. Trailing Debounce

The implementation above is a **trailing-edge debounce**.

Meaning:

```text
calls happen
↓
wait until quiet
↓
execute at end
```

This is the most common debounce behavior.

---

## 9. Leading Debounce

Sometimes you want the function to run immediately on the first call, then suppress repeated calls during the delay window.

Conceptually:

```text
first call
↓
execute immediately
↓
ignore/reset during delay
↓
ready again later
```

---

## 10. Leading + Trailing Options

A more configurable debounce may support:

```js
{
  leading: true,
  trailing: true,
}
```

Meaning:

- leading: run at start
- trailing: run after calls stop

Libraries such as Lodash expose this style of control.

---

## 11. Leading Debounce Example

Educational implementation:

```js
function debounceLeading(
  fn,
  delay
) {
  let timerId;

  return function (
    ...args
  ) {
    const shouldCall =
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
      shouldCall
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

## 12. Debounce with cancel()

Sometimes pending work should be cancelled explicitly.

```js
function debounce(
  fn,
  delay
) {
  let timerId;

  function debounced(
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

Useful when:

- component unmounts
- user leaves page
- operation becomes irrelevant

---

## 13. Debounce with flush()

A `flush()` feature can force pending work to execute immediately.

This is useful when:

- form is closing
- page is unloading
- autosave must complete now

Implementation details vary.

---

## 14. Debounce and API Search

Without debounce:

```text
r
re
rea
reac
react

→ 5 API calls
```

With 300ms debounce:

```text
r
re
rea
reac
react
↓
pause 300ms
↓
1 API call
```

This reduces unnecessary requests.

---

## 15. Debounce Does Not Cancel Previous Network Requests Automatically

Important:

Debouncing prevents **new function executions**.

But if an earlier request already started, debounce does not cancel it.

For search, combine:

```text
debounce
+
AbortController
```

if stale requests need cancellation.

---

## 16. Debounce + AbortController Concept

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

Debounce controls frequency.

AbortController handles already-started stale requests.

---

## 17. Debounce and Async Functions

You can debounce an async function, but remember:

```js
const debounced =
  debounce(
    async () => {
      return await task();
    },
    300
  );
```

The wrapper in a simple implementation does not automatically return a useful Promise to each caller.

Production-quality async debouncing may need explicit Promise-handling semantics.

---

## 18. React Problem — Recreating Debounced Function

Bad:

```jsx
function Search() {
  const handleChange =
    debounce(
      searchApi,
      300
    );

  return (
    <input
      onChange={
        handleChange
      }
    />
  );
}
```

Every render creates a new debounced function and new timer state.

That can break the intended debounce behavior.

---

## 19. Stable Debounced Function in React

One approach:

```js
const debouncedSearch =
  useMemo(
    () =>
      debounce(
        searchApi,
        300
      ),
    []
  );
```

Then clean up:

```js
useEffect(() => {
  return () => {
    debouncedSearch
      .cancel?.();
  };
}, [debouncedSearch]);
```

Be careful with stale closures if `searchApi` or dependencies change.

---

## 20. Stale Closure Connection

Suppose the debounced callback closes over old state.

Because it runs later, it may read stale values.

Solutions depend on the use case:

- include dependencies
- use refs
- pass latest values as arguments
- recreate intentionally

This connects your closure and React lessons.

---

## 21. Autosave Example

```js
const saveDraft =
  debounce(
    (content) => {
      localStorage.setItem(
        "draft",
        content
      );
    },
    500
  );
```

Each keystroke resets the timer.

Save occurs only after typing pauses.

---

## 22. Resize Example

```js
const handleResize =
  debounce(
    () => {
      calculateLayout();
    },
    200
  );

window.addEventListener(
  "resize",
  handleResize
);
```

Useful when only the final size matters.

---

## 23. When Debounce Is Appropriate

Use debounce when you care about:

> the final action after a burst of activity.

Examples:

- search
- form validation
- autosave
- resize completion
- filter input

---

## 24. When Debounce Is Wrong

For continuous updates such as:

- scroll progress
- drag position
- game input

waiting until activity stops may feel wrong.

Throttling is often a better fit.

---

## 25. Debounce Timing Diagram

```text
events:
A---B---C-----------

timer:
reset
    reset
        reset

execution:
             C
```

Only the final call runs after quiet time.

---

## 26. Interview Questions

### What is debouncing?

A technique that delays function execution until calls stop occurring for a specified period.

### What happens on every new call?

The previous pending timer is cancelled and a new timer is scheduled.

### Debounce use cases?

Search input, autosave, validation, resize completion.

### Does debounce cancel a request that already started?

No.

---

## Key Takeaways

- Debounce waits for inactivity.
- Each new call resets the timer.
- Closures store timer state.
- Trailing debounce runs after the burst.
- Leading debounce runs at the beginning.
- Cancellation support is useful.
- Debounce controls call frequency, not existing network requests.
- Stable function identity matters in React.
