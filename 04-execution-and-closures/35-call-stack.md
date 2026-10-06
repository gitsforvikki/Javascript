# Lesson 35 — Call Stack

The Call Stack is the mechanism JavaScript uses to track active function execution.

It is one of the most important interview concepts because it connects:

- execution contexts
- function calls
- recursion
- synchronous execution
- stack overflow
- asynchronous callbacks
- event loop

The core idea:

> The call stack stores active execution contexts in Last-In, First-Out order.

---

## 1. Stack Data Structure

A stack follows:

```text
LIFO
=
Last In
First Out
```

Imagine plates:

```text
push plate
push plate
push plate

remove
↓
top plate first
```

JavaScript function execution follows the same idea.

---

## 2. Global Execution Context Enters First

When a script starts:

```text
Call Stack

┌──────────────────────┐
│ Global Context       │
└──────────────────────┘
```

Global code begins executing.

---

## 3. Function Call Pushes a Context

```js
function greet() {
  console.log(
    "Hello"
  );
}

greet();
```

When `greet()` is called:

```text
Call Stack

┌──────────────────────┐
│ greet Context        │
├──────────────────────┤
│ Global Context       │
└──────────────────────┘
```

---

## 4. Function Return Pops the Context

After `greet()` finishes:

```text
Call Stack

┌──────────────────────┐
│ Global Context       │
└──────────────────────┘
```

The function execution context is removed.

---

## 5. Nested Function Calls

```js
function one() {
  two();
}

function two() {
  three();
}

function three() {
  console.log(
    "Done"
  );
}

one();
```

Flow:

```text
Global
↓
one()
↓
two()
↓
three()
```

Stack at deepest point:

```text
┌──────────────────────┐
│ three Context        │
├──────────────────────┤
│ two Context          │
├──────────────────────┤
│ one Context          │
├──────────────────────┤
│ Global Context       │
└──────────────────────┘
```

---

## 6. Pop Order

When `three()` returns:

```text
three pops
↓
two continues
```

Then:

```text
two pops
↓
one continues
```

Then:

```text
one pops
↓
global continues
```

LIFO order.

---

## 7. Output Trace Example

```js
function first() {
  console.log("A");
  second();
  console.log("B");
}

function second() {
  console.log("C");
}

first();
```

Output:

```text
A
C
B
```

Why?

```text
first starts
↓
prints A
↓
second pushed
↓
prints C
↓
second pops
↓
first resumes
↓
prints B
```

---

## 8. Execution Context vs Call Stack

Do not confuse them.

```text
Execution Context
→ runtime state for one execution

Call Stack
→ structure that stores active contexts
```

Example:

```text
one Context
two Context
three Context
```

are separate execution contexts.

The call stack holds them.

---

## 9. Synchronous Execution

JavaScript executes the current top stack frame before moving to the next pending task.

Example:

```js
function slow() {
  for (
    let i = 0;
    i < 1_000_000_000;
    i++
  ) {}
}

slow();

console.log(
  "after"
);
```

`console.log("after")` waits until `slow()` returns.

This is synchronous blocking.

---

## 10. Recursion

```js
function countdown(
  n
) {
  if (
    n === 0
  ) {
    return;
  }

  countdown(
    n - 1
  );
}

countdown(3);
```

Stack builds:

```text
countdown(3)
countdown(2)
countdown(1)
countdown(0)
```

Each call gets a separate execution context.

---

## 11. Stack Overflow

If recursion never stops:

```js
function recurse() {
  recurse();
}

recurse();
```

New execution contexts keep getting pushed.

Eventually:

```text
Maximum call stack size exceeded
```

or a similar `RangeError`.

---

## 12. Why Stack Size Is Limited

The runtime reserves finite memory for call-stack execution.

Infinite or extremely deep recursion consumes too many stack frames.

So recursion needs:

- base case
- progress toward base case

---

## 13. Error Stack Trace

When an error occurs:

```js
function a() {
  b();
}

function b() {
  c();
}

function c() {
  throw new Error(
    "Failed"
  );
}

a();
```

The error stack trace often shows:

```text
c
b
a
global
```

This reflects the chain of active calls.

---

## 14. Call Stack and setTimeout

```js
console.log(
  "start"
);

setTimeout(
  () => {
    console.log(
      "timer"
    );
  },
  0
);

console.log(
  "end"
);
```

Output:

```text
start
end
timer
```

