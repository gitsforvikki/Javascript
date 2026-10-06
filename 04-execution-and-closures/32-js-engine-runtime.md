# Lesson 32 — JavaScript Engine and Runtime Overview

Before learning execution context, hoisting, the call stack, closures, or the event loop, you need a clean mental model of **where JavaScript actually runs**.

The two terms that are often confused are:

- **JavaScript Engine**
- **JavaScript Runtime**

They are related, but they are not the same thing.

---

## 1. JavaScript Engine

A JavaScript engine is the software that reads and executes JavaScript code.

Examples:

- **V8** — Chrome, Edge, Node.js
- **SpiderMonkey** — Firefox
- **JavaScriptCore** — Safari

The engine is responsible for tasks such as:

- parsing JavaScript
- creating internal representations of code
- compiling/interpreting code
- executing instructions
- managing memory
- garbage collection
- optimizing frequently executed code

---

## 2. JavaScript Runtime

A JavaScript runtime is the complete environment in which JavaScript runs.

It includes:

```text
JavaScript Engine
+
Runtime APIs
+
Task scheduling infrastructure
```

Examples of runtime environments:

- Browser
- Node.js
- Deno
- Bun

---

## 3. Engine vs Runtime

A simple mental model:

```text
JavaScript Engine
→ executes JavaScript

JavaScript Runtime
→ engine + environment capabilities
```

Example:

```js
console.log(
  window
);
```

`window` is not part of the JavaScript language itself.

It is provided by the **browser runtime**.

Similarly:

```js
setTimeout(
  () => {
    console.log("Hello");
  },
  1000
);
```

`setTimeout` is not implemented by the JavaScript language specification as a core language feature.

It is provided by the runtime environment.

---

## 4. Browser Runtime

A browser runtime conceptually contains:

```text
Browser
│
├── JavaScript Engine
│
├── Web APIs
│   ├── DOM
│   ├── setTimeout
│   ├── fetch
│   ├── localStorage
│   └── event listeners
│
├── Task Queue
├── Microtask Queue
└── Event Loop
```

The engine executes JavaScript.

The browser provides additional APIs.

---

## 5. Node.js Runtime

Node.js uses the V8 engine but adds a different runtime environment.

Conceptually:

```text
Node.js
│
├── V8 Engine
│
├── Node APIs
│   ├── fs
│   ├── http
│   ├── process
│   └── timers
│
└── libuv
    ├── event loop
    ├── async I/O
    └── thread pool
```

So:

> Chrome and Node.js can both use V8, but they are different runtimes.

---

## 6. JavaScript Is Single-Threaded — What Does That Mean?

For normal JavaScript execution in one execution environment, the engine executes one JavaScript call stack at a time.

That means:

```text
one active JavaScript call
↓
another call waits
```

Example:

```js
function first() {
  console.log("first");
}

function second() {
  console.log("second");
}

first();
second();
```

`first()` completes before `second()` executes.

---

## 7. Then How Can JavaScript Be Asynchronous?

Because the runtime can perform some work outside the JavaScript call stack.

Example:

```js
setTimeout(
  () => {
    console.log("timer");
  },
  1000
);

console.log("end");
```

The JavaScript engine does not sit inside the callback for one second.

Conceptually:

```text
JavaScript Engine
↓
register timer with runtime
↓
continue executing
↓
runtime waits
↓
callback becomes eligible later
```

The Event Loop section explains how the callback comes back to JavaScript.

---

## 8. Parsing

Before code executes, the engine must understand the source code.

Example:

```js
const total =
  price * quantity;
```

The engine parses this source into an internal structure, commonly described conceptually as an **AST — Abstract Syntax Tree**.

You do not need to memorize the exact internal engine representation for interviews.

Important idea:

```text
Source Code
↓
Parsing
↓
Internal Representation
↓
Execution / Compilation
```

---

## 9. Syntax Errors Can Happen Before Execution

Example:

```js
const = 10;
```

The parser cannot construct a valid program.

So the code fails before ordinary runtime execution begins.

This is different from:

```js
const user = null;

console.log(
  user.name
);
```

which is syntactically valid but fails during execution.

---

## 10. Compilation and Interpretation

Modern JavaScript engines are neither simply:

```text
pure interpreters
```

nor simply:

```text
traditional ahead-of-time compilers
```

They usually combine techniques.

A simplified mental model:

```text
JavaScript Source
↓
Parse
↓
Initial execution
↓
Observe hot code
↓
Optimize hot paths
```

This is commonly called **JIT — Just-In-Time compilation**.

---

## 11. JIT Compilation

Frequently executed code may be optimized by the engine.

