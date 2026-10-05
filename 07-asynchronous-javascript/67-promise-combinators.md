# Lesson 67 — `Promise.allSettled()`, `Promise.race()` and `Promise.any()`

You already learned `Promise.all()`.

Now we will compare the other major Promise combinators:

- `Promise.allSettled()`
- `Promise.race()`
- `Promise.any()`

The key question is:

> What condition causes the returned Promise to settle?

---

## 1. Quick Comparison

```text
Promise.all()
→ wait for all success
→ reject if any rejects

Promise.allSettled()
→ wait for every input to settle
→ never fail because one input rejected

Promise.race()
→ first settled input wins
→ fulfillment OR rejection

Promise.any()
→ first fulfilled input wins
→ rejects only if all inputs reject
```

---

# 2. Promise.allSettled()

Syntax:

```js
Promise.allSettled([
  promise1,
  promise2,
  promise3,
]);
```

It waits until **every input settles**.

Settlement means:

- fulfilled
- rejected

Unlike `Promise.all()`, one rejection does not reject the combined result.

---

# 3. allSettled Example

```js
const p1 =
  Promise.resolve("A");

const p2 =
  Promise.reject("B failed");

const p3 =
  Promise.resolve("C");

Promise.allSettled([
  p1,
  p2,
  p3,
]).then((results) => {
  console.log(results);
});
```

Result shape:

```js
[
  {
    status: "fulfilled",
    value: "A",
  },
  {
    status: "rejected",
    reason: "B failed",
  },
  {
    status: "fulfilled",
    value: "C",
  },
]
```

---

# 4. Why allSettled Exists

Imagine a dashboard with independent sections:

- profile
- notifications
- recommendations
- analytics

You may want to show every successful result even if one section fails.

```js
const results =
  await Promise.allSettled([
    loadProfile(),
    loadNotifications(),
    loadRecommendations(),
  ]);
```

Now you can inspect each result independently.

---

# 5. Result Order

Like `Promise.all()`, result order follows **input order**, not completion order.

```text
input:
[A, B, C]

completion:
C, A, B

result array:
[A result, B result, C result]
```

---

# 6. Processing allSettled Results

```js
const results =
  await Promise.allSettled([
    loadProfile(),
    loadJobs(),
    loadStats(),
  ]);

for (const result of results) {
  if (
    result.status ===
    "fulfilled"
  ) {
    console.log(
      result.value
    );
  } else {
    console.error(
      result.reason
    );
  }
}
```

This is useful when partial success is acceptable.

---

# 7. Promise.race()

`Promise.race()` settles as soon as the **first input settles**.

That means:

- first fulfillment can win
- first rejection can also win

Example:

```js
const fast =
  new Promise(
    (resolve) => {
      setTimeout(
        () => resolve("fast"),
        500
      );
    }
  );

const slow =
  new Promise(
    (resolve) => {
      setTimeout(
        () => resolve("slow"),
        2000
      );
    }
  );

Promise.race([
  fast,
  slow,
]).then(console.log);
```

Output:

```text
fast
```

---

# 8. race() Can Reject First

```js
const failure =
  new Promise(
    (
      resolve,
      reject
    ) => {
      setTimeout(
        () =>
          reject(
            new Error(
              "failed first"
            )
          ),
        300
      );
    }
  );

const success =
  new Promise(
    (resolve) => {
      setTimeout(
        () =>
          resolve("success"),
        1000
      );
    }
  );
```

Then:

```js
Promise.race([
  failure,
  success,
]).catch((error) => {
  console.log(
    error.message
  );
});
```

Output:

```text
failed first
```

Because the rejection settled first.

---

# 9. Common race() Use Case — Timeout

Conceptually:

```js
function timeout(ms) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      setTimeout(() => {
        reject(
          new Error(
            "Timed out"
          )
        );
      }, ms);
    }
  );
}
```

Then:

```js
Promise.race([
  fetchData(),
  timeout(5000),
]);
```

Whichever settles first determines the race result.

Important:

> The losing operation is not automatically cancelled.

---

# 10. race() Does Not Cancel Losers

This is crucial.

```js
Promise.race([
  requestA,
  requestB,
]);
```

If `requestA` settles first, `requestB` may still continue.

So:

```text
race winner chosen
≠
other operations cancelled
```

Use cancellation mechanisms such as `AbortController` when actual cancellation is required.

