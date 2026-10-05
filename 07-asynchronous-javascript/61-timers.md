# Lesson 61 — `setTimeout` and `setInterval`

Timers are often the first asynchronous APIs developers learn.

The two main timer APIs are:

- `setTimeout()`
- `setInterval()`

They look simple, but several details matter for interviews and real applications.

The most important rule is:

> A timer delay is a **minimum scheduling delay**, not a guaranteed exact execution time.

---

## 1. setTimeout()

Syntax:

```js
setTimeout(
  callback,
  delay
);
```

Example:

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

This means:

> Make the callback eligible after approximately at least 1000 ms, then let the runtime schedule it when JavaScript can execute it.

It does **not** mean:

> Run this callback exactly at 1000 ms.

---

# 2. setTimeout Does Not Pause JavaScript

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");
```

Output:

```text
A
C
B
```

The timer registration returns quickly.

JavaScript continues with `C`.

---

# 3. Why Delay Is Not Exact

Consider:

```js
setTimeout(() => {
  console.log("timer");
}, 1000);

const start = Date.now();

while (
  Date.now() - start < 5000
) {
  // block main JS thread
}
```

The timer becomes eligible after roughly one second.

But JavaScript is blocked for about five seconds.

The callback cannot run until the call stack becomes available.

So:

```text
timer requested: 1000 ms
actual execution: later than 1000 ms
```

---

# 4. setTimeout(..., 0)

A classic interview example:

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

console.log("C");
```

Output:

```text
A
C
B
```

Why?

`0` does not mean "execute immediately before the next line."

It means the callback is scheduled asynchronously with no intentional extra delay beyond runtime constraints.

Current synchronous code still finishes first.

---

# 5. Return Value / Timer ID

`setTimeout()` returns a timer identifier/handle.

Example:

```js
const timerId =
  setTimeout(() => {
    console.log("done");
  }, 5000);
```

You can cancel it:

```js
clearTimeout(timerId);
```

Then the callback will not run if cancellation happens before execution.

---

# 6. clearTimeout()

Example:

```js
const timerId =
  setTimeout(() => {
    console.log(
      "will not run"
    );
  }, 2000);

clearTimeout(timerId);
```

Useful for:

- debouncing
- component cleanup
- cancelling delayed actions
- preventing stale behavior

---

# 7. setInterval()

Syntax:

```js
setInterval(
  callback,
  delay
);
```

Example:

```js
setInterval(() => {
  console.log("tick");
}, 1000);
```

The callback is scheduled repeatedly at roughly the specified interval.

---

# 8. clearInterval()

```js
const intervalId =
  setInterval(() => {
    console.log("tick");
  }, 1000);
```

Stop it:

```js
clearInterval(
  intervalId
);
```

Without cancellation, an interval may continue indefinitely.

---

# 9. setInterval Is Not Perfectly Precise

Suppose:

```js
setInterval(() => {
  // heavy synchronous work
}, 1000);
```

If the callback takes a long time or the main thread is busy, execution can be delayed.

So do not assume:

```text
exactly every 1000.000 ms
```

Timers are scheduling tools, not high-precision clocks.

---

# 10. Repeated setTimeout vs setInterval

Sometimes repeated `setTimeout` is safer than `setInterval`.

Example:

```js
function poll() {
  setTimeout(async () => {
    await checkStatus();

    poll();
  }, 5000);
}

poll();
```

Why can this be useful?

Because the next timer is scheduled after the previous work completes.

With `setInterval`, repeated scheduling can happen independently of whether the previous async operation finished.

---

# 11. Async Work Inside setInterval

Potential issue:

```js
setInterval(
  async () => {
    await fetchData();
  },
  1000
);
```

If `fetchData()` takes longer than one second, another interval callback may begin before the previous async operation has finished.

This can cause overlapping requests.

Repeated `setTimeout` can avoid that pattern when sequential behavior is required.

---

# 12. Passing Arguments to setTimeout

Browsers/Node timer APIs can pass extra arguments:

```js
function greet(
  name,
  role
) {
  console.log(
    name,
    role
  );
}

setTimeout(
  greet,
  1000,
  "Vikash",
  "Developer"
);
```

However, closures are often clearer:

```js
setTimeout(() => {
  greet(
    "Vikash",
    "Developer"
  );
}, 1000);
```

---

# 13. Common Mistake — Calling the Function Immediately

Wrong:

```js
setTimeout(
  greet(),
  1000
);
```

This calls `greet()` immediately and passes its return value.

Correct:

```js
setTimeout(
  greet,
  1000
);
```

or:

```js
setTimeout(() => {
  greet();
}, 1000);
```

---

# 14. this and Timer Callbacks

Recall Section 5.

This can lose the intended receiver:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(
      this.name
    );
  },
};

