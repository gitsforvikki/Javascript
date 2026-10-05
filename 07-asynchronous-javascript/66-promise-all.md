# Lesson 66 — `Promise.all()`

So far, most examples have involved **dependent sequential operations**:

```text
get user
↓
get user's orders
↓
get order details
```

But sometimes operations are independent.

Example:

- load profile
- load notifications
- load recommendations

If they do not depend on one another, starting them together can be more efficient.

`Promise.all()` is one of the most important tools for this.

---

## 1. Basic Syntax

```js
Promise.all([
  promise1,
  promise2,
  promise3,
]);
```

It returns a new Promise.

That Promise:

- fulfills when **all input promises fulfill**
- rejects when **any input rejects**

---

# 2. Basic Example

```js
const p1 =
  Promise.resolve(10);

const p2 =
  Promise.resolve(20);

const p3 =
  Promise.resolve(30);

Promise.all([
  p1,
  p2,
  p3,
])
  .then((values) => {
    console.log(values);
  });
```

Output:

```js
[10, 20, 30]
```

---

# 3. Result Order Is Input Order

This is important.

```js
const slow =
  new Promise(
    (resolve) => {
      setTimeout(() => {
        resolve("slow");
      }, 2000);
    }
  );

const fast =
  new Promise(
    (resolve) => {
      setTimeout(() => {
        resolve("fast");
      }, 500);
    }
  );
```

Now:

```js
Promise.all([
  slow,
  fast,
])
  .then((values) => {
    console.log(values);
  });
```

Output:

```js
["slow", "fast"]
```

Even though `fast` finishes first, results preserve input order.

---

# 4. Promise.all Waits for All Fulfillments

```js
const p1 =
  new Promise(
    (resolve) => {
      setTimeout(
        () => resolve("A"),
        1000
      );
    }
  );

const p2 =
  new Promise(
    (resolve) => {
      setTimeout(
        () => resolve("B"),
        2000
      );
    }
  );
```

`Promise.all([p1, p2])` fulfills only after both are fulfilled.

Conceptually:

```text
p1 done at ~1s
p2 done at ~2s
↓
all done
↓
Promise.all fulfills
```

---

# 5. Fail-Fast Rejection Behavior

```js
const p1 =
  Promise.resolve("A");

const p2 =
  Promise.reject(
    new Error("Failed")
  );

const p3 =
  Promise.resolve("C");

Promise.all([
  p1,
  p2,
  p3,
])
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

Output:

```text
Failed
```

As soon as one input rejection determines failure, the returned `Promise.all()` Promise rejects.

This is commonly called **fail-fast behavior**.

---

# 6. Important: Promise.all Does Not Cancel Other Operations

This distinction is crucial.

Suppose:

```js
const p1 =
  fetch("/api/a");

const p2 =
  fetch("/api/b");

const p3 =
  fetch("/api/c");
```

If `p2` rejects, the Promise returned by `Promise.all()` rejects.

But that does **not** automatically cancel `p1` and `p3`.

Their underlying operations may continue.

Mental model:

```text
Promise.all result
can reject early

≠

other async operations
automatically cancelled
```

Cancellation requires separate mechanisms such as `AbortController` where supported.

---

# 7. Independent vs Dependent Operations

Use `Promise.all()` when tasks are independent.

Good:

```js
Promise.all([
  loadProfile(),
  loadNotifications(),
  loadRecommendations(),
]);
```

Each can start immediately.

Not appropriate:

```text
get user
↓
need user.id
↓
get orders
```

You cannot start `getOrders(user.id)` before you know `user.id`.

That is a dependent sequence and should use chaining or `async/await`.

---

# 8. Sequential vs Parallel-Concurrent Start

Sequential:

```js
getProfile()
  .then((profile) => {
    return getNotifications();
  })
  .then((notifications) => {
    return getRecommendations();
  });
```

This waits for each previous operation even if they are unrelated.

Better when independent:

```js
Promise.all([
  getProfile(),
  getNotifications(),
  getRecommendations(),
]);
```

All operations are initiated before waiting for collective completion.

---

# 9. Timing Example

Suppose:

```text
profile: 2s
notifications: 1s
recommendations: 3s
```

Sequential:

```text
2 + 1 + 3
≈ 6 seconds
```

Started together:

```text
max(2, 1, 3)
≈ 3 seconds
```

Ignoring overhead and environmental variation.

This is why concurrency matters.

---

# 10. Non-Promise Values Are Accepted

```js
Promise.all([
  Promise.resolve(10),
  20,
  Promise.resolve(30),
])
  .then((values) => {
    console.log(values);
  });
```

Output:

```js
[10, 20, 30]
```

Non-Promise values are treated like already-fulfilled inputs.

---

# 11. Empty Iterable

```js
Promise.all([])
  .then((values) => {
    console.log(values);
  });
```

Result:

```js
[]
```

The returned Promise is fulfilled with an empty array.

Its handlers still follow normal Promise asynchronous scheduling.

---

# 12. Fetch Example

```js
Promise.all([
  fetch("/api/profile"),
  fetch("/api/jobs"),
])
  .then(
    ([
      profileResponse,
      jobsResponse,
    ]) => {
      return Promise.all([
        profileResponse.json(),
        jobsResponse.json(),
      ]);
    }
  )
  .then(
    ([
      profile,
      jobs,
    ]) => {
      console.log(
        profile,
        jobs
      );
    }
  );
