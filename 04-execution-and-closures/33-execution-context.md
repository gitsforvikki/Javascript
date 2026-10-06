# Lesson 33 — Execution Context

Execution Context is one of the most important JavaScript interview concepts.

If you understand execution context, many topics become much easier:

- hoisting
- scope
- lexical environment
- call stack
- closures
- `this`
- function execution

The core idea:

> An execution context is the environment JavaScript creates to execute a piece of code.

---

## 1. Why JavaScript Needs an Execution Context

Suppose JavaScript runs:

```js
const name =
  "Vikash";

function greet() {
  const message =
    "Hello";

  console.log(
    message,
    name
  );
}

greet();
```

JavaScript needs to track:

- variables
- function declarations
- lexical scope
- current `this`
- where to search for outer variables

That execution information belongs to an execution context.

---

## 2. Main Types of Execution Context

For interview purposes, focus on:

1. **Global Execution Context**
2. **Function Execution Context**
3. **Eval Execution Context** — rarely important in modern application code

Modules also have module-specific execution semantics, but global/function contexts are the core interview model.

---

## 3. Global Execution Context

When a script begins, JavaScript creates a global execution context.

Example:

```js
const a = 10;

function test() {
  console.log(a);
}
```

Before calling `test()`, global code runs inside the global execution context.

Conceptually:

```text
Global Execution Context
│
├── a
├── test
├── lexical environment
└── this binding
```

---

## 4. Function Execution Context

Every time a normal function is invoked, a new function execution context is created.

```js
function add(
  a,
  b
) {
  const result =
    a + b;

  return result;
}

add(2, 3);
```

Calling `add()` creates a new execution context containing:

- parameters
- local variables
- function declarations
- lexical environment
- `this` binding for ordinary functions

---

## 5. Every Call Gets a New Context

```js
function greet(
  name
) {
  const message =
    `Hello ${name}`;

  return message;
}

greet(
  "Vikash"
);

greet(
  "Rahul"
);
```

These are separate calls.

Conceptually:

```text
Call 1
↓
Execution Context A
name = "Vikash"

Call 2
↓
Execution Context B
name = "Rahul"
```

The local variables are independent.

---

## 6. Execution Context Is Not the Same as Scope

This distinction is important.

### Scope

Describes where variables are accessible based on source-code structure.

### Execution Context

Represents a runtime instance created while code executes.

Example:

```js
function test() {
  const value = 10;
}
```

The function has a lexical scope defined by where it was written.

But every call to `test()` creates a new execution context.

---

## 7. Execution Context Is Not the Same as Call Stack

Another common confusion.

```text
Execution Context
→ execution environment/state

Call Stack
→ data structure that tracks active execution contexts
```

Think:

```text
execution context
= frame

call stack
= stack holding frames
```

---

## 8. Conceptual Structure of an Execution Context

A simplified mental model:

```text
Execution Context
│
├── Lexical Environment
│   ├── local bindings
│   └── outer reference
│
├── Variable Environment
│
└── this binding
```

Exact specification terminology evolves, but this model is useful for interviews.

---

## 9. Lexical Environment

The lexical environment stores bindings and knows where the outer lexical environment is.

Conceptually:

```text
current variables
+
reference to outer scope
```

This is how JavaScript resolves variables through the scope chain.

---

## 10. Example of Outer Lookup

```js
const globalValue =
  "global";

function outer() {
  const outerValue =
    "outer";

  function inner() {
    const innerValue =
      "inner";

    console.log(
      innerValue,
      outerValue,
      globalValue
    );
  }

  inner();
}

outer();
```

Inside `inner()`:

```text
look locally
↓
look in outer lexical environment
↓
look in global lexical environment
```

This is lexical scope resolution.

---

## 11. Function Context and Parameters

```js
function multiply(
  a,
  b
) {
  const result =
    a * b;

  return result;
}
```

The function execution context contains bindings for:

```text
a
b
result
```

Parameters behave like local bindings initialized from arguments.

---

## 12. arguments Object

