# Lesson 158 — Implement Throttle

Throttling limits how often a function can execute during continuous activity.

The core rule is:

> Execute at most once during a configured time window.

This interview question tests:

- closures
- timers
- timestamps
- preserving `this`
- leading/trailing behavior
- event frequency control

---

## 1. Throttle vs Debounce

```text
Debounce
→ wait until calls stop

Throttle
→ allow periodic execution
```

Use cases:

```text
search input
→ debounce

scroll / mousemove
→ throttle
```

---

## 2. Basic Timestamp Throttle

```js
function throttle(
  fn,
  delay
) {
  let lastRun = 0;

  return function (
    ...args
  ) {
    const now =
      Date.now();

    if (
      now -
        lastRun >=
      delay
    ) {
      lastRun = now;

      fn.apply(
        this,
        args
      );
    }
  };
}
```

This is a leading-only throttle.

---

## 3. Timing Mental Model

Delay:

```text
1000ms
```

Events:

```text
A-B-C-D-E-F-G
```

Possible executions:

```text
A-----D-----G
```

Calls keep happening, but only one runs per time window.

---

## 4. Timer-Based Throttle

```js
function throttle(
  fn,
  delay
) {
  let waiting =
    false;

  return function (
    ...args
  ) {
    if (waiting) {
      return;
    }

    fn.apply(
      this,
      args
    );

    waiting =
      true;

    setTimeout(
      () => {
        waiting =
          false;
      },
      delay
    );
  };
}
```

Also leading-only.

---

## 5. Why Leading-Only Can Miss Final State

Suppose scroll position changes many times.

If only the first call in each window executes, the latest value may never run after activity stops.

That is why trailing execution can matter.

---

## 6. Trailing Throttle

A stronger implementation remembers the latest suppressed call.

```js
function throttle(
  fn,
  delay
) {
  let timerId;
  let lastRun = 0;
  let lastArgs;
  let lastThis;

  return function (
    ...args
  ) {
    const now =
      Date.now();

    const remaining =
      delay -
      (
        now -
        lastRun
      );

    lastArgs =
      args;

    lastThis =
      this;

    if (
      remaining <= 0
    ) {
      clearTimeout(
        timerId
      );

      timerId =
        undefined;

      lastRun =
        now;

      fn.apply(
        lastThis,
        lastArgs
      );

      lastArgs =
        undefined;

      lastThis =
        undefined;
    } else if (
      !timerId
    ) {
      timerId =
        setTimeout(
          () => {
            lastRun =
              Date.now();

            timerId =
              undefined;

            fn.apply(
              lastThis,
              lastArgs
            );

            lastArgs =
              undefined;

            lastThis =
              undefined;
          },
          remaining
        );
    }
  };
}
```

This supports leading execution plus one trailing call.

---

## 7. Why Store Latest Args?

During the throttle window, many calls may happen.

For trailing execution, usually the latest state matters.

So update:

```js
lastArgs =
  args;

lastThis =
  this;
```

on every call.

---

## 8. Cancel Support

A useful throttle can expose:

```js
throttled.cancel();
```

Example:

```js
function createThrottle(
  fn,
  delay
) {
  let timerId;
  let lastRun = 0;
  let lastArgs;
  let lastThis;

  function throttled(
    ...args
  ) {
    // throttle logic
  }

  throttled.cancel =
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

      lastRun = 0;
    };

  return throttled;
}
```

---

## 9. Scroll Example

```js
const handleScroll =
  throttle(
    () => {
      console.log(
        window.scrollY
      );
    },
    200
  );

window.addEventListener(
  "scroll",
  handleScroll
);
```

---

## 10. Mousemove Example

```js
const handleMove =
  throttle(
    (event) => {
      updateCursor(
        event.clientX,
        event.clientY
      );
    },
    50
  );
```

---

## 11. requestAnimationFrame Alternative

For visual UI work, `requestAnimationFrame` can be better than time-based throttling.

```js
let scheduled =
  false;

window.addEventListener(
  "scroll",
  () => {
    if (
      scheduled
    ) {
      return;
    }

    scheduled =
      true;

    requestAnimationFrame(
      () => {
        updateUI();

        scheduled =
          false;
      }
    );
  }
);
```

This aligns updates with browser paint.

---

## 12. Throttle Does Not Prevent Async Overlap

```js
const save =
  throttle(
    async () => {
      await sendData();
    },
    1000
  );
```

If one request lasts 3 seconds, another may start after 1 second.

Throttle controls:

```text
start frequency
```

not:

```text
concurrency
```

---

## 13. React Pitfall

If you create throttle on every render:

```js
const onScroll =
  throttle(
    handler,
    200
  );
```

internal state resets each render.

Use stable function creation when needed.

---

## 14. Common Wrong Implementation

```js
function throttle(
  fn,
  delay
) {
  return function (
    ...args
  ) {
    setTimeout(
      () =>
        fn(...args),
      delay
    );
  };
}
```

This is not throttle.

It merely delays every call.

---

## 15. Leading vs Trailing

```text
Leading
→ run at beginning of window

Trailing
→ run latest pending call at end
```

Production throttle implementations often allow both to be configured.

---

## 16. Complexity

Per call:

```text
Time:
O(1)

State:
O(1)
```

excluding stored arguments/function work.

---

## Interview Explanation

A strong answer:

> Throttle stores timing state in closure and allows execution only once per interval. A basic version can use timestamps or a boolean lock. A better version also schedules the latest suppressed call on the trailing edge, preserves `this` and arguments, and exposes cancel behavior.

---

## Key Takeaways

- Throttle limits frequency during continuous activity.
- It differs from debounce.
- Leading execution runs immediately.
- Trailing execution preserves the latest pending call.
- Preserve `this` and arguments.
- Throttle does not control async concurrency.
- `requestAnimationFrame` is often better for visual work.
