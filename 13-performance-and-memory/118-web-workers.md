# Lesson 118 — Web Workers Overview

Web Workers let JavaScript run code on a background thread separate from the browser's main UI thread.

The key idea is:

> Move suitable CPU-heavy work off the main thread so the UI can remain responsive.

---

## 1. Main Thread vs Worker

Without worker:

```text
Main thread
├── JavaScript
├── events
├── layout
└── rendering coordination
```

With worker:

```text
Main thread
├── UI
└── events

Worker thread
└── CPU-heavy JavaScript
```

---

## 2. Basic Worker

Main script:

```js
const worker =
  new Worker(
    "./worker.js"
  );
```

Worker file:

```js
// worker.js

self.onmessage =
  (event) => {
    console.log(
      event.data
    );
  };
```

---

## 3. Sending Data to Worker

Main:

```js
worker.postMessage({
  numbers: [
    1,
    2,
    3,
  ],
});
```

Worker receives:

```js
self.onmessage =
  (event) => {
    const data =
      event.data;
  };
```

---

## 4. Sending Result Back

Worker:

```js
self.postMessage({
  result: 42,
});
```

Main:

```js
worker.onmessage =
  (event) => {
    console.log(
      event.data
    );
  };
```

---

## 5. Message Passing Model

Workers do not normally share ordinary JavaScript objects directly.

Conceptually:

```text
main
↓ postMessage
structured clone / transfer
↓
worker

worker
↓ postMessage
structured clone / transfer
↓
main
```

This connects directly to `structuredClone()`.

---

## 6. Worker Cannot Directly Access DOM

Inside a worker:

```js
document.querySelector(
  "div"
);
```

is not available like it is on the main browser thread.

Workers do not directly manipulate the DOM.

---

## 7. Why No DOM Access?

The DOM is tied to the main browser UI environment.

Worker code should:

- compute
- parse
- transform
- analyze

Then send results back to main thread for DOM/UI updates.

---

## 8. Good Worker Use Cases

Examples:

- large data transformation
- image processing
- cryptographic computation
- compression
- parsing large files
- search/index calculations
- simulation
- CPU-heavy algorithms

---

## 9. Bad Worker Use Cases

Avoid using a worker for tiny work such as:

```js
a + b
```

Worker creation and message passing have overhead.

Use workers when the offloaded work is significant enough.

---

## 10. CPU Heavy Example

Worker:

```js
self.onmessage =
  (event) => {
    const limit =
      event.data.limit;

    let total = 0;

    for (
      let i = 0;
      i < limit;
      i++
    ) {
      total += i;
    }

    self.postMessage({
      total,
    });
  };
```

The main UI can remain more responsive while the worker computes.

---

## 11. Worker Lifecycle

Create:

```js
const worker =
  new Worker(
    "./worker.js"
  );
```

Use.

Then terminate when no longer needed:

```js
worker.terminate();
```

Worker code can also close itself.

---

## 12. Error Handling

Main:

```js
worker.onerror =
  (event) => {
    console.error(
      event.message
    );
  };
```

You should also design message-level error responses for expected task failures.

---

## 13. Module Workers

Modern browsers support module workers:

```js
const worker =
  new Worker(
    new URL(
      "./worker.js",
      import.meta.url
    ),
    {
      type: "module",
    }
  );
```

This allows ESM syntax inside the worker where supported.

---

## 14. Worker Global Scope

Workers use a worker-specific global environment.

You often see:

```js
self
```

instead of:

```js
window
```

There is no normal browser `window` object inside a dedicated worker.

---

## 15. Structured Clone Cost

Sending large objects can require copying.

Example:

```js
worker.postMessage(
  hugeObject
);
```

If the message is very large, cloning itself can be expensive.

So offloading work is not free.

---

## 16. Transferable Objects

For binary data, transfer ownership instead of copying.

Example:

```js
const buffer =
  new ArrayBuffer(
    10_000_000
  );

worker.postMessage(
  buffer,
  [
    buffer,
  ]
);
```

After transfer, the sender's buffer is detached.

This can avoid large memory copies.

---

## 17. SharedArrayBuffer

`SharedArrayBuffer` can enable true shared memory between execution contexts.

But it introduces:

- synchronization complexity
- Atomics
- security requirements
- race-condition concerns

It is an advanced topic beyond normal worker usage.

---

## 18. Worker vs Promise

Promise:

```text
async control flow
usually same JS thread for callback execution
```

Worker:

