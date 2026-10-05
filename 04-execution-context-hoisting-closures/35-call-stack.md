# Lesson 35 — Call Stack

The **call stack** is the mechanism JavaScript uses to track active function calls and determine what code should continue when a function finishes.

It is fundamental to understanding:

- execution contexts
- nested function calls
- recursion
- stack overflow
- asynchronous JavaScript
- event loop behavior

---

# 1. What is a Stack?

A stack follows:

> **LIFO — Last In, First Out**

Think of a stack of plates.

```text
Add plate C
Add plate B
Add plate A

Top
 ↓
┌───┐
│ A │ ← removed first
├───┤
│ B │
├───┤
│ C │
└───┘
```

The last item added is the first one removed.

The JavaScript call stack follows this general behavior for active calls.

---

# 2. What Does the Call Stack Track?

When code begins executing, the relevant global/script execution is active.

When a function is called, its execution context is pushed onto the stack.

When the function returns or completes, its context is popped.

Simplified:

```text
Call function
     ↓
Push execution context
     ↓
Execute function
     ↓
Function completes
     ↓
Pop execution context
```

---

# 3. Simple Example

```js
function greet() {
  console.log("Hello");
}

greet();
```

Simplified stack flow:

```text
1. Start global code

┌──────────────────┐
│ Global Execution │
└──────────────────┘

2. greet() called

┌──────────────────┐
│ greet()          │ ← top
├──────────────────┤
│ Global Execution │
└──────────────────┘

3. greet() finishes

┌──────────────────┐
│ Global Execution │
└──────────────────┘

4. Global code completes

Call stack becomes empty
```

---

# 4. Nested Function Calls

Consider:

```js
function first() {
  second();
}

function second() {
  third();
}

function third() {
  console.log("Done");
}

first();
```

Execution:

```text
Global
  ↓
first()
  ↓
second()
  ↓
third()
```

At the deepest point:

```text
Top
 ↓
┌──────────┐
│ third()  │
├──────────┤
│ second() │
├──────────┤
│ first()  │
├──────────┤
│ Global   │
└──────────┘
```

Then functions finish in reverse order:

```text
third() completes
      ↓
second() continues and completes
      ↓
first() continues and completes
      ↓
global execution continues
```

This is LIFO behavior.

---

# 5. Trace the Output

```js
function first() {
  console.log("First start");

  second();

  console.log("First end");
}

function second() {
  console.log("Second");
}

console.log("Global start");

first();

console.log("Global end");
```

Output:

```text
Global start
First start
Second
First end
Global end
```

Why?

```text
Global
│
├── print "Global start"
│
├── call first()
│      │
│      ├── print "First start"
│      │
│      ├── call second()
│      │      └── print "Second"
│      │
│      └── print "First end"
│
└── print "Global end"
```

JavaScript must finish the current synchronous call before returning to its caller.

---

# 6. Execution Context + Call Stack

These concepts connect directly.

```text
Execution Context
= environment for executing a piece of code

Call Stack
= structure tracking active execution contexts
```

Example:

```js
function multiply(a, b) {
  return a * b;
}

function calculate() {
  return multiply(5, 10);
}

calculate();
```

At the deepest point:

```text
Call Stack
│
├── multiply context
├── calculate context
└── global context
```

When `multiply` returns:

```text
multiply popped
      ↓
calculate resumes
```

---

# 7. Recursion and the Call Stack

A recursive function calls itself.

```js
function countdown(number) {
  if (number === 0) {
    return;
  }

  console.log(number);

  countdown(number - 1);
}

countdown(3);
```

Calls:

```text
countdown(3)
    ↓
countdown(2)
    ↓
countdown(1)
    ↓
countdown(0)
```

At the deepest point, multiple invocations of the same function are active:

```text
┌──────────────┐
│ countdown(0) │
├──────────────┤
│ countdown(1) │
├──────────────┤
│ countdown(2) │
├──────────────┤
│ countdown(3) │
├──────────────┤
│ Global       │
└──────────────┘
```

Each invocation has its own execution context.

---

# 8. Stack Overflow

The call stack has finite capacity.

Consider infinite recursion:

```js
function run() {
  run();
}

run();
```

Calls keep being added:

```text
run()
 ↓
run()
 ↓
run()
 ↓
run()
 ↓
...
```

Eventually the engine cannot keep adding stack frames and throws an error commonly shown as:

```text
RangeError: Maximum call stack size exceeded
```

This is called a **stack overflow**.

---

# 9. Long Synchronous Work Blocks the Stack

Consider:

```js
console.log("Start");

for (let i = 0; i < 1_000_000_000; i++) {
  // expensive synchronous work
}

console.log("End");
```

While that loop is executing, the current JavaScript execution remains busy.

In a browser's main thread, expensive synchronous work can make the UI unresponsive.

This leads to an important rule:

> JavaScript cannot execute another ordinary callback on the same call stack until the currently running synchronous work yields/completes.

This becomes crucial when studying the event loop.

---

# 10. Call Stack and Asynchronous Code — Preview

Consider:

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

Why doesn't `B` run immediately even though the delay is zero?

Because the callback is scheduled through the runtime and cannot execute on the JavaScript call stack until the current synchronous execution has completed and scheduling rules allow it.

Later we will add:

- task queue
- microtask queue
- event loop

to this model.

For now:

```text
Call Stack
    ↓
runs current synchronous JavaScript
    ↓
must become available before scheduled callbacks execute
```

---

# 11. Reading Stack Traces

Errors often expose the chain of function calls.

Example:

```js
function first() {
  second();
}

function second() {
  third();
}

function third() {
  throw new Error("Something failed");
}

first();
```

A stack trace helps show that execution reached:

```text
first
  ↓
second
  ↓
third
  ↓
Error
```

Learning the call stack makes debugging stack traces much easier.

---

# 12. React Connection

React event handlers are JavaScript function calls.

```jsx
function handleClick() {
  validate();
  saveData();
}
```

If `validate()` performs expensive synchronous work, it occupies the call stack before `saveData()` or other scheduled work can proceed.

Understanding the call stack helps explain:

- blocking UI work
- event handlers
- synchronous rendering work
- async callbacks
- promise ordering

---

# 13. Interview Perspective

**What is the call stack?**

The call stack is a LIFO structure used to track active JavaScript function executions and where execution should return when functions complete.

**What happens when a function is called?**

A new function execution context/frame becomes active on the call stack. When the function completes, it is removed and execution returns to its caller.

**What causes maximum call stack size exceeded?**

Usually excessively deep or infinite recursive/nested calls that exhaust available stack capacity.

---

# Key Takeaways

- The call stack tracks active function executions.
- It follows LIFO behavior.
- Function calls are pushed onto the stack.
- Completed calls are popped from the stack.
- Each function invocation has its own execution context.
- Recursion creates multiple active calls.
- Infinite recursion can cause stack overflow.
- Long synchronous work blocks further JavaScript execution on that stack.
- The call stack is essential for understanding the event loop.