Traditional functions may also expose:

```js
arguments
```

Example:

```js
function show() {
  console.log(
    arguments
  );
}
```

Arrow functions do not create their own `arguments` object.

This is related to function execution semantics.

---

## 13. this Binding

For ordinary functions, execution context includes a `this` binding determined by how the function is called.

Example:

```js
const user = {
  name: "Vikash",

  show() {
    console.log(
      this.name
    );
  },
};

user.show();
```

During this call:

```text
this → user
```

Arrow functions behave differently because they do not create their own `this` binding.

---

## 14. Recursive Calls Create Multiple Contexts

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

Contexts:

```text
countdown(3)
countdown(2)
countdown(1)
countdown(0)
```

Each recursive call has its own execution context.

---

## 15. Execution Context Lifecycle

A simplified lifecycle:

```text
Function called
↓
Execution Context created
↓
Creation phase
↓
Execution phase
↓
Function returns
↓
Context removed from call stack
```

The next lesson explains the two phases in detail.

---

## 16. What Happens After Function Returns?

Normally, its execution context is removed from the call stack.

But this does **not** mean all local data is always immediately destroyed.

If a closure still references a lexical binding, that binding can remain reachable.

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

The outer function context is no longer on the call stack.

But the closure retains access to `count`.

This distinction is crucial.

---

## 17. Execution Context vs Closure

Wrong mental model:

```text
closure keeps
outer function
on the call stack
```

Correct:

```text
outer function returns
↓
execution context leaves call stack

but
↓
needed lexical bindings remain reachable
```

---

## 18. Global Context and Browser this

In a classic non-module browser script, global `this` is typically:

```js
window
```

But in ES modules:

```js
this
```

at top level is:

```text
undefined
```

Do not blindly say:

> global this is always window.

Runtime/module mode matters.

---

## 19. Function Declaration Example

```js
sayHello();

function sayHello() {
  console.log(
    "Hello"
  );
}
```

Why can this work before the textual declaration?

Because the function binding is established during the creation phase of the relevant execution context.

This leads directly into hoisting.

---

## 20. let / const Example

```js
console.log(
  count
);

let count = 10;
```

The binding exists in the lexical environment, but it is uninitialized until execution reaches the declaration.

That interval is the **Temporal Dead Zone**.

Again, execution context explains the mechanism behind the behavior.

---

## 21. Interview Mental Model

When JavaScript invokes a function, think:

```text
1. Create function execution context

2. Set up lexical environment

3. Create parameter/local bindings

4. Determine this

5. Push context onto call stack

6. Execute function body

7. Return

8. Pop context
```

This model connects multiple JavaScript topics together.

---

## 22. React Connection

Every React function component invocation is still a JavaScript function call.

Example:

```jsx
function Counter() {
  const count = 0;

  return (
    <button>
      {count}
    </button>
  );
}
```

Each render calls the component function again.

That means each render creates a new function execution context and new local bindings.

This helps explain stale closures later.

---

## Interview Questions

### What is an execution context?

The runtime environment created by JavaScript to execute global code or a function call.

### What are the main execution contexts?

Global execution context and function execution contexts.

### Is execution context the same as scope?

No. Scope is lexical accessibility; execution context is a runtime execution instance.

### Is execution context the same as call stack?

No. The call stack stores active execution contexts.

### Does a closure keep an execution context on the stack?

No. It retains access to lexical bindings after the outer context has left the stack.

---

## Interview Answer — 30 Seconds

> An execution context is the runtime environment JavaScript creates to execute code. There is a global execution context and a new function execution context for every function invocation. Each context contains lexical bindings, references to outer environments, and function-specific information such as `this`. Active execution contexts are managed by the call stack.

---

## Key Takeaways

- Global code gets a global execution context.
- Every function call creates a new function execution context.
- Contexts contain execution-time bindings and scope information.
- Scope and execution context are different concepts.
- The call stack stores active contexts.
- Context creation explains hoisting behavior.
- Closures retain lexical bindings, not call-stack frames.
