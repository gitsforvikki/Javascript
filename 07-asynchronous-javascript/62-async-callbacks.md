# Lesson 62 — Asynchronous Callbacks and Callback Hell

Before Promises became common, callbacks were the primary way to continue work after an asynchronous operation finished.

Callbacks are still important today because they appear in:

- timers
- DOM events
- Node.js APIs
- libraries
- custom utility functions

To understand Promises properly, first understand the problem they were designed to improve.

---

## 1. What Is a Callback?

A callback is a function passed to another function so that it can be invoked later.

Example:

```js
function greet(name) {
  console.log(
    `Hello ${name}`
  );
}

function processUser(callback) {
  const name = "Vikash";

  callback(name);
}

processUser(greet);
```

Here:

```js
greet
```

is passed as a callback.

---

# 2. Synchronous Callback vs Asynchronous Callback

Not every callback is asynchronous.

Example:

```js
[1, 2, 3].map((number) => {
  return number * 2;
});
```

The `map` callback runs synchronously.

But:

```js
setTimeout(() => {
  console.log("later");
}, 1000);
```

uses an asynchronous callback.

So:

> callback does not automatically mean async.

---

# 3. Why Async Callbacks Are Useful

Suppose we simulate loading a user:

```js
function getUser(callback) {
  setTimeout(() => {
    const user = {
      id: 1,
      name: "Vikash",
    };

    callback(user);
  }, 1000);
}
```

Usage:

```js
getUser((user) => {
  console.log(user);
});
```

The function cannot simply return the async result immediately because the result becomes available later.

---

# 4. Why This Does Not Work

```js
function getUser() {
  let user;

  setTimeout(() => {
    user = {
      id: 1,
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

Timeline:

```text
getUser()
↓
user = undefined
↓
timer registered
↓
return user
↓
undefined returned

later
↓
timer callback runs
↓
user receives object
```

The function returned before the asynchronous work finished.

---

# 5. Callback Continuation Model

Instead of trying to return the future value, we pass the next step as a callback.

```js
function getUser(callback) {
  setTimeout(() => {
    const user = {
      id: 1,
      name: "Vikash",
    };

    callback(user);
  }, 1000);
}
```

Mental model:

```text
start getUser
↓
async operation begins
↓
function returns
↓
operation completes later
↓
callback receives result
↓
next step continues
```

This pattern is often called **continuation passing**.

---

# 6. Multiple Dependent Async Operations

Suppose we need:

1. user
2. user's orders
3. details of first order

Example:

```js
function getUser(callback) {
  setTimeout(() => {
    callback({
      id: 1,
      name: "Vikash",
    });
  }, 500);
}

function getOrders(
  userId,
  callback
) {
  setTimeout(() => {
    callback([
      {
        id: 101,
        userId,
      },
    ]);
  }, 500);
}

function getOrderDetails(
  orderId,
  callback
) {
  setTimeout(() => {
    callback({
      id: orderId,
      amount: 1200,
    });
  }, 500);
}
```

Now:

```js
getUser((user) => {
  getOrders(
    user.id,
    (orders) => {
      getOrderDetails(
        orders[0].id,
        (details) => {
          console.log(
            details
          );
        }
      );
    }
  );
});
```

This works.

But readability is getting worse.

---

# 7. What Is Callback Hell?

Callback hell refers to deeply nested callback-based asynchronous code that becomes difficult to:

- read
- debug
- maintain
- extend
- handle errors in

Example shape:

```js
step1((result1) => {
  step2(
    result1,
    (result2) => {
      step3(
        result2,
        (result3) => {
          step4(
            result3,
            () => {
              // ...
            }
          );
        }
      );
    }
  );
});
```

This creates the famous:

```text
pyramid of doom
```

---

# 8. Why Callback Hell Is More Than Indentation

The real problem is not merely ugly indentation.

It creates harder control flow.

Questions become difficult:

```text
What happens on error?

Which callback owns the next step?

Can we reuse one step?

What if two branches run?

What if callback fires twice?

