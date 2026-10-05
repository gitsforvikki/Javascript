# Lesson 73 — Microtasks vs Macrotasks

This lesson focuses on the most interview-heavy event-loop distinction:

- microtasks
- tasks/macrotasks

The core priority rule is:

> Finish synchronous code first, then drain all microtasks, then move to the next task.

---

## 1. Quick Classification

### Microtasks

Common examples:

```text
Promise.then
Promise.catch
Promise.finally
queueMicrotask
MutationObserver
```

### Tasks / Macrotasks

Common examples:

```text
setTimeout
setInterval
DOM events
message events
initial script
```

---

# 2. Basic Priority Example

```js
setTimeout(() => {
  console.log("timeout");
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
timeout
```

Reason:

```text
sync
↓
microtask
↓
task
```

---

# 3. 0ms Timer Does Not Beat a Promise

This is one of the most common interview questions.

```js
setTimeout(() => {
  console.log("A");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("B");
  });
```

Output:

```text
B
A
```

Even with zero delay, the timer is a task.

Promise reaction is a microtask.

Microtasks are processed first after the current task completes.

---

# 4. Multiple Microtasks

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

Both microtasks drain before the timer task.

---

# 5. Nested Microtask

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    Promise.resolve()
      .then(() => {
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

After A runs, it queues B.

The event loop keeps draining microtasks before moving to timer C.

---

# 6. Microtasks Are Not Recursive Function Calls

When a Promise handler queues another Promise handler, the second one does not run inside the same JavaScript call stack frame.

Instead:

```text
microtask A runs
↓
queues microtask B
↓
A finishes
↓
stack empty
↓
B runs as next microtask
```

That difference matters for stack behavior.

---

# 7. Chained Promise Interleaving

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

Why not A B C?

Initial microtask queue:

```text
A-handler
C-handler
```

After A finishes, B becomes queued:

```text
C-handler
B-handler
```

So output is:

```text
A
C
B
```

---

# 8. Task Creates Microtask

```js
setTimeout(() => {
  console.log("A");

  Promise.resolve()
    .then(() => {
      console.log("B");
    });
}, 0);

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

After first timer task A completes, its microtask B runs before the second timer task C.

---

# 9. Microtask Creates Task

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    setTimeout(() => {
      console.log("B");
    }, 0);
  });

setTimeout(() => {
  console.log("C");
}, 0);
```

Typical output:

```text
A
C
B
```

Why?

The timer for C was scheduled during the original script.

The timer for B is scheduled later from the microtask.

So C generally becomes the earlier eligible timer task.

---

# 10. queueMicrotask vs Promise.then

Both schedule microtasks.

```js
queueMicrotask(() => {
  console.log("A");
});

Promise.resolve()
  .then(() => {
    console.log("B");
  });
```

Typical output:

```text
A
B
```

because A was queued first.

---

# 11. Error Behavior Difference

There is a subtle difference between:

```js
queueMicrotask(() => {
  throw new Error("Boom");
});
```

and:

```js
Promise.resolve()
  .then(() => {
    throw new Error("Boom");
  });
```

The first creates an uncaught exception-style error in the microtask callback.

The second turns the thrown error into a rejected Promise.

So while both use microtask scheduling, their error channels differ.

---

# 12. Macrotask Terminology

You will often hear:

```text
macrotask queue
```

In browser specifications, the standard term is generally:

```text
task
```

For interviews:

```text
macrotask
≈ task
```

Use the terminology the interviewer uses, but understand the actual scheduling model.

---

# 13. Priority Does Not Mean "Microtasks Always Run First"

Important nuance.

Microtasks do not run before the currently executing synchronous code.

Correct order:

```text
current synchronous task
↓
microtasks
↓
next task
```

So this:

```js
Promise.resolve()
  .then(() => {
    console.log("microtask");
  });

console.log("sync");
```

outputs:

```text
sync
microtask
```

---

# 14. One Task at a Time

The event loop generally chooses one task, runs it to completion, then performs a microtask checkpoint.

So:

```text
task A
↓
microtasks
↓
task B
↓
microtasks
↓
task C
```

This repeating pattern is the basis of output tracing.

---

# 15. Rendering Can Be Delayed by Microtasks

Because the browser often gets rendering opportunities between tasks, a massive microtask chain can delay rendering.

Example:

```js
function loop() {
  queueMicrotask(loop);
}

loop();
```

This can monopolize the event loop.

So microtasks should remain small and finite.

---

# 16. Promise Chains and Rendering

A large Promise chain can also generate many microtasks.

That does not mean Promises are bad.

It means:

> high-priority work can still starve lower-priority work if abused.

---

# 17. Complex Example

Predict:

```js
console.log("1");

setTimeout(() => {
  console.log("2");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("3");

    setTimeout(() => {
      console.log("4");
    }, 0);
  })
  .then(() => {
    console.log("5");
  });

queueMicrotask(() => {
  console.log("6");
});

console.log("7");
```

Step 1 — synchronous:

```text
1
7
```

Initial microtasks:

```text
then(3)
queueMicrotask(6)
```

Tasks:

```text
timer(2)
```

Run microtask 3:

```text
prints 3
schedules timer 4
queues then(5)
```

Queue becomes:

```text
6
5
```

Run them:

```text
6
5
```

Then tasks:

```text
2
4
```

Final output:

```text
1
7
3
6
5
2
4
```

---

# 18. Another Interview Example

```js
console.log("A");

Promise.resolve()
  .then(() => {
    console.log("B");

    return Promise.resolve();
  })
  .then(() => {
    console.log("C");
  });

Promise.resolve()
  .then(() => {
    console.log("D");
  });

setTimeout(() => {
  console.log("E");
}, 0);

console.log("F");
```

Start:

```text
sync:
A
F
```

Initial microtasks:

```text
B-handler
D-handler
```

Run B.

Because B returns an already-fulfilled Promise, the chain continuation is still scheduled through Promise resolution steps rather than simply jumping ahead immediately.

For interview purposes, the safe rule is:

> trace Promise-chain continuation as newly queued microtask work, not as synchronous continuation.

Then D runs before C.

Typical output:

```text
A
F
B
D
C
E
```

---

# 19. Browser Event Example

Suppose a click handler runs as a task:

```js
button.addEventListener(
  "click",
  () => {
    console.log("click");

    Promise.resolve()
      .then(() => {
        console.log(
          "promise"
        );
      });

    setTimeout(() => {
      console.log(
        "timer"
      );
    }, 0);
  }
);
```

After the click handler finishes:

```text
promise
```

runs before the later timer task.

---

# 20. React Connection

React event handlers run within the browser scheduling environment.

Promises created inside handlers schedule microtasks.

Timers schedule later tasks.

Understanding this helps reason about:

- state updates after Promises
- delayed effects
- UI rendering timing
- stale closures
- async event flows

---

# 21. Interview Solving Framework

For every output question:

```text
1. Execute synchronous code

2. Write microtasks in order

3. Write tasks in order

4. Drain microtasks fully

5. Execute one task

6. Add any new microtasks/tasks created by it

7. Drain microtasks again

8. Repeat
```

This is much more reliable than intuition.

---

# 22. Quick Priority Table

| Type | Examples | When |
| --- | --- | --- |
| Synchronous | normal statements | immediately in current stack |
| Microtask | Promise handlers, queueMicrotask | after current task, before next task |
| Task/Macrotask | timers, events | one task at a time after microtasks |

---

# Interview Questions

### Why does Promise.then usually run before setTimeout(..., 0)?

Because Promise reactions are microtasks, while timer callbacks are tasks. Microtasks are drained before the next task begins.

### Can a microtask schedule another microtask?

Yes, and the queue keeps draining until empty.

### Are microtasks executed before synchronous code?

No. Current synchronous code finishes first.

### What is microtask starvation?

When microtasks continually schedule more microtasks and prevent the event loop from progressing to tasks or rendering.

---

# Key Takeaways

- Synchronous work comes first.
- Microtasks run after the current task.
- All available microtasks are drained before the next task.
- Timers are tasks/macrotasks.
- Promise handlers are microtasks.
- Promise chains can interleave because later handlers are queued later.
- A task can schedule microtasks that run before the next task.
- Microtasks can delay rendering if abused.
- Use queue classification and step-by-step tracing for interview problems.