```text
separate execution thread
```

This distinction is crucial.

---

## 19. Worker vs setTimeout

```js
setTimeout(
  heavyTask,
  0
);
```

still runs `heavyTask` on the main thread later.

A worker actually executes suitable code off the main thread.

---

## 20. Worker vs Async/Await

```js
await heavyCalculation();
```

does not automatically move CPU work elsewhere.

Async/await manages Promise flow.

Workers provide separate-thread execution.

---

## 21. Web Worker Types

Common worker types:

### Dedicated Worker

Used by one page/context.

```js
new Worker(...)
```

### Shared Worker

Can potentially communicate with multiple same-origin browsing contexts.

Less commonly used.

### Service Worker

Different purpose:

- network proxying
- caching
- offline behavior
- background capabilities

A Service Worker is not simply a "Web Worker for calculations."

---

## 22. Dedicated Worker Focus

For CPU-heavy application logic, a Dedicated Worker is usually the first model to learn.

Flow:

```text
main creates worker
↓
send task
↓
worker computes
↓
worker sends result
↓
main updates UI
```

---

## 23. Worker Pool Concept

Creating one worker per tiny task may be expensive.

For repeated CPU work:

```text
create small pool
↓
reuse workers
↓
dispatch jobs
```

This is more advanced but useful in high-performance apps.

---

## 24. Cancellation

You can terminate a worker:

```js
worker.terminate();
```

But for reusable workers, a better architecture may send cancellation messages and let the worker stop specific tasks cooperatively.

---

## 25. React Connection

Conceptually:

```js
useEffect(() => {
  const worker =
    new Worker(
      workerUrl
    );

  worker.onmessage =
    (event) => {
      setResult(
        event.data
      );
    };

  return () => {
    worker.terminate();
  };
}, []);
```

Worker lifecycle should be cleaned up when the owning component is gone.

---

## 26. Next.js Connection

Workers are browser APIs when used on the client.

They should not be confused with:

- Node.js worker_threads
- serverless runtime concurrency
- Next.js server components

Those are different execution environments.

---

## 27. Worker Communication Design

Good messages are explicit:

```js
worker.postMessage({
  type:
    "PROCESS_DATA",
  payload:
    data,
});
```

Worker:

```js
self.onmessage =
  (event) => {
    switch (
      event.data.type
    ) {
      case "PROCESS_DATA":
        // ...
        break;
    }
  };
```

This scales better than ambiguous raw values.

---

## 28. Do Not Send More Data Than Needed

If the worker needs:

```text
ids + scores
```

do not send a massive full application state object.

Smaller messages mean less cloning/transfer overhead.

---

## 29. Worker Does Not Automatically Make Code Faster

A worker may improve **responsiveness** while increasing total overhead.

Costs include:

- startup
- serialization
- messaging
- coordination

Measure both:

- task completion time
- UI responsiveness

---

## 30. Decision Checklist

Use a worker when:

```text
Is work CPU-heavy?
↓
yes

Does it block main thread noticeably?
↓
yes

Can it run without direct DOM access?
↓
yes

Is message overhead acceptable?
↓
yes
```

Then a worker is a good candidate.

---

## Interview Questions

### What is a Web Worker?

A browser mechanism for executing JavaScript on a background thread separate from the main UI thread.

### Can a worker directly access the DOM?

No.

### Promise vs Web Worker?

Promises manage asynchronous control flow; workers provide separate-thread execution.

### Why use transferables?

To transfer ownership of certain binary resources without expensive copying.

### Does a worker always make work faster?

No. It primarily helps responsiveness and parallel CPU execution when overhead is justified.

---

## Section 13 Complete Mental Model

```text
memory allocated
↓
reachability
↓
garbage collection
↓
unwanted reachable objects
↓
memory leaks
↓
cleanup ownership
↓
efficient algorithms/data structures
↓
browser rendering cost
↓
main-thread blocking
↓
chunk / defer / optimize
↓
Web Worker for suitable CPU work
```

---

## Key Takeaways

- Web Workers run JavaScript off the main UI thread.
- Workers communicate through messages.
- Ordinary objects are cloned/transferred, not shared directly.
- Workers cannot directly manipulate the DOM.
- Use workers for meaningful CPU-heavy tasks.
- `setTimeout`, Promises, and async/await do not offload CPU work.
- Transferables can reduce large binary-copy cost.
- Worker lifecycle and cleanup matter.
- Measure whether worker overhead is justified.
