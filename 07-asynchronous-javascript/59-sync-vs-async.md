# Lesson 59 — Synchronous vs Asynchronous JavaScript

JavaScript often gets described as:

> single-threaded

That statement is useful, but incomplete.

The JavaScript language executes code on a single main call stack, yet JavaScript applications can still handle timers, network requests, user events, file operations, and many other asynchronous tasks.

To understand async JavaScript properly, first separate:

- synchronous execution
- asynchronous operations
- JavaScript engine
- runtime environment
- callbacks/promises
- event loop

This lesson focuses on the first mental model.

---

## 1. What Does Synchronous Mean?

Synchronous code runs in order, one step at a time.

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

Each statement finishes before the next one begins.

Mental model:

```text
A
↓
B
↓
C
```

---

# 2. JavaScript Call Stack

When functions execute, JavaScript uses the call stack.

Example:

```js
function second() {
  console.log("second");
}

function first() {
  console.log("first");

  second();

  console.log("done");
}

first();
```

Execution:

```text
first()
  ↓
console.log("first")
  ↓
second()
  ↓
console.log("second")
  ↓
second returns
  ↓
console.log("done")
  ↓
first returns
```

Output:

```text
first
second
done
```

---

# 3. What Does Single-Threaded Mean Here?

For ordinary JavaScript execution on the main thread:

> Only one JavaScript call stack is executing at a time.

That means JavaScript cannot execute two pieces of ordinary synchronous JavaScript on the same call stack at the exact same moment.

Example:

```js
function longTask() {
  // expensive synchronous work
}

longTask();

console.log("after");
```

`console.log("after")` cannot execute until `longTask()` returns.

---

# 4. Blocking Code

Synchronous code can block later work.

Example:

```js
console.log("start");

const start = Date.now();

while (
  Date.now() - start < 3000
) {
  // block for about 3 sec
}

console.log("end");
```

During that loop, the JavaScript thread is busy.

In a browser, this can prevent:

- clicks from being handled
- rendering updates
- animations
- timer callbacks
- other JavaScript tasks

This is called **blocking the main thread**.

---

# 5. Why We Need Asynchronous Operations

Imagine performing an API request synchronously:

```text
send request
↓
wait 3 seconds
↓
receive response
↓
continue
```

If JavaScript blocked the main thread while waiting, the page could become unresponsive.

Instead, asynchronous APIs allow JavaScript to initiate work and continue executing other code.

Conceptually:

```text
start operation
      ↓
runtime handles waiting
      ↓
JavaScript continues
      ↓
operation completes later
      ↓
result scheduled for JavaScript
```

---

# 6. Basic Asynchronous Example

```js
console.log("A");

setTimeout(() => {
  console.log("B");
}, 1000);

console.log("C");
```

Output:

```text
A
C
B
```

Why?

The timer does not block JavaScript for one second.

Instead:

```text
console.log("A")
↓
start timer
↓
continue immediately
↓
console.log("C")
↓
timer becomes ready later
↓
callback runs
```

---

# 7. Async Does Not Mean "Runs at the Same Time on the Call Stack"

This distinction is very important.

When:

```js
setTimeout(callback, 1000);
```

is used, the JavaScript engine is not keeping the callback running beside the current JavaScript code.

Instead, the runtime manages the timer externally.

The callback returns to JavaScript later when it is eligible to execute.

A better mental model:

```text
JavaScript stack
     │
     ├── synchronous code
     │
     └── asks runtime to handle async work
                 ↓
          runtime waits
                 ↓
          callback/result ready
                 ↓
          scheduled back to JS
```

---

# 8. Async Operation vs Async Callback

The asynchronous operation and the JavaScript callback are not the same thing.

Example:

```js
setTimeout(() => {
  console.log("done");
}, 1000);
```

The timer waiting is handled by the runtime.

The callback:

```js
() => {
  console.log("done");
}
```

is normal JavaScript code.

It executes later on the JavaScript call stack.

---

# 9. Common Asynchronous Operations

Browser examples:

- timers
- HTTP requests
- DOM events
- user input
- WebSocket messages
- IndexedDB operations
- some file APIs

Node.js examples:

- file system I/O
- network I/O
- database communication
- timers
- sockets

The exact APIs come from the environment, not from the core JavaScript language alone.

---

# 10. Synchronous Example with Function Calls

