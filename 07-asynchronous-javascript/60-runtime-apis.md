# Lesson 60 — Runtime APIs and Asynchronous Operations

JavaScript itself does not directly provide every feature you use in an application.

Features such as:

- `setTimeout`
- `fetch`
- DOM events
- file-system access
- network sockets

come from the **runtime environment**.

This distinction is one of the most important foundations of asynchronous JavaScript.

---

## 1. JavaScript Engine vs Runtime

A JavaScript engine executes JavaScript.

Examples include engines such as:

- V8
- SpiderMonkey
- JavaScriptCore

But applications run inside a broader runtime environment.

Browser mental model:

```text
Browser Runtime
│
├── JavaScript Engine
│     ├── call stack
│     └── memory
│
├── Timer APIs
├── Network APIs
├── DOM APIs
├── Event APIs
└── Event Loop / Scheduling
```

The engine executes JavaScript.

The runtime provides surrounding capabilities.

---

# 2. Browser Runtime APIs

Browsers provide APIs such as:

```js
setTimeout(...)
fetch(...)
document.querySelector(...)
addEventListener(...)
localStorage
WebSocket
```

These are commonly called browser APIs or Web APIs.

They are available to JavaScript because the browser exposes them.

---

# 3. Node.js Runtime APIs

Node.js provides a different environment.

Examples:

```js
setTimeout(...)
fetch(...)
process
Buffer
fs
http
path
```

Some APIs overlap with browsers, but many are environment-specific.

For example:

```js
document
```

normally exists in browsers, not standard Node.js.

---

# 4. Why Runtime APIs Matter for Async

Consider:

```js
setTimeout(() => {
  console.log("done");
}, 1000);
```

The JavaScript engine does not sit on the call stack counting one second.

Instead:

```text
JavaScript
↓
calls runtime timer API
↓
runtime tracks timer
↓
JavaScript continues
↓
timer expires
↓
callback becomes eligible
↓
scheduler/event loop eventually runs callback
```

---

# 5. Timer Example Step by Step

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");
```

### Step 1

```js
console.log("A");
```

Output:

```text
A
```

### Step 2

```js
setTimeout(...)
```

The runtime registers a timer.

The callback does not execute yet.

### Step 3

JavaScript continues:

```js
console.log("C");
```

Output:

```text
C
```

### Step 4

After the delay condition is satisfied, the callback is scheduled for later execution.

Eventually:

```text
B
```

---

# 6. Runtime Owns the Waiting

This mental model is crucial:

```text
JavaScript engine
→ executes code

runtime API
→ handles waiting/external work

event loop/scheduler
→ coordinates when callbacks can return
```

Do not imagine the callback sitting inside the call stack for one second.

---

# 7. Network Request Example

```js
fetch("/api/users")
  .then((response) => {
    console.log(response);
  });
```

Conceptually:

```text
JavaScript
↓
request started through runtime networking
↓
JS continues
↓
network operation progresses
↓
response arrives
↓
promise becomes settled
↓
reaction scheduled
↓
callback eventually runs
```

Promises use microtask scheduling, which you will study later.

---

# 8. DOM Event Example

```js
button.addEventListener(
  "click",
  () => {
    console.log("clicked");
  }
);
```

The callback does not run immediately.

Instead:

```text
register listener
↓
JavaScript continues
↓
user clicks later
↓
browser creates event
↓
callback scheduled
↓
callback executes
```

Again, the browser runtime is handling the external event source.

---

# 9. Async Does Not Mean Every Runtime API Uses a New Thread

Avoid this oversimplification:

> every asynchronous operation gets its own thread

That is not a safe rule.

Different runtime APIs may use:

- operating-system async I/O
- internal event systems
- thread pools
- background threads
- platform-specific mechanisms

The important JavaScript-level idea is:

> the waiting work does not need to occupy the active JavaScript call stack.

---

# 10. Browser Rendering and JavaScript

The browser also handles:

- layout
- paint
- compositing
- user input
- networking

Long synchronous JavaScript can delay browser work because the main thread is busy.

Example:

```js
while (true) {
  // infinite main-thread block
}
```

The browser may become unresponsive.

This is why main-thread performance matters.

---

# 11. Web Workers Preview

Browsers can create workers:

```js
const worker =
  new Worker("worker.js");
