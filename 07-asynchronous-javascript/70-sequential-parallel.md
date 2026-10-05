# Lesson 70 — Sequential vs Parallel Async Execution

One of the most important async performance skills is deciding whether operations should run:

- sequentially
- concurrently

People often call the second case "parallel" in everyday code discussions, but a more precise term for Promise-based I/O is usually **concurrent async execution**.

The key question is:

> Does one operation depend on the result of another?

---

## 1. Sequential Execution

```js
async function load() {
  const user =
    await getUser();

  const orders =
    await getOrders(
      user.id
    );

  return orders;
}
```

This must be sequential because `getOrders()` needs `user.id`.

Flow:

```text
getUser
↓
wait
↓
user available
↓
getOrders(user.id)
↓
wait
```

This is correct sequencing.

---

# 2. Unnecessary Sequential Execution

Consider:

```js
async function load() {
  const profile =
    await getProfile();

  const stats =
    await getStats();

  const jobs =
    await getJobs();
}
```

If these operations are independent, this is slower than necessary.

Flow:

```text
profile starts
↓
wait
↓
stats starts
↓
wait
↓
jobs starts
↓
wait
```

---

# 3. Start Together with Promise.all

```js
async function load() {
  const [
    profile,
    stats,
    jobs,
  ] =
    await Promise.all([
      getProfile(),
      getStats(),
      getJobs(),
    ]);
}
```

All three operations are initiated before waiting for the group.

---

# 4. Timing Example

Suppose:

```text
profile → 2 sec
stats   → 1 sec
jobs    → 3 sec
```

Sequential:

```text
2 + 1 + 3
≈ 6 sec
```

Concurrent start:

```text
max(2, 1, 3)
≈ 3 sec
```

Ignoring overhead.

---

# 5. Concurrency Is Not CPU Parallelism

This is important.

```js
await Promise.all([
  fetchA(),
  fetchB(),
  fetchC(),
]);
```

allows I/O operations to overlap.

But:

```js
Promise.all([
  Promise.resolve(
    heavyCpuTaskA()
  ),
  Promise.resolve(
    heavyCpuTaskB()
  ),
]);
```

does not make those CPU-heavy synchronous functions run on separate threads.

They execute synchronously before being wrapped.

---

# 6. Start Promises Before Awaiting

This also creates concurrency:

```js
async function load() {
  const profilePromise =
    getProfile();

  const statsPromise =
    getStats();

  const profile =
    await profilePromise;

  const stats =
    await statsPromise;
}
```

Both operations started first.

However, `Promise.all()` often communicates the intent more clearly.

---

# 7. Dependent vs Independent Decision

Use this decision:

```text
Does B need A's result?
        │
      YES
        ↓
   sequential

        │
       NO
        ↓
start together
Promise.all
```

This simple question prevents many performance mistakes.

---

# 8. Mixed Dependency Example

Suppose:

1. user and config are independent
2. orders need user.id
3. recommendations need user.id
4. orders and recommendations are independent after user exists

Good structure:

```js
async function loadPage() {
  const [
    user,
    config,
  ] =
    await Promise.all([
      getUser(),
      getConfig(),
    ]);

  const [
    orders,
    recommendations,
  ] =
    await Promise.all([
      getOrders(user.id),
      getRecommendations(
        user.id
      ),
    ]);

  return {
    user,
    config,
    orders,
    recommendations,
  };
}
```

This respects dependencies without creating unnecessary waits.

---

# 9. Sequential Loop

```js
for (const id of ids) {
  const user =
    await getUser(id);

  console.log(user);
}
```

This processes one ID at a time.

Use this when:

- order matters
- rate limiting matters
- each step depends on previous state
- you intentionally want one request at a time

---

# 10. Concurrent Loop Pattern

If independent:

```js
const promises =
  ids.map((id) => {
    return getUser(id);
  });

const users =
  await Promise.all(
    promises
  );
```

Or shorter:

```js
const users =
  await Promise.all(
    ids.map(getUser)
  );
```

All requests begin close together.

---

# 11. Danger of Unlimited Concurrency

If:

```js
ids.length === 10000
```

then:

```js
await Promise.all(
  ids.map(getUser)
);
```

may start thousands of operations at once.

That can cause:

- rate-limit errors
- memory pressure
- connection exhaustion
- server overload

So:

> concurrent does not mean unlimited.

---

# 12. Concurrency Limits

For large batches, use controlled concurrency.

Conceptually:

