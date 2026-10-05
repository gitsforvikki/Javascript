# Lesson 104 — Throttling

Throttling limits how frequently a function can execute while events continue firing.

The core idea is:

> Allow execution at most once during a specified time window.

Throttling is useful for:

- scroll events
- resize progress
- mouse movement
- drag events
- telemetry
- repeated UI updates

---

## 1. Debounce vs Throttle First

Debounce:

```text
wait until activity stops
↓
run once
```

Throttle:

```text
activity continues
↓
allow periodic execution
```

This distinction is the most important part of the lesson.

---

## 2. Timing Example

Events:

```text
A-B-C-D-E-F-G-H
```

With a throttle window:

```text
execute A
ignore some
execute D
ignore some
execute G
```

Execution happens periodically while activity continues.

---

## 3. Basic Timestamp Throttle

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
      now - lastRun >=
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

This is a basic leading throttle.

---

## 4. How It Works

Suppose delay is:

```text
1000ms
```

First event:

```text
allowed
```

More events during next second:

```text
ignored
```

After one second:

```text
next event allowed
```

---

## 5. Timer-Based Throttle

Another implementation:

```js
function throttle(
  fn,
  delay
) {
  let waiting = false;

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

    waiting = true;

    setTimeout(
      () => {
        waiting = false;
      },
      delay
    );
  };
}
```

Again, leading execution only.

---

## 6. Leading Edge

Leading throttle means:

```text
first event in window
↓
execute immediately
```

Then suppress calls until the window opens again.

---

## 7. Trailing Edge

Trailing throttle can also run the most recent suppressed call at the end of the window.

Conceptually:

```text
first call executes
↓
more calls happen
↓
remember latest arguments
↓
window ends
↓
execute latest call
```

This is often useful for UI accuracy.

---

## 8. Why Trailing Matters

Imagine scroll position updates.

Leading-only throttle may miss the final scroll position.

Trailing execution ensures the latest state is processed after the throttle window.

---

## 9. More Complete Throttle Concept

A robust throttle usually tracks:

- last execution time
- pending timer
- latest arguments
- latest `this`
- leading option
- trailing option

Libraries often handle these edge cases better than handwritten production implementations.

---

## 10. Scroll Example

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

Even if scroll fires many times per second, your handler runs at a controlled rate.

---

## 11. Mousemove Example

```js
const handleMove =
  throttle(
    (event) => {
      updatePosition(
        event.clientX,
        event.clientY
      );
    },
    50
  );

window.addEventListener(
  "mousemove",
  handleMove
);
```

Useful when continuous updates are needed but full event frequency is unnecessary.

---

## 12. Debounce Search vs Throttle Scroll

Good mental mapping:

```text
Search input
→ debounce

Scroll progress
→ throttle
```

Search usually cares about the final value after typing stops.

Scroll UI often needs periodic updates while scrolling continues.

---

## 13. API Rate Limiting Use Case

Throttle can restrict how often an action is initiated.

Example:

```js
const sendLocation =
  throttle(
    (position) => {
      sendToServer(
        position
      );
    },
    1000
  );
```

At most one update per second is sent.

But client-side throttle is not a security mechanism; the server must enforce its own rate limits.

---

## 14. Throttle Does Not Guarantee Exact Timing

Timers and event-loop scheduling can delay execution.

So:

```text
200ms throttle
```

means roughly:

> do not run more frequently than the configured interval under this implementation

not:

> execute at exact 200ms clock boundaries.

---

## 15. requestAnimationFrame for Visual Work

For scroll/animation-related visual updates, `requestAnimationFrame()` may be a better fit than an arbitrary timer throttle.

Example:

```js
let scheduled = false;

window.addEventListener(
  "scroll",
  () => {
    if (scheduled) {
      return;
    }

    scheduled = true;

    requestAnimationFrame(
      () => {
        updateUi();

        scheduled =
          false;
      }
    );
  }
);
```

This aligns visual work with browser rendering.

---

## 16. requestAnimationFrame vs Throttle

Timer throttle:

```text
control by milliseconds
```

`requestAnimationFrame`:

```text
align work with next browser paint
```

For visual DOM updates, rAF is often more natural.

---

## 17. Throttle with cancel()

A production wrapper may expose:

```js
throttled.cancel();
```

This is especially useful for trailing timers.

