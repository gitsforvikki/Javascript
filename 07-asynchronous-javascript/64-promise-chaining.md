# Lesson 64 — Promise Chaining

Promise chaining is where Promises become much more powerful than nested callbacks.

The most important rule is:

> Every call to `.then()` returns a **new Promise**.

The value returned from one handler determines what the next Promise in the chain does.

---

## 1. Basic Chain

```js
Promise.resolve(10)
  .then((value) => {
    return value * 2;
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

```text
20
```

Flow:

```text
Promise fulfills 10
↓
first then gets 10
↓
returns 20
↓
new Promise fulfills 20
↓
second then gets 20
```

---

# 2. then() Returns a New Promise

```js
const p1 =
  Promise.resolve(10);

const p2 =
  p1.then((value) => {
    return value * 2;
  });

console.log(
  p1 === p2
);
```

Output:

```text
false
```

`p2` is a new Promise.

This is the foundation of chaining.

---

# 3. Returning a Normal Value

```js
Promise.resolve(5)
  .then((value) => {
    return value + 5;
  })
  .then((value) => {
    console.log(value);
  });
```

The first handler returns:

```text
10
```

Therefore the next Promise fulfills with `10`.

Rule:

> Returning a normal value from a `.then()` handler fulfills the new Promise with that value.

---

# 4. Returning Nothing

```js
Promise.resolve("A")
  .then((value) => {
    console.log(value);
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

```text
A
undefined
```

Why?

The first handler does not explicitly return anything.

JavaScript returns:

```js
undefined
```

So the next Promise fulfills with `undefined`.

---

# 5. The Most Common Mistake — Forgetting return

Wrong:

```js
getUser()
  .then((user) => {
    getOrders(user.id);
  })
  .then((orders) => {
    console.log(orders);
  });
```

The first handler does not return `getOrders(...)`.

So the next handler receives:

```text
undefined
```

Correct:

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

---

# 6. Returning Another Promise

This is the most important chaining behavior.

```js
Promise.resolve(10)
  .then((value) => {
    return new Promise(
      (resolve) => {
        setTimeout(() => {
          resolve(
            value * 2
          );
        }, 1000);
      }
    );
  })
  .then((value) => {
    console.log(value);
  });
```

The next `.then()` waits for the returned Promise.

Mental model:

```text
handler returns Promise
        ↓
chain adopts/waits for it
        ↓
returned Promise settles
        ↓
next handler continues
```

---

# 7. Flattening Nested Async Work

Callback-like nesting:

```js
getUser()
  .then((user) => {
    getOrders(
      user.id
    ).then(
      (orders) => {
        console.log(
          orders
        );
      }
    );
  });
```

This works but creates nested Promises.

Better:

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

This is called **flattening the chain**.

---

# 8. Dependent Async Sequence

Suppose:

```js
getUser()
  .then((user) => {
    return getOrders(
      user.id
    );
  })
  .then((orders) => {
    return getOrderDetails(
      orders[0].id
    );
  })
  .then((details) => {
    console.log(details);
  });
```

Each step waits for the Promise returned by the previous step.

This is far easier to follow than callback nesting.

---

# 9. Handler Return Rules

Inside `.then()`, four outcomes matter.

### Return normal value

```js
return 10;
```

Next Promise:

```text
fulfilled with 10
```

### Return Promise

```js
return fetchData();
```

Next Promise:

```text
adopts returned Promise
```

### Throw error

```js
throw new Error("Failed");
```

Next Promise:

```text
rejected
```

### Return nothing

```js
return;
```

Next Promise:

```text
fulfilled with undefined
```

This table is central to Promise understanding.

---

# 10. Returning Promise.resolve()

```js
Promise.resolve(1)
  .then((value) => {
    return Promise.resolve(
      value + 1
    );
  })
  .then((value) => {
    console.log(value);
  });
```

Output:

```text
2
```

The chain waits for/adopts the returned Promise.

---

# 11. Thenable Adoption

Promises can also adopt compatible "thenable" objects.

At a practical learning level:

> If a handler returns a Promise-like async result, the chain waits for that result rather than nesting a Promise inside a Promise.

You usually work with real Promises from APIs such as `fetch()`.

---

# 12. Promise Chains Do Not Mutate Earlier Promises

```js
const original =
  Promise.resolve(10);

const doubled =
  original.then(
    (value) => {
      return value * 2;
    }
  );
```

`original` still represents `10`.

`doubled` represents the transformed result `20`.

Think functionally:

```text
original Promise
      ↓ then
new Promise
      ↓ then
another new Promise
```

---

# 13. Branching vs Chaining

Branching:

```js
const promise =
  Promise.resolve(10);

promise.then(
  (value) => {
    console.log(
      value + 1
    );
  }
);

promise.then(
  (value) => {
    console.log(
      value + 100
    );
  }
);
```

Both handlers receive `10`.

Chaining:

```js
promise
  .then((value) => {
    return value + 1;
  })
  .then((value) => {
    console.log(
      value + 100
    );
  });
```

Second handler receives `11`.

---

# 14. Nested Promise Anti-Pattern

Avoid:

```js
getUser()
  .then((user) => {
    return getOrders(
      user.id
    ).then(
      (orders) => {
        return getDetails(
          orders[0].id
        );
      }
    );
  });
```

Prefer:

```js
getUser()
  .then((user) => {
    return getOrders(
      user.id
    );
  })
  .then((orders) => {
    return getDetails(
      orders[0].id
    );
  });
```

Keep the chain flat whenever the tasks are sequentially dependent.

---

# 15. Keeping Previous Data

Sometimes later steps need earlier values.

Option 1:

```js
let currentUser;

getUser()
  .then((user) => {
    currentUser = user;

    return getOrders(
      user.id
    );
  })
  .then((orders) => {
    console.log(
      currentUser,
      orders
    );
  });
```

But external mutable variables are often not ideal.

Better return combined data:

```js
getUser()
  .then((user) => {
    return getOrders(
      user.id
    ).then(
      (orders) => {
        return {
          user,
          orders,
        };
      }
    );
  })
  .then(
    ({ user, orders }) => {
      console.log(
        user,
        orders
      );
    }
  );
```

Or later use `async/await`, which often makes this simpler.

---

# 16. Fetch Chain Example

```js
fetch("/api/users")
  .then((response) => {
    return response.json();
  })
  .then((users) => {
    console.log(users);
  });
```

Why two `.then()` calls?

Because:

```js
fetch(...)
```

returns a Promise of a `Response`.

And:

```js
response.json()
```

returns another Promise representing body parsing.

So:

```text
fetch Promise
↓
Response
↓
response.json Promise
↓
parsed JavaScript value
```

---

# 17. Returning vs Just Calling

Compare:

```js
.then(() => {
  doAsyncTask();
})
```

with:

```js
.then(() => {
  return doAsyncTask();
})
```

First:

```text
chain does not wait
for doAsyncTask
```

Second:

```text
chain waits for
doAsyncTask
```

This single `return` is one of the most important Promise-chaining details.

---

# 18. Promise Chaining Timeline

```js
Promise.resolve(1)
  .then((x) => {
    return x + 1;
  })
  .then((x) => {
    return x * 10;
  })
  .then(console.log);
```

Flow:

```text
1
↓
+1
↓
2
↓
*10
↓
20
↓
console.log
```

Each stage transforms the result for the next stage.

---

# 19. React / Application Connection

Promise chaining often appears in:

- fetch flows
- authentication
- payment flows
- sequential mutations
- API transformations

Example:

```js
login(credentials)
  .then((session) => {
    return loadProfile(
      session.userId
    );
  })
  .then((profile) => {
    console.log(profile);
  });
```

Later, `async/await` will express the same flow with synchronous-looking syntax.

---

# Interview Questions

### What does then() return?

A new Promise.

### What happens if a then() handler returns a normal value?

The new Promise fulfills with that value.

### What happens if it returns another Promise?

The new Promise adopts/waits for that returned Promise.

### What happens if the handler throws?

The Promise returned by `.then()` becomes rejected.

### Why is forgetting return a common bug?

Because the chain cannot wait for or receive the result of an async operation that was called but not returned.

---

# Key Takeaways

- Every `.then()` returns a new Promise.
- Promise chains transform values step by step.
- Returning a value fulfills the next Promise with that value.
- Returning another Promise makes the chain wait for it.
- Throwing rejects the next Promise.
- Returning nothing produces `undefined`.
- Always return dependent async Promises from handlers.
- Flat chains are usually easier to read than nested Promises.
- Branching from one Promise is different from chaining.