Conceptually:

```js
function add(
  a,
  b
) {
  return a + b;
}
```

If this function runs many times with predictable types, the engine may generate optimized machine code.

If later assumptions become invalid, the engine may **deoptimize**.

You will study this more deeply in the JavaScript Internals section.

---

## 12. Memory Management

The engine also manages memory for:

- variables
- objects
- functions
- closures
- execution contexts

Example:

```js
const user = {
  name: "Vikash",
};
```

The engine allocates memory for the object and tracks references to it.

When an object becomes unreachable, garbage collection may eventually reclaim its memory.

---

## 13. Engine Responsibilities vs Runtime Responsibilities

A useful interview table:

| JavaScript Engine | Runtime Environment |
| --- | --- |
| Parse JavaScript | Provide external APIs |
| Execute JavaScript | DOM / filesystem / networking |
| Manage execution contexts | Schedule async callbacks |
| Maintain call stack | Event loop infrastructure |
| Garbage collection | Browser/Node-specific capabilities |
| Optimize code | Environment integration |

---

## 14. Example — fetch()

```js
fetch(
  "/api/users"
)
  .then(
    console.log
  );
```

High-level flow:

```text
JavaScript
↓
calls fetch()

Runtime
↓
performs network operation

Promise settles
↓
microtask scheduled

Event loop
↓
allows callback to run

Engine
↓
executes callback
```

This shows why engine and runtime must be understood separately.

---

## 15. Example — DOM

```js
document.querySelector(
  "#app"
);
```

`document` is provided by the browser runtime.

Node.js does not provide a browser DOM by default.

So the same JavaScript language can run in different runtimes with different APIs.

---

## 16. Example — Node.js

Browser:

```js
window
document
```

Node.js:

```js
process
Buffer
```

These are runtime-provided APIs.

They are not universal JavaScript language features.

---

## 17. Common Interview Trap

Question:

> Is setTimeout part of JavaScript?

Best answer:

> The timer function is provided by the runtime environment. Browsers provide timer APIs, and Node.js provides compatible timer APIs. The JavaScript engine executes the callback when it eventually reaches the call stack.

---

## 18. Common Interview Trap — Is JavaScript Compiled or Interpreted?

Avoid saying only:

```text
JavaScript is interpreted.
```

Modern engines use parsing, bytecode/intermediate representations, JIT compilation, and optimization.

A better answer:

> Modern JavaScript engines use a combination of interpretation and JIT compilation.

---

## 19. Common Interview Trap — Is JavaScript Single-Threaded?

A precise answer:

> JavaScript execution for a given main call stack is single-threaded, but the runtime can perform asynchronous operations outside that stack. Browsers can also use workers, and Node.js can use worker threads and libuv facilities.

This is more accurate than saying:

```text
JavaScript can only ever do one thing.
```

---

## 20. Connection to the Next Lessons

The engine/runtime model leads directly into:

```text
JavaScript Engine
↓
Execution Context
↓
Creation Phase
↓
Execution Phase
↓
Call Stack
↓
Hoisting
↓
Lexical Environment
↓
Closures
```

This sequence is essential for interview questions.

---

## Interview Questions

### What is a JavaScript engine?

Software that parses, executes, optimizes JavaScript, and manages memory.

### What is a JavaScript runtime?

The complete environment containing the engine plus APIs and scheduling infrastructure.

### Is V8 a runtime?

No. V8 is a JavaScript engine.

### Is Node.js a JavaScript engine?

No. Node.js is a runtime that uses V8.

### Is setTimeout part of the JavaScript language?

No. It is provided by the runtime environment.

### Is JavaScript interpreted or compiled?

Modern engines use both interpretation-like execution and JIT compilation/optimization techniques.

---

## Interview Answer — 30 Seconds

> A JavaScript engine such as V8 is responsible for parsing, executing, optimizing JavaScript, and managing memory. A runtime such as Chrome or Node.js includes that engine plus environment-specific APIs and asynchronous scheduling infrastructure. JavaScript executes one call stack at a time, while the runtime handles things like timers, networking, and I/O outside that stack and later schedules callbacks back into JavaScript.

---

## Key Takeaways

- Engine and runtime are not the same thing.
- V8, SpiderMonkey, and JavaScriptCore are engines.
- Browsers and Node.js are runtimes.
- Runtime APIs such as DOM, timers, and filesystem APIs are not core JavaScript syntax.
- Modern engines use parsing, JIT compilation, optimization, and garbage collection.
- JavaScript execution uses one active call stack at a time.
- Asynchronous behavior is enabled by the runtime plus scheduling mechanisms.
