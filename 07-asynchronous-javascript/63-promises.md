# Lesson 63 — Promises

A Promise is one of the most important abstractions in asynchronous JavaScript.

The core definition is:

> A Promise is an object representing the eventual completion or failure of an asynchronous operation.

Instead of handing another function your callback directly, you receive a Promise and attach reactions to it.

---

## 1. Promise States

A Promise has three important states:

```text
pending
   ↓
fulfilled
or
rejected
```

### pending

The operation has not completed yet.

### fulfilled

The operation completed successfully.

### rejected

The operation failed.

A Promise that is fulfilled or rejected is called **settled**.

---

# 2. A Promise Settles Only Once

A Promise can transition:

```text
pending → fulfilled
```

or:

```text
pending → rejected
```

But not:

```text
fulfilled → rejected
```

or:

```text
rejected → fulfilled
```

Once settled, its state cannot change.

---

# 3. Creating a Promise

Syntax:

```js
const promise =
  new Promise(
    (
      resolve,
      reject
    ) => {
      // work
    }
  );
```

The function passed to `new Promise()` is called the **executor**.

It receives:

- `resolve`
- `reject`

---

# 4. The Executor Runs Synchronously

This is a very important interview concept.

```js
console.log("A");

const promise =
  new Promise(
    (resolve) => {
      console.log("B");

      resolve("done");
    }
  );

console.log("C");
```

Output:

```text
A
B
C
```

Why?

The Promise executor itself runs immediately and synchronously.

Promise handlers such as `.then()` run asynchronously later.

---

# 5. Resolving a Promise

```js
const promise =
  new Promise(
    (resolve) => {
      setTimeout(() => {
        resolve(
          "Success"
        );
      }, 1000);
    }
  );
```

After the timer completes:

```js
resolve("Success");
```

fulfills the Promise with the value:

```text
"Success"
```

---

# 6. Rejecting a Promise

```js
const promise =
  new Promise(
    (
      resolve,
      reject
    ) => {
      setTimeout(() => {
        reject(
          new Error(
            "Request failed"
          )
        );
      }, 1000);
    }
  );
```

Now the Promise becomes rejected.

---

# 7. Consuming a Promise with then()

```js
promise.then(
  (value) => {
    console.log(value);
  }
);
```

The callback passed to `.then()` runs when the Promise fulfills.

Example:

```js
const promise =
  Promise.resolve(
    "Hello"
  );

promise.then(
  (value) => {
    console.log(value);
  }
);
```

Output later:

```text
Hello
```

---

# 8. then() Is Asynchronous Even for an Already-Fulfilled Promise

```js
console.log("A");

Promise.resolve("done")
  .then(() => {
    console.log("B");
  });

console.log("C");
```

Output:

```text
A
C
B
```

Even though the Promise is already fulfilled, the `.then()` handler does not run synchronously in the current stack.

It is scheduled as a microtask.

You will study microtasks deeply later.

---

# 9. catch()

For rejection:

```js
promise.catch(
  (error) => {
    console.error(
      error
    );
  }
);
```

Example:

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

---

# 10. finally()

```js
promise.finally(() => {
  console.log(
    "finished"
  );
});
```

`finally()` runs after settlement regardless of fulfillment or rejection.

Typical use:

- hide loading spinner
- cleanup
- reset temporary state

---

# 11. A Promise Is Not the Final Value

Suppose:

```js
const result =
  fetch("/api/users");
```

`result` is not the response body.

It is a Promise representing future completion.

Similarly:

```js
const promise =
  Promise.resolve(10);
```

`promise` is a Promise object.

It is not the number `10`.

---

# 12. Promise.resolve()

```js
const promise =
  Promise.resolve(42);
```

This creates a fulfilled Promise.

Use:

```js
promise.then(
  (value) => {
    console.log(value);
  }
);
```

---

# 13. Promise.reject()

```js
const promise =
  Promise.reject(
    new Error(
      "Something failed"
    )
  );
```

This creates a rejected Promise.

---

# 14. resolve() with Another Promise

This is subtle.

```js
const inner =
  Promise.resolve(100);

const outer =
  new Promise(
    (resolve) => {
      resolve(inner);
    }
  );
```

The outer Promise adopts the eventual state/value of `inner`.

Conceptually:

```text
outer resolve(inner)
       ↓
outer follows inner
       ↓
inner fulfills 100
       ↓
outer fulfills 100
```

This behavior is important for promise chaining.

---

# 15. Resolve Does Not Always Mean "Fulfilled Immediately"

If you do:

```js
resolve(otherPromise);
```

the Promise may remain pending while it follows the supplied Promise.

