# Lesson 156 — Implement `Promise.all`

`Promise.all()` is one of the most important async implementation questions in JavaScript interviews.

It tests whether you understand:

- Promises
- iterable inputs
- result ordering
- concurrent settlement
- fail-fast rejection
- non-Promise values
- empty input
- asynchronous coordination

---

## 1. Native Promise.all

```js
const result =
  await Promise.all([
    Promise.resolve(10),
    Promise.resolve(20),
    Promise.resolve(30),
  ]);
```

Result:

```js
[10, 20, 30]
```

---

## 2. Main Behavior

`Promise.all()`:

```text
takes an iterable
↓
waits for every input
↓
preserves input order
↓
resolves with array
```

But:

```text
if any input rejects
↓
outer Promise rejects immediately
```

---

## 3. Result Order vs Completion Order

This is very important.

```js
const slow =
  new Promise(
    (resolve) =>
      setTimeout(
        () =>
          resolve("A"),
        100
      )
  );

const fast =
  new Promise(
    (resolve) =>
      setTimeout(
        () =>
          resolve("B"),
        10
      )
  );

Promise.all([
  slow,
  fast,
]);
```

Even though `fast` finishes first, result is:

```js
["A", "B"]
```

because output preserves input order.

---

## 4. Non-Promise Values

```js
Promise.all([
  10,
  Promise.resolve(
    20
  ),
  30,
]);
```

resolves to:

```js
[10, 20, 30]
```

Why?

Each input is normalized through:

```js
Promise.resolve(
  value
)
```

---

## 5. Thenables

`Promise.resolve()` also assimilates thenables.

So a good implementation should not check only:

```js
value instanceof Promise
```

That would miss Promise-like objects.

Use:

```js
Promise.resolve(value)
```

---

## 6. First Basic Implementation

```js
function customPromiseAll(
  promises
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      const results = [];

      let completed = 0;

      promises.forEach(
        (
          promise,
          index
        ) => {
          Promise.resolve(
            promise
          )
            .then(
              (value) => {
                results[index] =
                  value;

                completed++;

                if (
                  completed ===
                  promises.length
                ) {
                  resolve(
                    results
                  );
                }
              }
            )
            .catch(
              reject
            );
        }
      );
    }
  );
}
```

Good start, but it has issues:

- assumes array
- empty array never resolves
- `.length` may not exist for generic iterable

---

## 7. Empty Input

Native behavior:

```js
Promise.all([]);
```

resolves with:

```js
[]
```

So our implementation must explicitly handle empty input.

---

## 8. Convert Iterable to Array

Use:

```js
const items =
  Array.from(
    iterable
  );
```

Now we support iterable inputs such as:

- arrays
- Sets
- other finite iterables

---

## 9. Interview-Ready Implementation

```js
function customPromiseAll(
  iterable
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      const items =
        Array.from(
          iterable
        );

      const length =
        items.length;

      if (
        length === 0
      ) {
        resolve([]);
        return;
      }

      const results =
        new Array(
          length
        );

      let completed = 0;

      items.forEach(
        (
          item,
          index
        ) => {
          Promise.resolve(
            item
          ).then(
            (value) => {
              results[index] =
                value;

              completed++;

              if (
                completed ===
                length
              ) {
                resolve(
                  results
                );
              }
            },
            reject
          );
        }
      );
    }
  );
}
```

---

## 10. Why results[index] Instead of push()?

Wrong:

```js
results.push(
  value
);
```

This stores completion order.

Correct:

```js
results[index] =
  value;
```

This preserves input order.

---

## 11. Why Use Promise.resolve?

```js
Promise.resolve(
  item
)
```

handles:

- real Promises
- plain values
- thenables

This matches native normalization behavior much better.

---

## 12. Fail-Fast Rejection

```js
const p1 =
  Promise.resolve(
    "A"
  );

const p2 =
  Promise.reject(
    new Error(
      "Failed"
    )
  );

const p3 =
  new Promise(
    (resolve) =>
      setTimeout(
        () =>
          resolve("C"),
        1000
      )
  );
```

```js
Promise.all([
  p1,
  p2,
  p3,
]);
```

rejects when `p2` rejects.

It does not wait to produce a successful result array.

---

## 13. Important: Fail Fast Does Not Cancel Others

This is a very common interview trap.

When one Promise rejects:

```text
Promise.all rejects
```

but other asynchronous operations may still continue.

`Promise.all` does not automatically cancel them.

Cancellation requires something like:

- AbortController
- cooperative cancellation
- task-specific cancellation API

---

