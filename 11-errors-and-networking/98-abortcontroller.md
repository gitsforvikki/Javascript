# Lesson 98 — `AbortController` and Request Cancellation

Sometimes an async request is no longer useful.

Examples:

- user navigated away
- user typed a newer search query
- component unmounted
- timeout expired
- user clicked Cancel

In these cases, you may want to **cancel** the operation rather than merely ignore its result.

The browser provides:

```text
AbortController
AbortSignal
```

---

## 1. Basic Mental Model

```text
AbortController
      │
      └── signal
           │
           ↓
       async API

controller.abort()
      ↓
signal becomes aborted
      ↓
API reacts to cancellation
```

---

## 2. Basic Fetch Cancellation

```js
const controller =
  new AbortController();

const responsePromise =
  fetch(
    "/api/jobs",
    {
      signal:
        controller.signal,
    }
  );
```

Later:

```js
controller.abort();
```

The fetch is cancelled if it is still in progress and the API honors the signal.

---

## 3. AbortSignal

```js
controller.signal
```

is the object passed to cancellable APIs.

Useful properties:

```js
signal.aborted
signal.reason
```

---

## 4. Abort Is One-Way

After:

```js
controller.abort();
```

the signal remains aborted.

You cannot reset the same controller.

For another operation:

```js
const nextController =
  new AbortController();
```

---

## 5. Fetch Rejection on Abort

When aborted, Fetch rejects.

Modern environments may expose an abort-related error/reason.

Example:

```js
try {
  await fetch(
    "/api/jobs",
    {
      signal:
        controller.signal,
    }
  );
} catch (error) {
  if (
    controller.signal
      .aborted
  ) {
    console.log(
      "Request cancelled"
    );

    return;
  }

  throw error;
}
```

Checking the signal is often clearer than relying only on error-name strings.

---

## 6. Why Cancellation Matters

Without cancellation:

```text
old request starts
↓
new request starts
↓
new request finishes
↓
old request finishes later
↓
old result overwrites new UI
```

This is a race condition.

Cancellation can prevent stale requests from continuing.

---

## 7. Search-As-You-Type Example

```js
let controller;

async function search(
  query
) {
  controller?.abort();

  controller =
    new AbortController();

  const response =
    await fetch(
      `/api/search?q=${encodeURIComponent(
        query
      )}`,
      {
        signal:
          controller.signal,
      }
    );

  return response.json();
}
```

Each new search aborts the previous request.

---

## 8. Cancellation vs Ignoring Results

Two strategies:

### Ignore stale result

```text
request keeps running
↓
result discarded later
```

### Abort request

```text
signal cancellation
↓
runtime/API attempts to stop work
```

Cancellation can save:

- bandwidth
- server/client work
- memory
- stale-state bugs

But server-side work may already have begun.

---

## 9. Important: Client Abort Does Not Guarantee Server Rollback

Suppose:

```text
POST /payment
↓
server receives request
↓
payment processing starts
↓
client aborts connection
```

The server may still finish the operation.

So:

> Aborting a fetch is not a transaction rollback.

For important mutations, design idempotency and server-side consistency separately.

---

## 10. abort(reason)

Modern AbortController supports an optional reason:

```js
controller.abort(
  new Error(
    "User navigated away"
  )
);
```

Then:

```js
controller.signal.reason
```

can describe why cancellation happened.

---

## 11. Abort Before Starting

```js
const controller =
  new AbortController();

controller.abort();

await fetch(url, {
  signal:
    controller.signal,
});
```

The operation sees an already-aborted signal and rejects accordingly.

---

## 12. One Signal, Multiple Operations

You can pass one signal to several operations:

```js
const controller =
  new AbortController();

const a =
  fetch(
    "/api/a",
    {
      signal:
        controller.signal,
    }
  );

const b =
  fetch(
    "/api/b",
    {
      signal:
        controller.signal,
    }
  );
```

Then:

```js
controller.abort();
```

cancels both signal-linked requests.

Useful for cancelling a group of related operations.

---

## 13. AbortSignal.timeout()

