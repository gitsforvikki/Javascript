# Lesson 71 — Event Loop Mental Model

The event loop is one of the most important JavaScript concepts for understanding:

- asynchronous execution
- timers
- Promises
- user events
- network callbacks
- rendering behavior
- interview output questions

The goal of this lesson is to build a **correct mental model** before memorizing execution orders.

---

## 1. Start with the Core Problem

JavaScript executes ordinary code using a call stack.

But applications also use asynchronous operations such as:

```js
setTimeout(...)
fetch(...)
Promise.then(...)
addEventListener(...)
```

If JavaScript executes only one call stack at a time, how can all these things work?

The answer involves:

```text
JavaScript Call Stack
+
Runtime APIs
+
Queues
+
Event Loop
```

---

# 2. JavaScript Is Single-Threaded at the Main Call Stack

A useful starting statement:

> Main-thread JavaScript executes one call stack at a time.

Example:

```js
console.log("A");
console.log("B");
console.log("C");
```

Output:

```text
A
B
C
```

Nothing else can interrupt ordinary synchronous JavaScript in the middle of the current stack.

---

# 3. The Call Stack

Example:

```js
function second() {
  console.log("second");
}

function first() {
  second();
}

first();
```

Stack conceptually:

```text
first()
  ↓
second()
  ↓
console.log()
```

As functions return, they are removed from the stack.

---

# 4. Runtime APIs Handle Asynchronous Waiting

Consider:

```js
setTimeout(() => {
  console.log("timer");
}, 1000);
```

The JavaScript call stack does not sit and wait for one second.

Instead:

```text
JavaScript
↓
calls timer API
↓
runtime tracks timer
↓
JavaScript continues
```

When the timer becomes eligible, its callback is scheduled for later execution.

---

# 5. The Event Loop's Job

A useful mental model:

> The event loop coordinates when queued asynchronous JavaScript can run on the call stack.

It does **not** execute your callback itself.

It helps move eligible work into JavaScript execution when the call stack is available and scheduling rules allow it.

---

# 6. Basic Event Loop Diagram

```text
┌──────────────────────┐
│ JavaScript Call Stack│
└──────────┬───────────┘
           │
           │ async API call
           ↓
┌──────────────────────┐
│ Runtime / Web APIs   │
│ timers, network, DOM │
└──────────┬───────────┘
           │
           │ completion
           ↓
┌──────────────────────┐
│ Queues               │
│ tasks / microtasks   │
└──────────┬───────────┘
           │
           ↓
      Event Loop
           │
           ↓
      Call Stack
```

This is simplified but directionally correct.

---

# 7. First Example

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

Trace:

```text
1. console.log("A")
2. setTimeout registers callback
3. console.log("C")
4. current script finishes
5. timer callback becomes eligible
6. callback runs later
```

Even with `0`, the timer callback cannot interrupt the current script.

---

# 8. Current Script Is Also Work

When JavaScript begins executing a script, that synchronous script itself is running as a task.

Conceptually:

```text
current script task
↓
run all synchronous code
↓
stack becomes empty
↓
process queued work
```

This explains why timers do not jump ahead of remaining synchronous statements.

---

# 9. Promise Example

```js
console.log("A");

Promise.resolve()
  .then(() => {
    console.log("B");
  });

console.log("C");
```

Output:

```text
A
C
B
```

The Promise handler is asynchronous too.

But Promise handlers use the **microtask queue**, which has different scheduling priority from timer tasks.

That distinction comes in Lessons 72–73.

---

# 10. Timer + Promise Example

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

Typical output:

```text
A
D
C
B
```

Why?

High-level rule:

```text
1. synchronous code first
2. microtasks
3. next task/macrotask
```

We will break this down carefully next.

---

# 11. Event Loop Does Not Mean "Everything Async Goes Into One Queue"

This is a common oversimplification.

There are different scheduling categories.

Important ones:

- task queue
- microtask queue

Promise reactions do not use the same queue as timers.

So avoid thinking:

```text
all callbacks
↓
one queue
```

That model is too weak for real output questions.

---

# 12. Rendering Fits Into the Browser Loop

In browsers, rendering can happen between tasks.

Simplified:

```text
task runs
↓
microtasks drain
↓
browser may render
↓
next task
```

This is why long-running JavaScript can delay painting.

---

# 13. Long Synchronous Work Blocks Everything

```js
setTimeout(() => {
  console.log("timer");
}, 0);

const start = Date.now();

while (
  Date.now() - start < 5000
) {
  // block
}
```

The timer may be ready almost immediately.

But it cannot execute while synchronous code is still blocking the call stack.

So:

```text
ready
≠
running
```

---

# 14. Ready vs Running

This distinction is extremely important.

An async callback can be:

```text
not ready
↓
ready
↓
queued
↓
waiting
↓
running
```

