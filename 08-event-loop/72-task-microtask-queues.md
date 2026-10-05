# Lesson 72 — Task Queue and Microtask Queue

The event loop becomes much easier to understand once you separate two major scheduling queues:

- **task queue**
- **microtask queue**

These queues do not have equal priority.

The most important rule is:

> After the current task finishes, JavaScript drains the microtask queue before moving to the next task.

---

## 1. What Is a Task?

A task is a larger unit of scheduled work.

Common browser task sources include:

- initial script execution
- `setTimeout`
- `setInterval`
- user events
- message events
- some I/O/event callbacks

People often call these **macrotasks**, though the HTML specification generally uses the term **task**.

---

# 2. What Is a Microtask?

Microtasks are higher-priority follow-up jobs that run after the current task but before the next task.

Common sources:

- Promise `.then()`
- Promise `.catch()`
- Promise `.finally()`
- `queueMicrotask()`
- MutationObserver callbacks

---

# 3. Basic Scheduling Rule

Simplified:

```text
current task
↓
run all synchronous code
↓
microtask checkpoint
↓
drain ALL microtasks
↓
next task
```

This is the most important queue rule.

---

# 4. Basic Example

```js
setTimeout(() => {
  console.log("task");
}, 0);

Promise.resolve()
  .then(() => {
    console.log(
      "microtask"
    );
  });
```

Typical output:

```text
microtask
task
```

Why?

Both are scheduled during the current script.

After the script finishes:

1. microtasks are drained
2. timer task runs later

---

# 5. Add Synchronous Code

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("C");
  });

console.log("D");
```

Output:

```text
A
D
C
B
```

Categorize first:

```text
sync:
A
D

microtask:
C

task:
B
```

Then execute by priority.

---

# 6. Current Script Is a Task

The initial script itself runs as a task.

So:

```text
script task
├── synchronous statements
├── schedules microtasks
└── schedules future tasks
```

When the script task completes, the event loop performs a microtask checkpoint.

---

# 7. queueMicrotask()

You can explicitly schedule a microtask:

```js
queueMicrotask(() => {
  console.log(
    "microtask"
  );
});
```

Compare:

```js
setTimeout(() => {
  console.log("task");
}, 0);
```

Typical output:

```text
microtask
task
```

---

# 8. Microtask Queue Is FIFO

Microtasks are generally processed in the order they are queued.

Example:

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  });

queueMicrotask(() => {
  console.log("B");
});

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Typical output:

```text
A
B
C
```

They were queued in that order.

---

# 9. Microtasks Can Queue More Microtasks

This is crucial.

```js
Promise.resolve()
  .then(() => {
    console.log("A");

    queueMicrotask(() => {
      console.log("B");
    });
  });

Promise.resolve()
  .then(() => {
    console.log("C");
  });
```

Initial microtask queue:

```text
A-handler
C-handler
```

Run A-handler:

```text
prints A
queues B
```

Queue becomes:

```text
C-handler
B
```

Final output:

```text
A
C
B
```

---

# 10. Drain Means Until Empty

When the event loop starts processing microtasks, it keeps going until the microtask queue becomes empty.

That includes microtasks scheduled by other microtasks.

This is why microtasks can delay the next task.

---

# 11. Microtask Starvation

Consider:

```js
function repeat() {
  queueMicrotask(repeat);
}

repeat();
```

This continually schedules another microtask.

The queue may never become empty.

As a result, the browser may be unable to move on to:

- timer tasks
- user events
- rendering

This is called **microtask starvation**.

---

# 12. Task Queue Example

```js
setTimeout(() => {
  console.log("timer 1");
}, 0);

setTimeout(() => {
  console.log("timer 2");
}, 0);
```

The timers become tasks.

They generally execute in scheduling order when eligible, though exact timer timing can vary.

---

# 13. A Task Can Schedule a Microtask

```js
setTimeout(() => {
  console.log("task 1");

  Promise.resolve()
    .then(() => {
      console.log(
        "microtask inside task 1"
      );
    });
}, 0);

setTimeout(() => {
  console.log("task 2");
}, 0);
```

Typical output:

```text
task 1
microtask inside task 1
task 2
```

Why?

After task 1 finishes, microtasks are drained before task 2 begins.

---

# 14. Task-by-Task Mental Model

```text
Task 1
↓
run sync code
↓
drain microtasks
↓
possible render
↓
Task 2
↓
run sync code
↓
drain microtasks
↓
...
```

Microtasks happen between tasks.

---

# 15. Promise Chaining and the Microtask Queue

```js
Promise.resolve()
  .then(() => {
    console.log("A");
  })
  .then(() => {
    console.log("B");
  });
```

Important:

The second `.then()` handler is not queued at the exact same initial moment as the first.

Flow:

```text
first then queued
↓
first then runs
↓
its returned Promise fulfills
↓
second then becomes queued
↓
second then runs
```

This matters in complex ordering questions.

---

# 16. Interleaving Example

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

Initial queue:

```text
A-handler
C-handler
```

Run A:

```text
prints A
queues B-handler later
```

Queue:

```text
C-handler
B-handler
```

Output:

```text
A
C
B
```

This is a very common interview pattern.

---

# 17. MutationObserver

Browser `MutationObserver` callbacks are also processed as microtasks.

Example conceptually:

```text
DOM mutation
↓
observer callback scheduled
↓
microtask queue
```

You do not need to memorize every API, but know that Promise handlers are not the only microtasks.

---

# 18. Rendering and Queues

A simplified browser cycle:

```text
task
↓
microtasks drained
↓
browser may render
↓
next task
```

This means a very long microtask chain can delay rendering.

---

# 19. Task Queue Is Not Literally One Universal Queue

Browsers may have multiple task sources/queues and choose among them according to specification/runtime rules.

For learning and interviews, it is still useful to think:

```text
tasks
vs
microtasks
```

But avoid assuming there is always exactly one physical "macrotask queue."

---

# 20. Naming: Task vs Macrotask

You will often hear:

```text
macrotask
```

in interviews and tutorials.

Examples:

- timer callback
- DOM event callback

More specification-aligned terminology is:

```text
task
```

For interview purposes, understand both terms.

---

# 21. React / Frontend Connection

Microtasks and tasks affect:

- Promise-based state flows
- event handlers
- timers
- rendering opportunities
- UI responsiveness

Knowing queue priority helps explain why a Promise callback may run before a zero-delay timer.

---

# 22. Output-Tracing Strategy

For code like:

```js
console.log("A");

setTimeout(...);

Promise.resolve()
  .then(...);

queueMicrotask(...);

console.log("B");
```

Create buckets:

```text
SYNC
A
B

MICROTASKS
Promise handler
queueMicrotask callback

TASKS
timer callback
```

Then execute:

```text
sync
↓
all microtasks
↓
next task
```

---

# Interview Questions

### What is the difference between a task and a microtask?

Tasks represent larger scheduled work such as timers/events, while microtasks are higher-priority follow-up jobs such as Promise reactions.

### When are microtasks processed?

After the current task finishes and before the next task begins.

### Can microtasks schedule more microtasks?

Yes, and the queue continues draining until empty.

---

# Key Takeaways

- Promise handlers use the microtask queue.
- Timers and many events are tasks.
- Synchronous code finishes first.
- After each task, the microtask queue is drained completely.
- Microtasks can schedule more microtasks.
- A long microtask chain can delay timers and rendering.
- Promise chains may interleave because later handlers are queued only after earlier handlers settle.
- "Task" is the more specification-aligned term; "macrotask" is common interview terminology.