So distinguish:

- resolved
- fulfilled

They are often informally used as synonyms, but technically resolution can involve adopting another Promise's state.

For beginner/interview purposes:

> fulfilled means successfully settled with a value.

---

# 16. Executor Error Automatically Rejects

```js
const promise =
  new Promise(() => {
    throw new Error(
      "Boom"
    );
  });
```

The thrown error causes the Promise to reject.

Equivalent conceptually:

```js
new Promise(
  (
    resolve,
    reject
  ) => {
    try {
      throw new Error(
        "Boom"
      );
    } catch (error) {
      reject(error);
    }
  }
);
```

---

# 17. But Async Throws Inside setTimeout Are Different

Important:

```js
new Promise(
  () => {
    setTimeout(() => {
      throw new Error(
        "Boom"
      );
    }, 1000);
  }
);
```

The Promise constructor cannot automatically catch that later asynchronous throw because the executor has already finished.

Correct:

```js
new Promise(
  (
    resolve,
    reject
  ) => {
    setTimeout(() => {
      try {
        throw new Error(
          "Boom"
        );
      } catch (error) {
        reject(error);
      }
    }, 1000);
  }
);
```

---

# 18. Converting Callback Style to Promise Style

Callback:

```js
function getUser(callback) {
  setTimeout(() => {
    callback({
      id: 1,
      name: "Vikash",
    });
  }, 500);
}
```

Promise:

```js
function getUser() {
  return new Promise(
    (resolve) => {
      setTimeout(() => {
        resolve({
          id: 1,
          name: "Vikash",
        });
      }, 500);
    }
  );
}
```

Usage:

```js
getUser()
  .then((user) => {
    console.log(user);
  });
```

---

# 19. Promise Mental Model

```text
Promise created
     ↓
pending
     ↓
async operation progresses
     ↓
success?
  /       \
yes       no
 ↓         ↓
fulfilled rejected
 ↓         ↓
then      catch
```

---

# 20. Promises Do Not Start Work Because of then()

Common misconception:

> The async operation starts when `.then()` is attached.

Usually false.

Example:

```js
const promise =
  new Promise(
    (resolve) => {
      console.log(
        "executor runs"
      );

      resolve(1);
    }
  );
```

The executor runs immediately when the Promise is created.

`.then()` registers what should happen after fulfillment.

---

# 21. Multiple then() Handlers

```js
const promise =
  Promise.resolve(10);

promise.then(
  (value) => {
    console.log(
      "A",
      value
    );
  }
);

promise.then(
  (value) => {
    console.log(
      "B",
      value
    );
  }
);
```

Both handlers are attached to the same original Promise.

This is different from chaining:

```js
promise
  .then(...)
  .then(...);
```

Chaining creates a sequence of new Promises.

That is Lesson 64.

---

# 22. Promise Value Is Immutable After Settlement

```js
const promise =
  new Promise(
    (
      resolve,
      reject
    ) => {
      resolve("first");
      resolve("second");
      reject("error");
    }
  );
```

The first settlement wins.

The Promise fulfills with:

```text
first
```

Later attempts are ignored.

---

# 23. Fetch Connection

```js
fetch("/api/users")
```

returns a Promise.

Then:

```js
fetch("/api/users")
  .then((response) => {
    return response.json();
  });
```

Important:

```js
response.json()
```

also returns a Promise.

That is why chaining matters.

---

# 24. React Connection

React applications use Promises constantly for:

- API calls
- mutations
- route/data loading
- server actions
- asynchronous form handling

Example:

```js
fetch("/api/jobs")
  .then((response) => {
    return response.json();
  })
  .then((jobs) => {
    console.log(jobs);
  });
```

Understanding Promise states is required before `async/await` makes complete sense.

---

# Interview Questions

### What is a Promise?

An object representing eventual asynchronous completion or failure.

### What are the states of a Promise?

Pending, fulfilled, and rejected.

### Can a Promise settle more than once?

No. The first fulfillment or rejection determines its final state.

### Does the Promise executor run asynchronously?

No. The executor runs synchronously when the Promise is created.

### Are then() handlers synchronous?

No. Promise reactions are scheduled asynchronously as microtasks.

---

# Key Takeaways

- A Promise represents future completion or failure.
- Promises begin pending.
- They settle as fulfilled or rejected.
- Settlement happens only once.
- The Promise executor runs synchronously.
- `.then()` handles fulfillment.
- `.catch()` handles rejection.
- `.finally()` runs after either outcome.
- Promise handlers run asynchronously.
- A Promise object is not the eventual value itself.
- Resolving with another Promise adopts that Promise's eventual state.