Being ready does not mean immediate execution.

It must still wait for scheduling rules and stack availability.

---

# 15. Event Loop Does Not Interrupt Running JavaScript

Suppose:

```js
function heavy() {
  // long synchronous work
}

setTimeout(() => {
  console.log("timer");
}, 0);

heavy();
```

The event loop does not stop `heavy()` halfway and run the timer callback.

JavaScript is run-to-completion for the current stack.

---

# 16. Run-to-Completion

A task runs until its current JavaScript stack is finished.

Conceptually:

```text
start task
↓
execute JS
↓
finish current stack
↓
only then switch to other queued work
```

This property makes synchronous reasoning easier.

---

# 17. Browser Events

Example:

```js
button.addEventListener(
  "click",
  () => {
    console.log("clicked");
  }
);
```

The click callback does not run until the user clicks.

Then the browser schedules event-handler work.

So:

```text
register handler
↓
wait for user event
↓
event occurs
↓
handler becomes scheduled
↓
event loop eventually runs it
```

---

# 18. Network Requests

```js
fetch("/api/users")
  .then((response) => {
    console.log(response);
  });
```

Network activity happens outside the active JavaScript call stack.

When the Promise settles, its reaction handler is scheduled as a microtask.

---

# 19. Event Loop Is a Scheduling Model

The event loop is not just about timers.

It coordinates JavaScript work from many sources:

- script execution
- timers
- user events
- network completion
- Promise reactions
- message events
- rendering opportunities

Different environments may have different detailed implementations, but the scheduling concepts remain essential.

---

# 20. Browser vs Node.js

Browsers and Node.js both have event loops, but their detailed scheduling phases are not identical.

For this section, the mental model focuses primarily on browser-style JavaScript scheduling:

```text
task
↓
microtasks
↓
possible rendering
↓
next task
```

Node.js has additional phase details that can differ.

Do not assume every browser output rule maps perfectly to Node internals.

---

# 21. A Better Event Loop Algorithm

For learning purposes, use this simplified cycle:

```text
1. Take one task

2. Run it until the stack is empty

3. Drain the microtask queue completely

4. Browser may render

5. Take the next task

6. Repeat
```

This model will explain most common browser interview questions.

---

# 22. Important Consequence

Suppose one task schedules several Promise callbacks.

The event loop will usually drain all currently available microtasks before moving to the next timer task.

That means Promise chains can run before pending timers.

---

# 23. Example with Nested Promise

```js
setTimeout(() => {
  console.log("timer");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise 1");

    return Promise.resolve();
  })
  .then(() => {
    console.log("promise 2");
  });
```

Typical output:

```text
promise 1
promise 2
timer
```

Because microtasks are drained before the next task.

---

# 24. Event Loop and UI Responsiveness

If JavaScript keeps scheduling heavy work without yielding, the browser may struggle to:

- paint
- respond to input
- animate smoothly

This is why task scheduling matters for performance.

---

# 25. React Connection

React applications rely on this scheduling environment.

You will see event-loop behavior in:

- event handlers
- state updates
- timers
- Promise callbacks
- data fetching
- effects
- transitions
- browser rendering

Understanding the event loop helps explain why code does not always run in the visual order you expect.

---

# 26. Interview Tracing Method

When solving an output question:

```text
Step 1
Run all synchronous code

Step 2
Record scheduled microtasks

Step 3
Record scheduled tasks

Step 4
After current stack finishes:
drain microtasks

Step 5
Take next task

Step 6
Repeat
```

Do not guess from delay values alone.

---

# 27. Common Mistakes

## Mistake 1

```text
setTimeout(..., 0)
means immediate
```

Wrong.

It still waits until current synchronous work and higher-priority microtasks are done.

## Mistake 2

```text
Promise.then runs synchronously
```

Wrong.

Its handler is scheduled as a microtask.

## Mistake 3

```text
event loop executes callbacks itself
```

Too simplistic.

The event loop coordinates scheduling; JavaScript still executes callbacks on the call stack.

---

# Interview Questions

### What is the event loop?

A scheduling mechanism that coordinates when queued asynchronous JavaScript work can run on the call stack.

### Can an async callback interrupt currently running JavaScript?

No. Current JavaScript runs to completion before queued callbacks execute.

### What generally runs first: synchronous code or queued async callbacks?

Synchronous code in the current stack runs first.

---

# Key Takeaways

- JavaScript executes one main call stack at a time.
- Runtime APIs handle asynchronous waiting.
- Ready callbacks are queued for later execution.
- The event loop coordinates when queued work can run.
- Current synchronous JavaScript runs to completion.
- Promise handlers and timers do not use the same queue.
- Microtasks are drained before the next task.
- Browser rendering can occur between tasks.
- Long synchronous work blocks timers, input, and rendering.