## 14. Promise.all vs Promise.allSettled

```text
Promise.all
→ reject on first rejection

Promise.allSettled
→ wait for every input
→ report each outcome
```

---

## 15. Promise.all vs Promise.race

```text
Promise.all
→ wait for all success

Promise.race
→ settle when first input settles
```

---

## 16. Promise.all vs Promise.any

```text
Promise.all
→ all must fulfill

Promise.any
→ first fulfillment wins
→ rejects only if all reject
```

---

## 17. Example with Mixed Values

```js
customPromiseAll([
  1,
  Promise.resolve(2),
  new Promise(
    (resolve) =>
      setTimeout(
        () =>
          resolve(3),
        20
      )
  ),
]).then(
  console.log
);
```

Result:

```js
[1, 2, 3]
```

---

## 18. Rejection Example

```js
customPromiseAll([
  Promise.resolve(1),
  Promise.reject(
    "failed"
  ),
  Promise.resolve(3),
]).catch(
  console.error
);
```

Outer Promise rejects with:

```text
failed
```

---

## 19. Empty Set Example

```js
customPromiseAll(
  new Set()
).then(
  console.log
);
```

Result:

```js
[]
```

because Set is iterable.

---

## 20. Common Wrong Implementation

```js
function customPromiseAll(
  promises
) {
  return new Promise(
    async (
      resolve,
      reject
    ) => {
      const result = [];

      for (
        const promise
        of promises
      ) {
        try {
          result.push(
            await promise
          );
        } catch (
          error
        ) {
          reject(error);
          return;
        }
      }

      resolve(result);
    }
  );
}
```

This may preserve order, but it waits **sequentially**.

Native `Promise.all` observes all inputs concurrently.

---

## 21. Sequential vs Concurrent

Sequential:

```text
wait P1
↓
wait P2
↓
wait P3
```

Promise.all style:

```text
start/observe P1
start/observe P2
start/observe P3
↓
wait for all
```

This distinction is critical.

---

## 22. Async Executor Anti-Pattern

Avoid:

```js
new Promise(
  async (
    resolve,
    reject
  ) => {
    // ...
  }
);
```

The Promise constructor already expects you to manage resolve/reject directly.

An async executor can introduce confusing error behavior.

---

## 23. Iterable Error

```js
Array.from(
  iterable
)
```

can throw if the input is not iterable/array-like in an acceptable way.

A production-grade implementation should let such synchronous errors reject the returned Promise.

One way:

```js
function customPromiseAll(
  iterable
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      let items;

      try {
        items =
          Array.from(
            iterable
          );
      } catch (
        error
      ) {
        reject(error);
        return;
      }

      // continue...
    }
  );
}
```

---

## 24. More Robust Version

```js
function customPromiseAll(
  iterable
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      let items;

      try {
        items =
          Array.from(
            iterable
          );
      } catch (
        error
      ) {
        reject(error);
        return;
      }

      if (
        items.length ===
        0
      ) {
        resolve([]);
        return;
      }

      const results =
        new Array(
          items.length
        );

      let remaining =
        items.length;

      items.forEach(
        (
          item,
          index
        ) => {
          Promise.resolve(
            item
          ).then(
            (value) => {
              results[index] =
                value;

              remaining--;

              if (
                remaining ===
                0
              ) {
                resolve(
                  results
                );
              }
            },
            reject
          );
        }
      );
    }
  );
}
```

---

## 25. Complexity

For `n` inputs:

```text
Coordination time:
O(n)

Result space:
O(n)
```

Actual completion time depends on the slowest fulfilled operation, unless a rejection happens first.

---

## Interview Explanation

A strong answer:

> I convert the iterable to an array, handle the empty case, create a results array with matching length, and normalize every input using `Promise.resolve`. Each Promise writes to its original index so completion order does not affect result order. I count completions and resolve when all fulfill. Any rejection immediately rejects the outer Promise, but it does not cancel the remaining underlying operations.

---

## Section 18 Mental Model So Far

```text
find
→ first matching value

call/apply
→ explicit this binding
→ invoke now

bind
→ explicit this binding
→ invoke later

Promise.all
→ coordinate many async values
→ preserve order
→ fail fast
```

---

## Key Takeaways

- `Promise.all` accepts an iterable.
- Result order follows input order.
- Completion order does not matter.
- Plain values and thenables are normalized with `Promise.resolve`.
- Empty input resolves to `[]`.
- First rejection rejects the outer Promise.
- Other operations are not automatically cancelled.
- Avoid sequential `await` loops when implementing Promise.all behavior.