---

# 11. Promise.any()

`Promise.any()` fulfills as soon as the **first input fulfills**.

Rejected inputs are ignored while another Promise may still fulfill.

Example:

```js
const p1 =
  Promise.reject(
    "server 1 failed"
  );

const p2 =
  Promise.resolve(
    "server 2 success"
  );

const p3 =
  Promise.reject(
    "server 3 failed"
  );

Promise.any([
  p1,
  p2,
  p3,
]).then(console.log);
```

Output:

```text
server 2 success
```

---

# 12. race() vs any()

This is one of the most important distinctions.

```text
race()
→ first settled wins
→ rejection can win

any()
→ first fulfilled wins
→ rejections ignored unless all reject
```

---

# 13. Promise.any() When All Reject

```js
Promise.any([
  Promise.reject("A"),
  Promise.reject("B"),
])
  .catch((error) => {
    console.log(
      error instanceof
        AggregateError
    );
  });
```

Output:

```text
true
```

When all inputs reject, `Promise.any()` rejects with an `AggregateError`.

---

# 14. AggregateError

The error can contain the individual rejection reasons.

```js
Promise.any([
  Promise.reject(
    "server A"
  ),
  Promise.reject(
    "server B"
  ),
]).catch((error) => {
  console.log(
    error.errors
  );
});
```

Conceptually:

```js
[
  "server A",
  "server B",
]
```

---

# 15. Real-World any() Use Case

Suppose several mirrors/CDNs can provide the same data:

```js
const result =
  await Promise.any([
    fetchFromMirrorA(),
    fetchFromMirrorB(),
    fetchFromMirrorC(),
  ]);
```

You only need the first successful source.

That is a good use case for `Promise.any()`.

---

# 16. all() vs allSettled()

Use `Promise.all()` when:

> every result is required.

Use `Promise.allSettled()` when:

> you need to inspect every outcome even if some fail.

Example:

```text
checkout:
payment + inventory + order
→ probably fail if critical operation fails

dashboard widgets:
profile + stats + suggestions
→ partial success may be acceptable
```

---

# 17. race() vs any() Example

```js
const failure =
  new Promise(
    (
      resolve,
      reject
    ) => {
      setTimeout(
        () => reject("fail"),
        100
      );
    }
  );

const success =
  new Promise(
    (resolve) => {
      setTimeout(
        () => resolve("ok"),
        500
      );
    }
  );
```

### race

```js
Promise.race([
  failure,
  success,
]);
```

Result:

```text
rejected with "fail"
```

### any

```js
Promise.any([
  failure,
  success,
]);
```

Result:

```text
fulfilled with "ok"
```

---

# 18. Non-Promise Values

Combinators can accept ordinary values too.

```js
Promise.any([
  Promise.reject("fail"),
  42,
]);
```

The ordinary value behaves like an already-fulfilled input.

Result:

```text
42
```

---

# 19. Empty Input Edge Cases

These are useful interview details.

### Promise.all([])

Fulfilled with:

```js
[]
```

### Promise.allSettled([])

Fulfilled with:

```js
[]
```

### Promise.any([])

Rejected with `AggregateError`.

### Promise.race([])

Remains pending forever because there is no input to settle first.

---

# 20. React / Application Connection

Possible application decisions:

```text
Need every API result
→ Promise.all

Need partial dashboard results
→ Promise.allSettled

Need fastest settled result
→ Promise.race

Need first successful provider
→ Promise.any
```

Choosing the right combinator is a control-flow decision, not just syntax.

---

# Interview Questions

### Difference between Promise.all and Promise.allSettled?

`Promise.all()` rejects when any input rejects. `Promise.allSettled()` waits for every input and returns each outcome.

### Difference between Promise.race and Promise.any?

`race()` uses the first settled input, while `any()` uses the first fulfilled input.

### Does Promise.race cancel losing Promises?

No.

### What happens when every Promise passed to Promise.any rejects?

It rejects with an `AggregateError`.

---

# Key Takeaways

- `allSettled()` waits for every outcome.
- `race()` uses the first settlement.
- `any()` uses the first fulfillment.
- `any()` rejects only when all inputs reject.
- `race()` can reject immediately if the first settled input rejects.
- Promise combinators do not automatically cancel remaining operations.
- Choose the combinator based on the business requirement.