```

This example contains two levels:

1. wait for both HTTP responses
2. wait for both body-parsing Promises

---

# 13. Cleaner Decomposition

```js
function fetchJson(url) {
  return fetch(url)
    .then((response) => {
      if (!response.ok) {
        throw new Error(
          `HTTP ${response.status}`
        );
      }

      return response.json();
    });
}
```

Then:

```js
Promise.all([
  fetchJson(
    "/api/profile"
  ),
  fetchJson(
    "/api/jobs"
  ),
])
  .then(
    ([
      profile,
      jobs,
    ]) => {
      console.log(
        profile,
        jobs
      );
    }
  );
```

Much easier to read.

---

# 14. Destructuring Results

Because results preserve input order:

```js
Promise.all([
  getUser(),
  getJobs(),
  getNotifications(),
])
  .then(
    ([
      user,
      jobs,
      notifications,
    ]) => {
      // use values
    }
  );
```

This is a common real-world pattern.

---

# 15. One Rejection Loses Combined Fulfillment Result

If one Promise rejects:

```js
Promise.all([
  Promise.resolve("A"),
  Promise.reject("B failed"),
  Promise.resolve("C"),
]);
```

the returned Promise does not fulfill with something like:

```js
[
  "A",
  error,
  "C",
]
```

Instead, it rejects.

If you want to observe all outcomes regardless of failures, `Promise.allSettled()` is designed for that and will be covered later.

---

# 16. Handling Individual Failure Before Promise.all

Sometimes you intentionally convert one failure into a fallback.

```js
const safeNotifications =
  getNotifications()
    .catch(() => {
      return [];
    });
```

Then:

```js
Promise.all([
  getProfile(),
  safeNotifications,
])
  .then(
    ([
      profile,
      notifications,
    ]) => {
      // still fulfills if
      // notifications failed
    }
  );
```

Why?

Because the local `catch()` recovered that Promise into a fulfilled Promise with `[]`.

---

# 17. But Do Not Hide Important Failures Accidentally

This:

```js
task()
  .catch(() => {
    return null;
  });
```

turns failure into success with `null`.

That may be correct for optional data.

But it may hide a critical error.

Error recovery should be intentional.

---

# 18. Promise.all Does Not Make Synchronous Work Parallel

Consider CPU-heavy synchronous functions:

```js
Promise.all([
  Promise.resolve(
    heavyTask1()
  ),
  Promise.resolve(
    heavyTask2()
  ),
]);
```

The function calls:

```js
heavyTask1()
heavyTask2()
```

execute synchronously before being wrapped.

`Promise.all()` does not magically move synchronous CPU work to separate threads.

Important:

> Promise concurrency is about coordinating asynchronous results, not creating CPU parallelism.

---

# 19. Correct Async Function Start

If these return Promises:

```js
Promise.all([
  loadA(),
  loadB(),
  loadC(),
]);
```

their async operations can be initiated close together.

But if a function performs large synchronous work before returning its Promise, that synchronous work can still block.

---

# 20. Practical Application Example

Dashboard:

```js
Promise.all([
  loadProfile(),
  loadApplicationStats(),
  loadRecentJobs(),
])
  .then(
    ([
      profile,
      stats,
      jobs,
    ]) => {
      renderDashboard({
        profile,
        stats,
        jobs,
      });
    }
  )
  .catch((error) => {
    showError(
      error.message
    );
  });
```

All independent dashboard requests begin together.

---

# 21. React / Next.js Connection

You may use this pattern in server-side or client-side data aggregation:

```js
const [
  user,
  applications,
  stats,
] = await Promise.all([
  getUser(),
  getApplications(),
  getStats(),
]);
```

This `async/await` version uses the same `Promise.all()` semantics.

Very common in production code.

---

# 22. Promise.all Mental Model

```text
Promise.all([
  A,
  B,
  C
])
│
├── A fulfills
├── B fulfills
└── C fulfills
      ↓
fulfill with
[A, B, C]

OR

any one rejects
      ↓
returned Promise rejects
```

Remember:

```text
result order
=
input order

not
completion order
```

---

# Interview Questions

### What does Promise.all() return?

A Promise that fulfills with an array of input results when all inputs fulfill, or rejects if any input rejects.

### Does Promise.all preserve completion order?

No. It preserves input order in the returned results.

### Does Promise.all cancel remaining operations after one rejects?

No. The aggregate Promise rejects, but other underlying operations are not automatically cancelled.

### When should Promise.all be used?

When multiple asynchronous operations are independent and you need all successful results before continuing.

### Is Promise.all suitable for dependent operations?

Not when a later task requires the result of an earlier task before it can even start.

---

# Key Takeaways

- `Promise.all()` coordinates multiple async results.
- It fulfills only when all inputs fulfill.
- Results preserve input order.
- It rejects when any input rejects.
- Fail-fast rejection does not automatically cancel other operations.
- Use it for independent async operations.
- Do not use it for steps that depend on previous results.
- Non-Promise values are accepted.
- Local catches can intentionally convert individual failures into fallbacks.
- `Promise.all()` does not create CPU parallelism.