```

A Web Worker runs JavaScript in a separate execution context/thread.

This is different from normal asynchronous callbacks on the main thread.

Mental model:

```text
Main thread JS
      │
      ├── async callbacks
      │   still return to main stack
      │
      └── Web Worker
          separate JS execution context
```

You will study workers later.

---

# 12. Node.js Async I/O Preview

Node.js is designed heavily around non-blocking I/O.

For example:

```js
import fs from "node:fs";

fs.readFile(
  "data.txt",
  "utf8",
  (error, data) => {
    // runs later
  }
);
```

The file operation is delegated to the Node runtime.

JavaScript can continue meanwhile.

---

# 13. Runtime Differences Matter

Code may work in one environment but not another.

Browser:

```js
document.querySelector(
  "#app"
);
```

Node:

```js
process.env.NODE_ENV;
```

So when asking:

> Is this JavaScript API?

also ask:

> Is it part of ECMAScript itself, or supplied by the runtime?

---

# 14. ECMAScript vs Host APIs

At a simplified level:

### ECMAScript language

Includes things such as:

- variables
- functions
- objects
- promises
- classes
- arrays
- maps
- sets

### Host/runtime APIs

Examples:

- browser DOM
- browser timers
- networking
- Node file system
- process APIs

The exact boundary has details, but this distinction is enough for learning async JavaScript.

---

# 15. Where Do Callbacks Wait?

A misleading mental model is:

```text
callback waits in call stack
```

Wrong.

Better:

```text
callback registered
↓
runtime operation continues
↓
completion/event occurs
↓
callback/task is queued/scheduled
↓
event loop waits until JS stack can run it
```

The queue details will be covered deeply in the Event Loop section.

---

# 16. Runtime + Event Loop Overview

Simplified browser diagram:

```text
┌─────────────────────┐
│ JavaScript Call     │
│ Stack               │
└──────────┬──────────┘
           │
           │ calls API
           ↓
┌─────────────────────┐
│ Browser Runtime APIs│
│ timers/network/DOM  │
└──────────┬──────────┘
           │
           │ completion
           ↓
┌─────────────────────┐
│ Scheduling Queues   │
└──────────┬──────────┘
           │
           ↓
      Event Loop
           │
           ↓
       Call Stack
```

This is simplified, but it gives the correct direction.

---

# 17. React / Next.js Connection

Frontend code constantly uses runtime APIs:

```js
fetch(...)
setTimeout(...)
addEventListener(...)
AbortController
```

Next.js also runs code in different environments:

- browser/client
- server runtime
- sometimes edge/runtime-specific environments

Knowing which APIs belong to which runtime prevents many bugs.

For example, browser-only globals such as:

```js
window
document
localStorage
```

are not universally available during server-side execution.

---

# 18. Common Mistakes

## Mistake 1 — Thinking setTimeout belongs to the JavaScript language itself

It is provided by the runtime environment.

## Mistake 2 — Thinking the JavaScript engine handles all I/O directly

The surrounding runtime provides the APIs and infrastructure.

## Mistake 3 — Thinking all async operations use identical internal mechanisms

They do not.

The JavaScript-level scheduling model matters more than assuming a specific implementation.

---

# 19. Interview Questions

### What is the difference between the JavaScript engine and runtime?

The engine executes JavaScript. The runtime includes the engine plus host APIs and scheduling infrastructure such as timers, networking, DOM APIs, and event-loop integration.

### Is setTimeout part of ECMAScript?

No. It is supplied by runtime environments such as browsers and Node.js.

### Where does asynchronous waiting happen?

The runtime handles the waiting operation outside the active JavaScript call stack, then schedules JavaScript work when appropriate.

---

# Key Takeaways

- JavaScript engines execute JavaScript code.
- Runtime environments provide additional APIs.
- Browsers and Node.js expose different host capabilities.
- Async waiting does not occupy the active JavaScript call stack.
- Timers, network requests, and events are coordinated by the runtime.
- The event loop helps schedule ready JavaScript callbacks.
- Runtime APIs do not all necessarily use the same internal threading mechanism.
- Environment awareness is especially important in React/Next.js applications.
