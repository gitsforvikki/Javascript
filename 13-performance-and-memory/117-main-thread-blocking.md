# Lesson 117 — Main Thread Blocking and Long Tasks

In browser JavaScript, much of your application work happens on the **main thread**.

The main thread commonly handles:

- JavaScript execution
- DOM events
- style calculation
- layout
- paint coordination

If JavaScript occupies the main thread for too long, the page becomes less responsive.

---

## 1. Blocking Example

```js
const start =
  performance.now();

while (
  performance.now() -
    start <
  5000
) {
  // busy work
}
```

For five seconds, the browser cannot promptly respond to many user interactions.

---

## 2. Symptoms of Main-Thread Blocking

Users may experience:

- frozen clicks
- delayed typing
- scrolling jank
- animation stutter
- delayed paints
- input latency

---

## 3. Event Loop Connection

Remember:

```text
current task
↓
must finish
↓
then microtasks
↓
then next task/render opportunity
```

A huge synchronous task delays everything behind it.

---

## 4. Long Task Concept

In web performance tooling, a task taking roughly more than 50 ms is commonly treated as a **long task**.

Why 50 ms?

Because work that occupies the main thread that long can noticeably delay interaction handling.

---

## 5. 50ms Is Not a Performance Target

Do not interpret:

```text
49ms = good
50ms = bad
```

Smaller work is generally better for responsiveness.

The long-task threshold is primarily a diagnostic concept.

---

## 6. CPU-Heavy Loop

```js
function calculate() {
  let result = 0;

  for (
    let i = 0;
    i < 1_000_000_000;
    i++
  ) {
    result += i;
  }

  return result;
}
```

Calling this synchronously blocks the main thread until complete.

---

## 7. async Does Not Fix CPU Blocking

```js
async function calculate() {
  // huge synchronous loop
}
```

Adding `async` does not move the work to another thread.

The loop still blocks.

---

## 8. Promise Does Not Make CPU Work Parallel

This:

```js
Promise.resolve()
  .then(() => {
    heavyCalculation();
  });
```

moves when the calculation begins, but once it starts, it still occupies the main thread.

---

## 9. setTimeout Does Not Remove CPU Cost

```js
setTimeout(
  heavyCalculation,
  0
);
```

delays the heavy work until a later task.

But when it runs, it still blocks.

Useful distinction:

```text
defer work
≠
offload work
```

---

## 10. Chunking Work

Instead of processing everything at once:

```js
for (
  const item
  of hugeList
) {
  process(item);
}
```

break into chunks.

Conceptually:

```text
process chunk
↓
yield to browser
↓
process next chunk
```

---

## 11. Basic Chunking Example

```js
function processInChunks(
  items,
  chunkSize = 1000
) {
  let index = 0;

  function runChunk() {
    const end =
      Math.min(
        index +
          chunkSize,
        items.length
      );

    while (
      index < end
    ) {
      process(
        items[index]
      );

      index++;
    }

    if (
      index <
      items.length
    ) {
      setTimeout(
        runChunk,
        0
      );
    }
  }

  runChunk();
}
```

This yields between chunks so other tasks can run.

---

## 12. Chunking Tradeoff

Chunking improves responsiveness but can increase total completion time due to scheduling overhead.

The goal is not maximum raw speed.

The goal may be:

> keep UI responsive while work progresses.

---

## 13. requestAnimationFrame Chunking

For work tied to visual updates:

```js
requestAnimationFrame(
  nextChunk
);
```

can help coordinate work with rendering.

Keep work inside each frame small.

---

## 14. requestIdleCallback

Where supported:

```js
requestIdleCallback(
  callback
);
```

lets work run during browser idle opportunities.

Good for low-priority background tasks.

Not suitable for urgent work with strict timing unless paired with appropriate fallback/deadline logic.

---

## 15. scheduler APIs

Modern browsers are evolving APIs for task scheduling/priorities.

These can help express:

- user-blocking
- user-visible
- background work

But support varies, so rely on the platform you actually target.

---

## 16. Input Responsiveness

A browser cannot process an input event while the main thread is busy executing long JavaScript.

Flow:

```text
user clicks
↓
event queued
↓
main thread still busy
↓
click handler delayed
```

---

## 17. INP Connection

Interaction to Next Paint (INP) measures responsiveness to user interactions across a page visit.

Long main-thread work can worsen interaction latency.

The performance lesson:

> minimize blocking work around user interactions.

---

## 18. Microtask Starvation

Microtasks have high priority.

This can be dangerous:

```js
function loop() {
  queueMicrotask(
    loop
  );
}

loop();
```

The microtask queue may never empty.

Tasks and rendering can be starved.

---

## 19. Promise Chains Can Also Dominate

A huge stream of continuously generated microtasks can delay rendering.

Promises are not automatically "free".

Scheduling priority matters.

---

## 20. Heavy JSON Work

Large:

```js
JSON.parse(
  hugeString
);
```

or:

```js
JSON.stringify(
  hugeObject
);
```

is synchronous and can block the main thread.

For truly large data, consider:

- worker processing
- streaming
- server-side preprocessing

---

## 21. Rendering Large Lists

Rendering thousands of items can create:

- JavaScript work
- DOM work
- layout work

Use:

- pagination
- virtualization
- incremental rendering

---

## 22. Debounce and Throttle Connection

Debouncing/throttling can reduce how often expensive handlers run.

But if the handler itself takes 500 ms:

```text
throttle
does not make it cheap
```

Optimize or offload the work itself.

---

## 23. Web Worker Decision

If work is:

- CPU-heavy
- independent from direct DOM access
- expensive enough to justify messaging overhead

a Web Worker may help.

Lesson 118 covers this.

---

## 24. Measure Long Tasks

Use browser DevTools Performance panel.

Look for:

- long yellow scripting blocks
- delayed input
- dropped frames
- expensive functions

Profiling tells you where time is actually spent.

---

## 25. Break Up Work vs Worker

Use chunking when:

- work is moderate
- data stays on main thread
- implementation simplicity matters

Use worker when:

- work is truly CPU-heavy
- parallel/off-main-thread execution helps
- message transfer overhead is acceptable

---

## 26. React Connection

Expensive work during rendering:

```jsx
function Component() {
  const result =
    veryExpensiveCalculation();

  return <div>{result}</div>;
}
```

can block rendering.

Possible strategies:

- memoize legitimate repeated calculation
- preprocess data
- defer non-urgent UI work
- move CPU-heavy work to worker
- reduce data size

---

## 27. React Concurrent Features

Modern React can prioritize/defer some rendering work.

But:

> React concurrency does not make arbitrary CPU-heavy JavaScript execute on another thread.

A massive synchronous calculation can still block the browser.

---

## Interview Questions

### Does async/await prevent main-thread blocking?

No.

### Does setTimeout move work to another thread?

No.

### What is a long task?

A main-thread task lasting long enough—commonly over about 50 ms—to risk delaying responsiveness.

### How can you reduce blocking?

Reduce work, split it into chunks, defer low-priority work, or offload CPU-heavy tasks to a worker.

---

## Key Takeaways

- Long synchronous JavaScript blocks the main thread.
- Async syntax does not make CPU work parallel.
- Deferring work is different from offloading it.
- Chunking lets the browser regain control between pieces.
- Microtasks can starve tasks/rendering if abused.
- Large parsing/rendering operations can also block.
- Measure long tasks before choosing an optimization.
- Web Workers are appropriate for some CPU-heavy workloads.