```text
1000 tasks
↓
run 5 or 10 at a time
↓
when one finishes
start next
```

Libraries can help with this, or you can build a worker-pool pattern.

The important design lesson is:

> choose an appropriate concurrency level.

---

# 13. Sequential Can Be Correct for Rate Limits

```js
for (const item of items) {
  await sendRequest(
    item
  );
}
```

This may be intentionally slower but safer if an API permits only one request at a time.

Performance is not the only requirement.

---

# 14. Promise.all Error Behavior

```js
await Promise.all([
  taskA(),
  taskB(),
  taskC(),
]);
```

If one rejects, the `await` throws.

But the other already-started operations are not automatically cancelled.

This connects directly to Lesson 66.

---

# 15. Partial Failure Requirements

If you need every result regardless of failures:

```js
const results =
  await Promise.allSettled([
    taskA(),
    taskB(),
    taskC(),
  ]);
```

This changes the error policy while keeping concurrent execution.

---

# 16. Parallel-Looking but Actually Sequential

Common mistake:

```js
const a =
  await getA();

const b =
  await getB();

const c =
  await getC();
```

Developers sometimes think:

> all are async, so all run together.

No.

Each `await` pauses that function before the next call occurs.

This is sequential.

---

# 17. Correct Concurrent Version

```js
const [
  a,
  b,
  c,
] =
  await Promise.all([
    getA(),
    getB(),
    getC(),
  ]);
```

Now each function is called before the collective await completes.

---

# 18. Data Dependency Example

Bad attempt:

```js
await Promise.all([
  getUser(),
  getOrders(user.id),
]);
```

This cannot work if `user` is not known yet.

Correct:

```js
const user =
  await getUser();

const orders =
  await getOrders(
    user.id
  );
```

Dependency dictates sequencing.

---

# 19. Waterfall Problem

A request waterfall looks like:

```text
Request A
████

Request B
    ████

Request C
        ████
```

If B and C did not depend on earlier results, this is unnecessary latency.

Concurrent:

```text
A ████
B ███
C █████
```

Total time is closer to the slowest request rather than the sum.

---

# 20. React / Next.js Connection

Server-side data loading often benefits from concurrency.

Bad:

```js
const user =
  await getUser();

const jobs =
  await getJobs();

const stats =
  await getStats();
```

If independent:

```js
const [
  user,
  jobs,
  stats,
] =
  await Promise.all([
    getUser(),
    getJobs(),
    getStats(),
  ]);
```

This can significantly reduce server-render latency.

---

# 21. But Avoid Blind Promise.all

Do not mechanically replace every sequence with `Promise.all()`.

Check:

- dependencies
- ordering
- resource limits
- failure policy
- cancellation
- rate limits

Correctness comes before speed.

---

# 22. Combined Real-World Example

```js
async function loadDashboard(
  userId
) {
  const [
    profile,
    settings,
  ] =
    await Promise.all([
      getProfile(userId),
      getSettings(userId),
    ]);

  const [
    jobs,
    applications,
  ] =
    await Promise.all([
      getJobs(
        profile.skills
      ),
      getApplications(
        profile.id
      ),
    ]);

  return {
    profile,
    settings,
    jobs,
    applications,
  };
}
```

This structure:

- runs independent work together
- keeps dependent phases sequential
- avoids unnecessary waterfalls

---

# 23. Interview Questions

### What is the difference between sequential and concurrent async execution?

Sequential execution waits for one operation before starting the next. Concurrent execution starts independent operations before waiting for all results.

### Does Promise.all make JavaScript multi-threaded?

No.

### When should you not use Promise.all?

When operations depend on previous results, when concurrency must be limited, or when the desired failure policy differs.

### Why can await in a loop be slow?

Because each iteration may wait before the next async operation starts.

---

# Final Section 7 Mental Model

```text
Callbacks
↓
control-flow nesting problem
↓
Promises
↓
standard future result
↓
then/catch/finally
↓
chaining + propagation
↓
Promise combinators
↓
coordinate multiple results
↓
async/await
↓
clean Promise syntax
↓
try/catch
↓
clean error handling
↓
sequential vs concurrent
↓
correctness + performance
```

---

# Key Takeaways

- Sequential execution is required for true dependencies.
- Independent async work should often start together.
- `Promise.all()` is the common coordination tool.
- Promise concurrency is not CPU parallelism.
- `await` can accidentally create request waterfalls.
- Large batches may require concurrency limits.
- Rate limits and failure requirements influence execution strategy.
- Optimize async flow only after understanding dependencies.