Modern runtimes may support:

```js
AbortSignal.timeout(
  5000
);
```

Example:

```js
await fetch(
  "/api/jobs",
  {
    signal:
      AbortSignal.timeout(
        5000
      ),
  }
);
```

This creates a signal that aborts after the timeout.

Support should be checked for your target environment.

Lesson 99 will also show a portable timeout pattern.

---

## 14. AbortSignal.any()

Modern runtimes may support combining signals:

```js
const signal =
  AbortSignal.any([
    userController.signal,
    AbortSignal.timeout(
      5000
    ),
  ]);
```

Now either condition can cancel the operation.

Again, verify target-runtime support.

---

## 15. Custom Async API with Signal

You can design your own cancellable function.

```js
function delay(
  ms,
  signal
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      if (
        signal?.aborted
      ) {
        reject(
          signal.reason
        );

        return;
      }

      const id =
        setTimeout(
          resolve,
          ms
        );

      signal?.addEventListener(
        "abort",
        () => {
          clearTimeout(id);

          reject(
            signal.reason
          );
        },
        {
          once: true,
        }
      );
    }
  );
}
```

The caller controls cancellation through the signal.

---

## 16. Why Signal Is Better Than a Boolean Flag

Boolean:

```js
let cancelled = false;
```

only lets your own code check state.

AbortSignal provides a standard protocol:

- current cancellation state
- event notification
- cancellation reason
- compatibility with browser APIs

---

## 17. React useEffect Cleanup

Example:

```jsx
useEffect(() => {
  const controller =
    new AbortController();

  async function load() {
    try {
      const response =
        await fetch(
          "/api/jobs",
          {
            signal:
              controller.signal,
          }
        );

      const data =
        await response.json();

      setJobs(data);
    } catch (error) {
      if (
        controller.signal
          .aborted
      ) {
        return;
      }

      setError(error);
    }
  }

  load();

  return () => {
    controller.abort();
  };
}, []);
```

Cleanup aborts the request when the effect is disposed.

---

## 18. Next.js / Modern React Note

Not every request belongs in `useEffect`.

In modern Next.js, server-side fetching and framework-level data patterns are often preferable.

But when client-side requests are truly needed, cancellation remains useful.

---

## 19. Cancellation Is Not Failure in the Same Product Sense

A cancelled request may be expected behavior.

Example:

```text
user typed new query
→ old request aborted
```

You probably should not show:

```text
Something went wrong!
```

for intentional cancellation.

Handle cancellation separately from genuine failures.

---

## 20. Promise.race Timeout vs True Cancellation

You may see:

```js
await Promise.race([
  fetch(url),
  timeoutPromise,
]);
```

This can stop waiting for Fetch.

But it does not automatically cancel the underlying Fetch request.

Using AbortController can actually signal cancellation to Fetch.

Important distinction:

```text
stop awaiting
≠
cancel operation
```

---

## 21. Common Mistakes

### Mistake 1

Reusing an already-aborted controller.

### Mistake 2

Treating user cancellation as an application error.

### Mistake 3

Assuming abort rolls back server mutations.

### Mistake 4

Using Promise.race timeout and assuming the losing fetch is cancelled.

### Mistake 5

Creating a controller but forgetting to pass its signal.

---

## Interview Questions

### What is AbortController?

A standard controller that can signal cancellation to compatible asynchronous APIs.

### What is AbortSignal?

The signal object passed to the async operation.

### Can an AbortController be reused after abort?

No. Create a new controller for a new operation.

### Does aborting a fetch guarantee server work stops?

No.

### Promise.race timeout vs AbortController?

`Promise.race` can stop waiting, while AbortController can actually signal cancellation to Fetch.

---

## Key Takeaways

- AbortController provides standard cancellation.
- Pass `controller.signal` to cancellable APIs.
- `abort()` is one-way.
- Cancellation should often be handled separately from failure.
- One signal can cancel multiple operations.
- Aborting Fetch does not guarantee server rollback.
- Cancellation helps prevent stale-request race conditions.
- Timeout and cancellation are related but not identical concepts.