```js
function getUser() {
  return {
    name: "Vikash",
  };
}

const user =
  getUser();

console.log(user.name);
```

Everything completes immediately in normal sequence.

---

# 11. Asynchronous Data Cannot Usually Be Returned Like Synchronous Data

A common beginner mistake:

```js
function getUser() {
  let user;

  setTimeout(() => {
    user = {
      name: "Vikash",
    };
  }, 1000);

  return user;
}

console.log(
  getUser()
);
```

Output:

```text
undefined
```

Why?

Trace:

```text
getUser()
↓
user = undefined
↓
schedule timer
↓
return user
↓
undefined returned

later:
timer callback runs
↓
user assigned
```

The function already returned before the async result arrived.

---

# 12. How Async Results Are Normally Handled

Common approaches:

- callbacks
- promises
- async/await

Callback example:

```js
function getUser(callback) {
  setTimeout(() => {
    callback({
      name: "Vikash",
    });
  }, 1000);
}

getUser((user) => {
  console.log(
    user.name
  );
});
```

Later lessons will explain promises and `async/await`.

---

# 13. Sync vs Async Timeline

Synchronous:

```text
Task A
████

Task B
    ████

Task C
        ████
```

Each task finishes before the next begins.

Asynchronous waiting:

```text
JS:
start async work
██
continue work
  ████

Runtime async waiting:
  ───────────────

later callback:
                ██
```

JavaScript does not need to sit idle while the external operation waits.

---

# 14. Concurrency vs Parallelism

These words are often mixed up.

### Concurrency

Multiple tasks can make progress over overlapping periods.

A JavaScript application can be concurrent even if only one JavaScript callback runs on the main stack at a time.

### Parallelism

Multiple tasks literally execute at the same physical moment on different threads/cores.

Main-thread JavaScript execution is usually not parallel with another main-thread JavaScript callback.

However, runtimes can use background threads internally, and Web Workers/worker threads can run JavaScript in parallel in separate execution contexts.

For now:

> Async does not automatically mean parallel.

---

# 15. Browser Example

Suppose:

```js
console.log("loading");

fetch("/api/users");

console.log("continue");
```

Conceptually:

```text
JavaScript
↓
starts request
↓
browser/runtime handles network
↓
JavaScript continues
```

The browser does not freeze the call stack waiting for the server response.

---

# 16. Why Async Is Essential in Frontend Development

Frontend applications constantly perform operations that may take time:

- API calls
- image loading
- user interactions
- timers
- animations
- storage operations

Blocking the UI during these operations would create a poor user experience.

---

# 17. React Connection

React applications rely heavily on asynchronous behavior.

Examples:

- fetching data
- submitting forms
- debouncing search
- timers
- async server actions
- route loading
- background requests

Example:

```js
async function loadUsers() {
  const response =
    await fetch("/api/users");

  const users =
    await response.json();

  return users;
}
```

Understanding the runtime and event loop makes this code much easier to reason about.

---

# 18. Common Mistakes

## Mistake 1 — Async means immediate background JavaScript thread

Not necessarily.

The runtime may handle the waiting operation outside the main JavaScript stack, but the callback still returns to JavaScript execution later.

## Mistake 2 — setTimeout pauses execution

It does not.

```js
setTimeout(...);
```

schedules work and returns quickly.

## Mistake 3 — async means faster

Async code is not automatically faster.

Its main advantage is that waiting operations do not need to block other useful work.

---

# 19. Interview Questions

### Is JavaScript synchronous or asynchronous?

The JavaScript language executes synchronous code on a single call stack, but runtime environments provide asynchronous APIs that let applications perform non-blocking operations.

### Does asynchronous JavaScript mean two JavaScript callbacks run simultaneously on one call stack?

No.

### Why is asynchronous programming important?

It prevents waiting operations such as network requests and timers from unnecessarily blocking application execution.

---

# Key Takeaways

- Synchronous JavaScript runs step by step.
- JavaScript uses a call stack for function execution.
- Main-thread JavaScript executes one stack operation at a time.
- Long synchronous work blocks later JavaScript and can freeze the UI.
- Async APIs allow waiting work to be handled outside the active JavaScript call stack.
- Async callbacks still execute later as normal JavaScript.
- Async does not automatically mean parallel.
- Async results cannot usually be returned like immediate synchronous values.
- Callbacks, promises, and async/await are common ways to handle async results.