Why?

The timer callback does not immediately enter the call stack.

---

## 15. Async Mental Model

Conceptually:

```text
Global code
↓
setTimeout registered with runtime
↓
global code continues
↓
current stack becomes empty
↓
timer callback becomes eligible
↓
event loop schedules callback
↓
callback execution context pushed
```

This is the bridge between the call stack and event loop.

---

## 16. Zero-Millisecond Timer Does Not Mean Immediate

```js
setTimeout(
  callback,
  0
);
```

means roughly:

> schedule the callback after the minimum delay, when the runtime/event loop is able to run it.

It does not mean:

```text
run now
```

The current call stack must finish first.

---

## 17. Promise Microtasks

Example:

```js
console.log("A");

Promise.resolve()
  .then(
    () => {
      console.log("B");
    }
  );

console.log("C");
```

Output:

```text
A
C
B
```

The `.then` callback runs after current synchronous stack execution finishes.

Microtask ordering is covered later in the Event Loop section.

---

## 18. Long Tasks and Call Stack

If one function occupies the call stack for too long:

```js
function heavyWork() {
  // CPU-heavy loop
}
```

the browser cannot quickly process:

- clicks
- keyboard input
- timers
- rendering opportunities

This is called main-thread blocking.

---

## 19. Stack Frame Mental Model

Each function call frame conceptually contains information such as:

- local execution state
- parameters
- return location
- execution context information

Do not assume a JavaScript-visible "stack frame object" exists.

This is a conceptual/runtime model.

---

## 20. Closure Does Not Keep Stack Frame Active

Example:

```js
function outer() {
  let count = 0;

  return function () {
    return ++count;
  };
}

const counter =
  outer();
```

After `outer()` returns:

```text
outer execution context
→ removed from call stack
```

But `count` remains reachable through the closure's lexical environment.

This is one of the most important interview distinctions.

---

## 21. Call Stack vs Scope Chain

These are different.

### Call Stack

Tracks:

```text
who called whom
```

### Scope Chain

Tracks:

```text
where variables are searched lexically
```

A function's caller is not automatically its lexical parent.

---

## 22. Example Proving the Difference

```js
const value =
  "global";

function show() {
  console.log(
    value
  );
}

function caller() {
  const value =
    "caller";

  show();
}

caller();
```

Output:

```text
global
```

Call stack:

```text
global
↓
caller
↓
show
```

But scope lookup for `show` follows where `show` was defined, not who called it.

---

## 23. React Connection

A React component render is still a normal JavaScript function call.

Nested helper functions add more stack frames.

But state persistence across renders does not happen because old execution contexts remain on the stack.

React stores state externally to the local function execution context.

---

## 24. Debugger Connection

Browser DevTools often shows the call stack during a breakpoint.

Example:

```text
handleClick
saveUser
request
anonymous
```

Reading the stack tells you the path that led to the current line.

This is essential for debugging.

---

## 25. Interview Trace Problem

Code:

```js
function a() {
  console.log("1");
  b();
  console.log("2");
}

function b() {
  console.log("3");
  c();
}

function c() {
  console.log("4");
}

a();
```

Output:

```text
1
3
4
2
```

Reason:

```text
a
↓
b
↓
c
↑
b finishes
↑
a resumes
```

---

## 26. Interview Questions

### What is the call stack?

A LIFO structure used to track active JavaScript execution contexts/function calls.

### What happens when a function is called?

A new function execution context is created and pushed onto the call stack.

### What happens when it returns?

Its context is popped from the stack.

### What is stack overflow?

Too many nested active calls, often from uncontrolled recursion.

### Does setTimeout callback enter stack immediately?

No.

---

## Interview Answer — 30 Seconds

> The call stack is a LIFO structure that tracks active execution contexts. Global execution starts at the bottom, and each function call pushes a new function execution context. When that function returns, its context is popped and execution resumes in the previous frame. Long-running synchronous work blocks the stack, while asynchronous callbacks can only execute later when scheduling rules allow them onto the stack.

---

## Key Takeaways

- Call stack uses LIFO order.
- Every function call creates/pushes a function execution context.
- Returning pops that context.
- Recursion creates multiple stack frames.
- Infinite recursion causes stack overflow.
- Async callbacks do not run while synchronous stack work is active.
- Call stack and scope chain are different concepts.
- Closures do not keep old execution contexts on the stack.