What if callback never fires?
```

The problem is **control flow management**.

---

# 9. Inversion of Control

With callback APIs, you often hand your continuation to another function:

```js
thirdPartyTask(() => {
  console.log("continue");
});
```

Now you trust `thirdPartyTask` to call your callback:

- once
- at the right time
- with correct arguments
- not too early
- not too late
- not twice

This is called **inversion of control**.

Promises improve this by giving you a standardized object representing future completion.

---

# 10. Error-First Callback Pattern

Node.js traditionally uses error-first callbacks.

Example:

```js
function readData(callback) {
  setTimeout(() => {
    const error = null;
    const data = "Hello";

    callback(
      error,
      data
    );
  }, 500);
}
```

Usage:

```js
readData(
  (error, data) => {
    if (error) {
      console.error(error);
      return;
    }

    console.log(data);
  }
);
```

Convention:

```text
callback(error, result)
```

---

# 11. Error Handling Becomes Repetitive

For several dependent operations:

```js
getUser((error, user) => {
  if (error) {
    handleError(error);
    return;
  }

  getOrders(
    user.id,
    (error, orders) => {
      if (error) {
        handleError(error);
        return;
      }

      getOrderDetails(
        orders[0].id,
        (error, details) => {
          if (error) {
            handleError(error);
            return;
          }

          console.log(
            details
          );
        }
      );
    }
  );
});
```

The actual business flow becomes surrounded by repeated error-handling code.

Promises make propagation easier.

---

# 12. Named Functions Can Improve Callback Code

Instead of nesting everything:

```js
function handleUser(user) {
  getOrders(
    user.id,
    handleOrders
  );
}

function handleOrders(orders) {
  getOrderDetails(
    orders[0].id,
    handleDetails
  );
}

function handleDetails(details) {
  console.log(details);
}

getUser(handleUser);
```

This improves indentation.

But the underlying callback-based control flow still exists.

---

# 13. Callback Can Run More Than Once

A badly designed API could do:

```js
function task(callback) {
  callback("first");
  callback("second");
}
```

The consumer may not expect that.

A Promise, by contrast, can settle only once.

This is one major reliability benefit of Promises.

---

# 14. Callback Can Never Run

Another bad implementation:

```js
function task(callback) {
  // forgot to call callback
}
```

The caller may wait forever conceptually.

Promises do not magically solve every bug, but they provide a more standardized lifecycle:

```text
pending
↓
fulfilled
or
rejected
```

---

# 15. Callback-Based API Example

```js
function fetchProduct(
  id,
  callback
) {
  setTimeout(() => {
    if (!id) {
      callback(
        new Error(
          "Missing id"
        )
      );

      return;
    }

    callback(
      null,
      {
        id,
        name: "Laptop",
      }
    );
  }, 500);
}
```

Usage:

```js
fetchProduct(
  10,
  (error, product) => {
    if (error) {
      console.error(error);
      return;
    }

    console.log(product);
  }
);
```

This works, but Promise syntax will make composition easier.

---

# 16. Callback vs Promise Preview

Callback:

```js
getUser((user) => {
  getOrders(
    user.id,
    (orders) => {
      console.log(orders);
    }
  );
});
```

Promise style:

```js
getUser()
  .then((user) => {
    return getOrders(
      user.id
    );
  })
  .then((orders) => {
    console.log(orders);
  });
```

The Promise version grows vertically instead of nesting inward.

---

# 17. React Connection

Callbacks remain common in React:

```jsx
<button
  onClick={() => {
    console.log(
      "clicked"
    );
  }}
>
  Click
</button>
```

This callback is event-driven.

Async application flows, however, are usually handled with:

- Promises
- async/await
- query libraries

rather than deeply nested async callbacks.

---

# Interview Questions

### What is a callback?

A function passed to another function to be invoked by that function later or during its execution.

### Are all callbacks asynchronous?

No. Array callbacks like `map()` are commonly synchronous.

### What is callback hell?

A deeply nested callback structure that makes asynchronous control flow and error handling difficult to understand and maintain.

### What is inversion of control in callback-based async code?

You give another API responsibility for invoking your continuation correctly.

---

# Key Takeaways

- A callback is simply a function passed to another function.
- Callbacks can be synchronous or asynchronous.
- Async callbacks let later work continue when a result becomes available.
- Dependent callbacks can create deep nesting.
- Callback hell is mainly a control-flow and error-handling problem.
- Error-first callbacks are common in older Node.js APIs.
- Named callbacks improve readability but do not fully solve composition issues.
- Promises provide a standardized representation of future completion.