When a component or page is destroyed, pending callbacks should not continue unexpectedly.

---

## 18. React Function Identity Problem

Bad:

```jsx
function Component() {
  const onScroll =
    throttle(
      handleScroll,
      200
    );

  // new throttle each render
}
```

This resets internal throttle state.

Use stable creation when necessary.

---

## 19. Stable Throttle in React

Conceptually:

```js
const throttled =
  useMemo(
    () =>
      throttle(
        handler,
        200
      ),
    [handler]
  );
```

But if `handler` changes every render, the throttle still gets recreated.

You may need:

- `useCallback`
- refs
- carefully chosen dependencies

Again, do not memoize blindly.

---

## 20. Cleanup in React

```js
useEffect(() => {
  window.addEventListener(
    "scroll",
    throttled
  );

  return () => {
    window.removeEventListener(
      "scroll",
      throttled
    );

    throttled.cancel?.();
  };
}, [throttled]);
```

Stable function identity is required for correct listener removal.

---

## 21. Stale Closure Risk

Like debounce, throttled callbacks run later/periodically.

They may close over stale state.

Strategies include:

- pass current values as arguments
- use refs for latest mutable value
- rebuild when dependencies truly change

This is a closure-management problem, not a throttle-specific mystery.

---

## 22. Throttle and Async Functions

If the throttled function is async:

```js
const save =
  throttle(
    async () => {
      await sendData();
    },
    1000
  );
```

a time-based throttle does not automatically prevent overlapping async executions.

Example:

```text
call 1 starts request
↓
1 second passes
↓
call 2 starts
↓
request 1 still pending
```

If you require one-at-a-time execution, add concurrency control separately.

---

## 23. Throttle vs Concurrency Control

Throttle controls:

```text
how often operations start
```

Concurrency control controls:

```text
how many operations can be in progress
```

These are different concepts.

---

## 24. Throttle vs Rate Limiting

Client throttle:

```text
UI/application optimization
```

Server rate limiting:

```text
authoritative protection/policy
```

A user can bypass frontend throttling.

Never rely on it to protect a server.

---

## 25. Throttle Timing Diagram

```text
events:
A-B-C-D-E-F-G-H

windows:
|----|----|----|

runs:
A    D    G
```

Exact output depends on leading/trailing behavior.

---

## 26. Debounce vs Throttle Comparison

| Feature | Debounce | Throttle |
| --- | --- | --- |
| Main goal | Wait for quiet | Limit frequency |
| Runs during continuous activity | Usually no | Yes |
| Search input | Excellent | Sometimes |
| Scroll tracking | Usually poor | Excellent |
| Final trailing call | Common | Optional |
| Timer resets on every call | Yes | Not usually in the same way |

---

## 27. Which One Should I Choose?

Ask:

```text
Do I care mainly about
the final event after
activity stops?
↓
debounce

Do I need updates
while activity continues,
but less frequently?
↓
throttle
```

---

## 28. Common Mistakes

### Mistake 1

Using debounce for continuous visual feedback.

### Mistake 2

Using throttle for search and still sending many unnecessary requests.

### Mistake 3

Recreating throttle/debounce wrapper on every React render.

### Mistake 4

Assuming throttle prevents overlapping async requests.

### Mistake 5

Treating frontend throttle as server security.

---

## 29. Interview Questions

### What is throttling?

A technique that limits a function to executing at most once per configured interval while calls continue.

### Debounce vs throttle?

Debounce waits until activity stops. Throttle permits periodic execution during ongoing activity.

### Good throttle use cases?

Scroll, mousemove, drag, resize progress, telemetry.

### Does throttle cancel existing async work?

No.

---

## Section 12 Mental Model So Far

```text
mutation
↓
same object changes

immutability
↓
new reference created

reference identity
↓
=== for objects

shallow equality
↓
compare one level

deep equality
↓
recursive structure

debounce
↓
wait for inactivity

throttle
↓
limit frequency
```

---

## Key Takeaways

- Throttle limits execution frequency.
- It is designed for continuous event streams.
- Leading and trailing behavior matter.
- Throttle does not guarantee exact wall-clock timing.
- `requestAnimationFrame` can be better for visual updates.
- Async overlap requires separate control.
- React requires stable wrapper identity and cleanup.
- Debounce and throttle solve different problems.