setTimeout(
  user.showName,
  1000
);
```

The function is passed as a callback.

Do not assume the runtime later performs:

```js
user.showName();
```

A safer approach:

```js
setTimeout(() => {
  user.showName();
}, 1000);
```

or:

```js
setTimeout(
  user.showName.bind(user),
  1000
);
```

---

# 15. Timer Callback and Closures

```js
function createTimer() {
  const message =
    "Hello";

  setTimeout(() => {
    console.log(message);
  }, 1000);
}

createTimer();
```

Even after `createTimer()` returns, the callback still has access to `message` because of closure.

This connects directly to your closure lessons.

---

# 16. Loop + var Timer Problem

Classic example:

```js
for (
  var i = 1;
  i <= 3;
  i++
) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

Output:

```text
4
4
4
```

Why?

All callbacks close over the same `var` binding.

By the time they run, the loop has finished and `i` is `4`.

---

# 17. Fix with let

```js
for (
  let i = 1;
  i <= 3;
  i++
) {
  setTimeout(() => {
    console.log(i);
  }, 1000);
}
```

Output:

```text
1
2
3
```

`let` creates a new per-iteration binding for this loop pattern.

Each callback closes over its own binding.

---

# 18. Staggered Timers

```js
for (
  let i = 1;
  i <= 3;
  i++
) {
  setTimeout(() => {
    console.log(i);
  }, i * 1000);
}
```

Approximate behavior:

```text
1 → after ~1 sec
2 → after ~2 sec
3 → after ~3 sec
```

Again, actual execution can be later if the main thread is busy.

---

# 19. Countdown with setInterval

```js
let count = 5;

const id =
  setInterval(() => {
    console.log(count);

    count--;

    if (count === 0) {
      clearInterval(id);
    }
  }, 1000);
```

Output over time:

```text
5
4
3
2
1
```

Then the interval is cancelled.

---

# 20. Timer Cleanup Is Important

Unnecessary timers can cause:

- memory retention
- stale callbacks
- duplicate work
- unexpected state updates

Always clear timers when they are no longer needed.

This becomes especially important in UI components.

---

# 21. React Timer Cleanup

Example:

```jsx
useEffect(() => {
  const timerId =
    setTimeout(() => {
      console.log(
        "run later"
      );
    }, 1000);

  return () => {
    clearTimeout(
      timerId
    );
  };
}, []);
```

For intervals:

```jsx
useEffect(() => {
  const intervalId =
    setInterval(() => {
      console.log("tick");
    }, 1000);

  return () => {
    clearInterval(
      intervalId
    );
  };
}, []);
```

Cleanup prevents timers from continuing after the effect should be disposed.

---

# 22. Timer IDs Differ by Runtime

One subtle point:

In browsers, timer functions traditionally return numeric IDs.

In Node.js, the returned timer handle is typically an object.

So avoid assuming the exact runtime-specific type unless needed.

For normal application code, store the returned handle and pass it back to the corresponding clear function.

---

# 23. setTimeout and Event Loop Preview

When a timer expires:

```text
timer completes
↓
callback becomes scheduled
↓
it does NOT interrupt running JS
↓
current stack must finish
↓
event loop allows callback later
```

The full queue/event-loop behavior comes in Section 8.

---

# 24. Timers vs Promises Preview

Consider:

```js
setTimeout(() => {
  console.log("timeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise");
  });
```

A promise callback is not scheduled the same way as a timer callback.

Typically:

```text
promise
timeout
```

Why?

Promises use the microtask queue, while timers use a task/macrotask-style scheduling path.

You will study this deeply in the Event Loop lessons.

---

# 25. Interview Questions

### Does setTimeout(callback, 1000) guarantee execution after exactly one second?

No. It means the callback will not be eligible before the delay threshold, but actual execution may happen later depending on scheduling and call-stack availability.

### What does setTimeout(callback, 0) mean?

It schedules the callback asynchronously with minimal requested delay; it still waits until current synchronous JavaScript finishes and scheduling rules allow execution.

### What is the difference between setTimeout and setInterval?

`setTimeout` schedules one callback. `setInterval` repeatedly schedules callbacks until cancelled.

### Why might repeated setTimeout be preferred over setInterval for polling?

It can schedule the next run after the previous operation completes, avoiding overlapping async work.

---

# Key Takeaways

- `setTimeout` schedules one delayed callback.
- `setInterval` schedules repeated callbacks.
- Timer delays are minimum delays, not exact execution guarantees.
- `setTimeout(..., 0)` is still asynchronous.
- `clearTimeout` and `clearInterval` cancel timers.
- Long-running synchronous work delays timer callbacks.
- Passing a method directly may lose its intended `this`.
- Timer callbacks often demonstrate closures clearly.
- `var` and `let` behave differently in loop/timer closure examples.
- Repeated `setTimeout` can be safer than `setInterval` for sequential async polling.
- Timers should be cleaned up when no longer needed.
