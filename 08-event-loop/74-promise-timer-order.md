# Lesson 74 — Promise and Timer Execution Order

One of the most common JavaScript interview questions is:

> Why does a Promise callback usually run before `setTimeout(..., 0)`?

The short answer is:

```text
Promise handlers
→ microtasks

setTimeout callbacks
→ tasks/macrotasks
```

After the current synchronous task finishes:

```text
microtasks are drained
before
the next task runs
```

But to solve harder problems, you need more than that one sentence.

---

## 1. The Core Rule

Use this browser-oriented mental model:

```text
1. Run current synchronous code

2. Drain microtask queue completely

3. Run one task

4. Drain microtasks again

5. Repeat
```

Promise reactions use the microtask queue.

Timer callbacks use the task queue.

---

# 2. Basic Promise vs Timer

```js
setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise");
  });

console.log("sync");
```

Output:

```text
sync
promise
timer
```

Why?

```text
sync
↓
microtask
↓
task
```

---

# 3. Why 0ms Does Not Mean Immediate

```js
setTimeout(() => {
  console.log("A");
}, 0);

console.log("B");
```

Output:

```text
B
A
```

The timer callback cannot interrupt the current script.

`0` means approximately:

> no intentional extra delay beyond scheduling constraints.

It does not mean:

> run before remaining synchronous code.

---

# 4. Promise Handler Is Also Not Immediate

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  });

console.log("B");
```

Output:

```text
B
A
```

Promise callbacks are asynchronous too.

They just have higher scheduling priority than the next task.

---

# 5. Compare All Three Categories

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("3");
  });

console.log("4");
```

Classification:

```text
SYNC
1
4

MICROTASK
3

TASK
2
```

Output:

```text
1
4
3
2
```

---

# 6. Multiple Promise Handlers

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  });

Promise.resolve()
  .then(() => {
    console.log("B");
  });

setTimeout(() => {
  console.log("C");
}, 0);
```

Output:

```text
A
B
C
```

Both microtasks are drained before the timer task.

---

# 7. Promise Chain Ordering

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  })
  .then(() => {
    console.log("B");
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Output:

```text
A
C
B
```

Why?

Initially queued:

```text
A-handler
C-handler
```

After A runs, B becomes queued.

So queue becomes:

```text
C-handler
B-handler
```

This is why chained Promises can interleave.

---

# 8. Promise Inside Timer

```js
setTimeout(() => {
  console.log("timer");

  Promise.resolve()
    .then(() => {
      console.log(
        "promise inside timer"
      );
    });
}, 0);

setTimeout(() => {
  console.log(
    "second timer"
  );
}, 0);
```

Output:

```text
timer
promise inside timer
second timer
```

After the first timer task finishes, the newly created microtask runs before the second timer task.

---

# 9. Timer Inside Promise

```js
Promise.resolve()
  .then(() => {
    console.log("promise");

    setTimeout(() => {
      console.log(
        "timer inside promise"
      );
    }, 0);
  });

setTimeout(() => {
  console.log(
    "outer timer"
  );
}, 0);
```

Typical output:

```text
promise
outer timer
timer inside promise
```

Why?

The outer timer was scheduled during the initial script.

The inner timer is scheduled later, from the Promise microtask.

---

# 10. queueMicrotask vs Promise

```js
queueMicrotask(() => {
  console.log("A");
});

Promise.resolve()
  .then(() => {
    console.log("B");
  });
```

Both are microtasks.

If queued in this order, output is typically:

```text
A
B
```

---

# 11. Promise Constructor vs Promise Handler

Important distinction:

```js
console.log("A");

new Promise((resolve) => {
  console.log("B");

  resolve();
}).then(() => {
  console.log("C");
});

console.log("D");
```

Output:

```text
A
B
D
C
```

Why?

The Promise **executor** runs synchronously.

The `.then()` handler runs as a microtask.

---

# 12. async/await Ordering

```js
async function test() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

test();

console.log("C");
```

Output:

```text
A
C
B
```

The continuation after `await` is scheduled asynchronously, effectively through Promise/microtask semantics.

---

# 13. async Function + Timer

```js
async function test() {
  console.log("1");

  await Promise.resolve();

  console.log("2");
}

setTimeout(() => {
  console.log("3");
}, 0);

test();

console.log("4");
```

Output:

```text
1
4
2
3
```

Trace:

```text
sync:
1
4

microtask:
resume async function → 2

task:
timer → 3
```

---

# 14. Nested Promise Microtasks

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    Promise.resolve()
      .then(() => {
        console.log("B");
      });
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Output:

```text
A
C
B
```

The nested Promise callback is queued only when A runs.

---

# 15. Timer Delay Does Not Control Queue Priority

Suppose:

```js
setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise");
  });
```

Even though the timer delay is zero, the Promise handler still runs first after the current script.

Priority is determined by queue category, not merely delay value.

---

# 16. Long Synchronous Code Delays Both

```js
setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise");
  });

const start = Date.now();

while (
  Date.now() - start < 3000
) {}
```

Nothing asynchronous runs until the blocking loop finishes.

Then:

```text
promise
timer
```

So:

```text
ready
≠
running
```

---

# 17. Microtasks Created by Microtasks

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    queueMicrotask(() => {
      console.log("B");
    });
  });

setTimeout(() => {
  console.log("C");
}, 0);
```

Output:

```text
A
B
C
```

Microtasks keep draining until the queue becomes empty.

---

# 18. Promise Chain + Timer Example

```js
console.log("start");

setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("p1");
  })
  .then(() => {
    console.log("p2");
  });

console.log("end");
```

Output:

```text
start
end
p1
p2
timer
```

---

# 19. Output-Tracing Method

For Promise/timer questions:

```text
Step 1
Write synchronous output

Step 2
List microtasks in exact queue order

Step 3
List timer/tasks in scheduling order

Step 4
Drain microtasks completely

Step 5
Run one task

Step 6
Add any new microtasks/tasks created

Step 7
Repeat
```

This is more reliable than memorizing examples.

---

# 20. Common Mistakes

## Mistake 1

```text
0ms timer = immediate
```

Wrong.

## Mistake 2

```text
Promise.then executes synchronously
```

Wrong.

## Mistake 3

```text
All Promise.then callbacks in a chain
are queued at once
```

Wrong.

Later handlers are queued only when prior chain links settle.

---

# 21. Browser vs Node Note

The rules in this lesson target the common browser event-loop model.

Node.js also uses microtasks and timers, but it has additional scheduling phases and details.

For frontend interviews and browser JavaScript, this mental model is the correct baseline.

---

# Interview Questions

### Why does a Promise callback usually run before setTimeout(..., 0)?

Because Promise reactions are microtasks, and microtasks are drained before the next task such as a timer callback.

### Does a zero-delay timer execute immediately?

No.

### Does the Promise executor run asynchronously?

No. The executor runs synchronously; handlers run asynchronously.

### What happens after a timer task schedules a Promise handler?

That microtask runs before the next task.

---

# Key Takeaways

- Promise handlers are microtasks.
- Timer callbacks are tasks/macrotasks.
- Synchronous code always finishes first.
- Microtasks are drained before the next task.
- Promise chains queue later handlers only after earlier links settle.
- A timer can create microtasks that run before other timer tasks.
- A Promise can schedule a timer that runs later than already queued timers.
- Timer delay does not override microtask priority.
