# Lesson 65 — Promise Error Handling

Promise error handling is powerful because errors can propagate through a chain until something handles them.

The most important mental model is:

> A thrown error or rejected Promise turns the current chain path into a rejection.

That rejection travels forward until a rejection handler catches it.

---

## 1. Basic catch()

```js
Promise.reject(
  new Error("Failed")
)
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

`.catch(handler)` handles rejection.

---

# 2. catch() Is Equivalent to then(undefined, handler)

Conceptually:

```js
promise.catch(
  onRejected
);
```

is equivalent to:

```js
promise.then(
  undefined,
  onRejected
);
```

In practice, `.catch()` is clearer.

---

# 3. Throwing Inside then()

```js
Promise.resolve(10)
  .then((value) => {
    throw new Error(
      "Something broke"
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

Output:

```text
Something broke
```

The thrown error rejects the Promise returned by the `.then()`.

---

# 4. Rejection Propagates Through the Chain

```js
Promise.reject(
  new Error("Failed")
)
  .then(() => {
    console.log("A");
  })
  .then(() => {
    console.log("B");
  })
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

The fulfillment handlers are skipped while the chain remains rejected.

Mental model:

```text
rejection
↓
then fulfillment handler skipped
↓
then fulfillment handler skipped
↓
catch found
↓
handle error
```

---

# 5. Error Can Happen in the Middle

```js
Promise.resolve(10)
  .then((value) => {
    return value * 2;
  })
  .then(() => {
    throw new Error(
      "Middle failure"
    );
  })
  .then(() => {
    console.log(
      "will not run"
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

The chain jumps over later fulfillment handlers until a rejection handler is found.

---

# 6. Returning a Rejected Promise

```js
Promise.resolve()
  .then(() => {
    return Promise.reject(
      new Error(
        "Rejected"
      )
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

A returned rejected Promise makes the chain rejected.

---

# 7. catch() Can Recover the Chain

This is extremely important.

```js
Promise.reject(
  new Error("Failed")
)
  .catch((error) => {
    console.log(
      "handled"
    );

    return 100;
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

```text
handled
100
```

Why?

The `catch()` handler returned a normal value.

Therefore the Promise returned by `catch()` fulfills with `100`.

So:

> catch does not permanently mark the whole chain as failed.

It can recover.

---

# 8. Rethrowing an Error

Sometimes you want to log locally but keep the chain rejected.

```js
fetchData()
  .catch((error) => {
    console.error(
      "Logging:",
      error
    );

    throw error;
  })
  .catch((error) => {
    console.log(
      "Higher-level handler"
    );
  });
```

The first `catch` handles the rejection, then throws again.

That creates a new rejection.

---

# 9. Returning Promise.reject()

This also continues rejection:

```js
.catch((error) => {
  return Promise.reject(
    error
  );
});
```

But:

```js
throw error;
```

is often simpler inside a Promise handler.

---

# 10. Error in catch()

```js
Promise.reject(
  new Error("Original")
)
  .catch(() => {
    throw new Error(
      "Catch failed"
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

Output:

```text
Catch failed
```

A `catch()` handler is just another Promise handler.

It can itself return values, return Promises, or throw errors.

---

# 11. finally() Behavior

```js
Promise.resolve("success")
  .finally(() => {
    console.log(
      "cleanup"
    );
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

```text
cleanup
success
```

Normally, `finally()` passes through the previous fulfillment value.

For rejection:

```js
Promise.reject(
  new Error("Failed")
)
  .finally(() => {
    console.log(
      "cleanup"
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

The rejection continues through.

---

# 12. finally() Can Change the Outcome If It Throws

```js
Promise.resolve(
  "success"
)
  .finally(() => {
    throw new Error(
      "Cleanup failed"
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

Output:

```text
Cleanup failed
```

So `finally()` normally preserves the previous outcome, unless the finally callback itself fails or returns a rejected Promise.

---

# 13. Local vs Global-Style Catch Placement

Option A:

```js
step1()
  .then(step2)
  .then(step3)
  .catch(handleError);
```

One final `catch()` can handle failures from any earlier step in the chain.

This is one major advantage over repetitive callback error checks.

---

# 14. Catching Too Early

Consider:

```js
step1()
  .catch(() => {
    return "fallback";
  })
  .then((value) => {
    // chain is fulfilled
  });
```

The early catch transforms failure into success if it returns a normal value.

This may be intentional—or a bug.

Always decide whether the error should:

- be recovered
- be transformed
- be rethrown
- terminate the operation

---

# 15. Fetch Does Not Reject for Every HTTP Error

Important real-world concept:

```js
fetch("/api/users")
```

typically rejects for network-level failures, not simply because the server returned HTTP 404 or 500.

So you often write:

```js
fetch("/api/users")
  .then((response) => {
    if (!response.ok) {
      throw new Error(
        `HTTP ${response.status}`
      );
    }

    return response.json();
  })
  .catch((error) => {
    console.error(error);
  });
```

Throwing converts the bad HTTP response into a rejected Promise chain.

---

# 16. Synchronous Throw Inside Promise Handler

```js
Promise.resolve(
  '{"bad json"'
)
  .then((text) => {
    return JSON.parse(text);
  })
  .catch((error) => {
    console.log(
      "JSON failed"
    );
  });
```

`JSON.parse()` throws synchronously.

Because it throws inside a Promise handler, the returned Promise becomes rejected automatically.

---

# 17. Async Throw Outside the Promise Chain

This is different:

```js
Promise.resolve()
  .then(() => {
    setTimeout(() => {
      throw new Error(
        "Later error"
      );
    }, 1000);
  })
  .catch((error) => {
    console.log(
      "Will this catch?"
    );
  });
```

No.

Why?

The timer callback runs later outside that Promise chain.

The chain already fulfilled with `undefined`.

If you want the chain to represent that timer work, return a Promise:

```js
Promise.resolve()
  .then(() => {
    return new Promise(
      (
        resolve,
        reject
      ) => {
        setTimeout(() => {
          reject(
            new Error(
              "Later error"
            )
          );
        }, 1000);
      }
    );
  })
  .catch((error) => {
    console.log(
      error.message
    );
  });
```

---

# 18. Unhandled Rejections

If a Promise rejects and nothing handles it:

```js
Promise.reject(
  new Error(
    "Unhandled"
  )
);
```

the runtime may report an **unhandled promise rejection**.

This is a serious issue in applications.

Always make sure failures are intentionally handled at an appropriate level.

---

# 19. Error Transformation

Sometimes lower-level errors should become domain errors.

```js
loadUser()
  .catch((error) => {
    throw new Error(
      "Unable to load profile",
      {
        cause: error,
      }
    );
  });
```

This provides clearer application-level meaning while retaining the original cause.

---

# 20. React / Application Connection

Promise error handling appears in:

- API requests
- form submission
- authentication
- payments
- route loading
- mutations

Example:

```js
saveApplication(data)
  .then(() => {
    showSuccess();
  })
  .catch((error) => {
    showError(
      error.message
    );
  })
  .finally(() => {
    setSubmitting(false);
  });
```

This clearly separates:

- success
- failure
- cleanup

---

# 21. Error Propagation Diagram

```text
fulfilled
   ↓
then
   ↓
throws error
   ↓
rejected
   ↓
then success handler skipped
   ↓
catch
   ↓
returns fallback
   ↓
fulfilled again
   ↓
next then
```

This is the mental model to remember.

---

# Interview Questions

### How do errors propagate in a Promise chain?

A rejection or thrown error travels forward through the chain until a rejection handler handles it.

### What happens if catch() returns a normal value?

The Promise returned by `catch()` fulfills with that value, so the chain can continue successfully.

### What happens if catch() throws?

The Promise returned by `catch()` becomes rejected again.

### Does fetch reject automatically for HTTP 404 or 500?

Usually no. You typically check `response.ok` and throw manually for unwanted HTTP statuses.

---

# Key Takeaways

- `.catch()` handles rejected Promise paths.
- Thrown errors inside Promise handlers become rejections.
- Rejections propagate until handled.
- Fulfillment handlers are skipped while the chain is rejected.
- `catch()` can recover the chain by returning a value.
- Rethrowing keeps the chain rejected.
- `finally()` normally preserves the previous outcome.
- Async work not returned into the chain cannot be caught by that chain.
- Unhandled rejections should be avoided.
