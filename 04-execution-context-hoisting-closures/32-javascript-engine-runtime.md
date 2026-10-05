# Lesson 32 — JavaScript Engine and Runtime

Before learning execution context, call stack, hoisting, closures, and the event loop, we need a clear mental model of **where and how JavaScript runs**.

JavaScript code does not execute by itself. It needs a **JavaScript engine**, and applications usually run inside a larger **runtime environment** such as a browser or Node.js.

---

# 1. What is a JavaScript Engine?

A JavaScript engine is software that reads and executes JavaScript code.

Examples include:

| Engine | Common Environment |
| --- | --- |
| V8 | Chrome, Node.js |
| SpiderMonkey | Firefox |
| JavaScriptCore | Safari |

For this course, the exact engine implementation is less important than understanding the execution model shared by JavaScript.

A simplified flow is:

```text
JavaScript Source Code
        ↓
      Parser
        ↓
Internal Representation
        ↓
Interpreter / Compiler
        ↓
Machine Instructions
        ↓
       CPU
```

Modern engines use sophisticated interpretation and compilation techniques for performance. We will revisit engine internals later.

---

# 2. Parsing

Consider:

```js
const total = 10 + 20;

console.log(total);
```

Before executing the program, the engine needs to understand its structure.

Conceptually:

```text
Source Code
    ↓
Tokenization / Parsing
    ↓
Syntax Structure
    ↓
Execution
```

If the source contains invalid syntax:

```js
const = 10;
```

the engine cannot parse it correctly and reports a syntax error.

---

# 3. JavaScript Engine vs JavaScript Runtime

These terms are related but not identical.

## JavaScript Engine

The engine executes the JavaScript language itself.

Conceptually it manages things such as:

- execution contexts
- call stack
- memory
- objects and functions
- garbage collection
- language execution

## Runtime Environment

A runtime combines the JavaScript engine with additional APIs and infrastructure.

Browser example:

```text
Browser Runtime
│
├── JavaScript Engine
│   ├── Call Stack
│   ├── Heap / Memory
│   └── Language Execution
│
├── DOM APIs
├── fetch
├── timers
├── events
└── scheduling infrastructure
```

The browser provides capabilities that are not part of the core JavaScript language itself.

---

# 4. JavaScript vs Browser APIs

Consider:

```js
setTimeout(() => {
  console.log("Hello");
}, 1000);
```

`setTimeout` is commonly available in browsers, but it is not a core ECMAScript language feature.

Similarly:

```js
document.querySelector("#app");
```

The `document` object comes from the browser environment.

JavaScript itself defines language features such as:

```text
variables
functions
objects
arrays
promises
classes
operators
control flow
```

The environment provides additional APIs.

---

# 5. Browser Runtime

A simplified browser runtime looks like:

```text
             Browser
                │
      ┌─────────┴─────────┐
      │                   │
JavaScript Engine      Browser APIs
      │                   │
 Call Stack          DOM / Timers /
 Memory              Network / Events
      │                   │
      └─────────┬─────────┘
                │
          Event Loop /
             Queues
```

This diagram will become especially important when we study asynchronous JavaScript and the event loop.

For now, remember:

> The JavaScript engine executes JavaScript. The browser supplies additional capabilities around it.

---

# 6. Node.js Runtime

JavaScript can also run outside the browser using environments such as Node.js.

Simplified model:

```text
Node.js Runtime
│
├── V8 JavaScript Engine
│
├── Node APIs
│   ├── filesystem
│   ├── networking
│   ├── process
│   └── timers
│
└── event-loop infrastructure
```

Browser:

```js
document.querySelector("button");
```

Node.js does not normally provide the browser DOM, so `document` is not available by default.

Node.js, on the other hand, provides server-oriented capabilities such as filesystem access.

---

# 7. Memory: Stack and Heap — Initial Mental Model

At this stage, use a simplified mental model.

The engine needs memory for program execution.

```text
JavaScript Engine
│
├── Call Stack
│   └── tracks active function execution
│
└── Heap / Managed Memory
    └── objects, arrays, functions, etc.
```

We will refine this model later.

Do not treat “primitive = stack, object = heap” as a universal language rule. Engine implementations are free to optimize storage internally.

For learning JavaScript behavior, the more important distinction is:

- primitive values behave as values
- objects are accessed through references
- execution contexts are tracked through the call stack

---

# 8. Is JavaScript Single-Threaded?

JavaScript execution is commonly described as **single-threaded** because one JavaScript call stack executes JavaScript instructions in sequence on the main execution thread.

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

However, the surrounding runtime can handle other work such as timers, networking, rendering, or I/O and coordinate callbacks with JavaScript.

That is why this statement:

> “JavaScript is single-threaded, therefore it cannot do asynchronous work”

is incorrect.

A better model is:

```text
JavaScript execution
       ↓
single call stack

Runtime environment
       ↓
can coordinate asynchronous operations
```

We will understand this deeply in the asynchronous JavaScript and event-loop sections.

---

# 9. Why This Matters for React and Next.js

React code ultimately executes as JavaScript.

Understanding the runtime helps explain:

- why expensive synchronous JavaScript can block the browser
- why API requests do not freeze execution while waiting
- how event handlers are eventually executed
- why promises have specific execution ordering
- differences between browser and server environments

Next.js makes runtime awareness especially important because JavaScript may execute in different environments depending on the code.

---

# 10. Interview Perspective

**What is a JavaScript engine?**

A JavaScript engine is software responsible for parsing and executing JavaScript code and managing the language's runtime execution.

**What is the difference between a JavaScript engine and runtime?**

The engine executes JavaScript itself. A runtime environment combines an engine with additional APIs and infrastructure, such as browser DOM/network APIs or Node.js server APIs.

**Is setTimeout part of JavaScript itself?**

The timer API is supplied by the host/runtime environment rather than being a core ECMAScript language feature.

**Is JavaScript single-threaded?**

JavaScript code normally executes through one call stack at a time, while the surrounding runtime can coordinate asynchronous work outside that stack.

---

# Key Takeaways

- JavaScript needs an engine to execute.
- V8, SpiderMonkey, and JavaScriptCore are JavaScript engines.
- A runtime contains an engine plus environment-specific APIs.
- Browser APIs are not the same thing as the JavaScript language.
- The call stack tracks active JavaScript execution.
- The surrounding runtime enables asynchronous behavior.
- This model is the foundation for execution contexts and the event loop.
