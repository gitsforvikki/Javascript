# Lesson 68 — `async` and `await`

`async` and `await` do not replace Promises.

They are syntax built on top of Promises that make asynchronous code easier to read.

The most important rule is:

> An `async` function always returns a Promise.

And:

> `await` pauses that async function's continuation until the awaited value settles; it does not block the JavaScript thread.

---

## 1. Basic async Function

```js
async function getValue() {
  return 10;
}
```

Call it:

```js
const result =
  getValue();

console.log(result);
```

`result` is a Promise.

Equivalent conceptually:

```js
function getValue() {
  return Promise.resolve(10);
}
```

---

# 2. async Always Returns a Promise

Even this:

```js
async function greet() {
  return "Hello";
}
```

behaves like:

```js
Promise.resolve(
  "Hello"
);
```

Consume it:

```js
greet().then(
  (value) => {
    console.log(value);
  }
);
```

Output:

```text
Hello
```

---

# 3. Returning a Promise from async

```js
async function getData() {
  return Promise.resolve(
    "data"
  );
}
```

The async function does not produce a nested Promise like:

```text
Promise<Promise<data>>
```

Its returned Promise adopts the inner Promise's state.

---

# 4. await Basics

```js
async function example() {
  const value =
    await Promise.resolve(
      10
    );

  console.log(value);
}
```

Output:

```text
10
```

`await` gives you the fulfilled value of the Promise.

---

# 5. await Does Not Block the JavaScript Thread

Consider:

```js
async function run() {
  console.log("A");

  await Promise.resolve();

  console.log("B");
}

run();

console.log("C");
```

Output:

```text
A
C
B
```

Why?

At `await`, the async function yields control.

The continuation after `await` is scheduled later.

JavaScript continues with:

```js
console.log("C");
```

---

# 6. Mental Model of await

```text
async function starts
↓
runs synchronously
↓
reaches await
↓
evaluates awaited expression
↓
function yields
↓
other JavaScript can run
↓
awaited Promise settles
↓
continuation scheduled
↓
async function resumes
```

This is the correct mental model.

---

# 7. await Is Similar to then()

This:

```js
function loadUser() {
  return getUser()
    .then((user) => {
      return getOrders(
        user.id
      );
    })
    .then((orders) => {
      return orders;
    });
}
```

can be written:

```js
async function loadUser() {
  const user =
    await getUser();

  const orders =
    await getOrders(
      user.id
    );

  return orders;
}
```

The logic is still Promise-based.

---

# 8. await a Non-Promise Value

```js
async function test() {
  const value =
    await 42;

  console.log(value);
}
```

Output:

```text
42
```

Conceptually, `await` handles the value like:

```js
Promise.resolve(42)
```

---

# 9. Execution Before First await Is Synchronous

```js
async function test() {
  console.log("inside 1");

  await Promise.resolve();

  console.log("inside 2");
}

console.log("start");

test();

console.log("end");
```

Output:

```text
start
inside 1
end
inside 2
```

This is an important interview pattern.

---

# 10. Awaiting fetch()

```js
async function loadUsers() {
  const response =
    await fetch(
      "/api/users"
    );

  const users =
    await response.json();

  return users;
}
```

Two awaits are needed because:

```text
fetch()
→ Promise<Response>

response.json()
→ Promise<parsed data>
```

---

# 11. async Function Rejection

If an async function throws:

```js
async function fail() {
  throw new Error(
    "Failed"
  );
}
```

Calling:

```js
fail();
```

returns a rejected Promise.

Equivalent conceptually:

```js
return Promise.reject(
  new Error("Failed")
);
```

---

# 12. Rejected await Throws Inside async Function

```js
async function run() {
  await Promise.reject(
    new Error("Boom")
  );

  console.log(
    "not reached"
  );
}
```

The rejection behaves like a thrown error at the `await` point.

So the async function itself becomes rejected unless the error is caught.

---

# 13. async Does Not Mean Function Runs in Another Thread

This is a common mistake.

```js
async function heavy() {
  while (true) {
    // CPU-heavy synchronous loop
  }
}
```

This still blocks the JavaScript thread.

Adding `async` does not move the function to a background thread.

---

# 14. await Does Not Make Synchronous Work Asynchronous

```js
async function run() {
  const result =
    await heavySyncWork();
}
```

If `heavySyncWork()` itself blocks synchronously before returning, the thread is still blocked.

`await` only helps when the expression gives control back through Promise-based async behavior.

---

# 15. Multiple awaits Can Be Sequential

```js
async function load() {
  const a =
    await getA();

  const b =
    await getB();
}
```

This means:

```text
start A
↓
wait for A
↓
start B
↓
wait for B
```

That is sequential.

Sometimes correct, sometimes unnecessarily slow.

Lesson 70 will focus deeply on this.

---

# 16. Start First, Await Later

If independent:

```js
async function load() {
  const promiseA =
    getA();

  const promiseB =
    getB();

  const a =
    await promiseA;

  const b =
    await promiseB;
}
```

Now both async operations start before the first wait completes.

Even cleaner:

```js
const [a, b] =
  await Promise.all([
    getA(),
    getB(),
  ]);
```

---

# 17. await in Loops

```js
for (const id of ids) {
  const user =
    await getUser(id);

  console.log(user);
}
```

This runs sequentially.

That may be correct if:

- order matters
- rate limiting matters
- each step depends on previous work

But it may be inefficient if all requests are independent.

---

# 18. Top-Level await

In modern ES modules, top-level `await` is supported in appropriate module environments.

Example:

```js
const config =
  await loadConfig();
```

But its availability and behavior depend on the module/runtime environment.

For normal application code, you will often use `await` inside async functions.

---

# 19. async Arrow Function

```js
const loadData =
  async () => {
    const data =
      await getData();

    return data;
  };
```

Same Promise semantics apply.

---

# 20. async Methods

```js
const service = {
  async loadUser() {
    const user =
      await getUser();

    return user;
  },
};
```

Again, calling:

```js
service.loadUser();
```

returns a Promise.

---

# 21. Promise Chain vs async/await

Promise chain:

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

Async/await:

```js
async function load() {
  const user =
    await getUser();

  const orders =
    await getOrders(
      user.id
    );

  const details =
    await getDetails(
      orders[0].id
    );

  return details;
}
```

Same dependency flow.

Different syntax.

---

# 22. React / Next.js Connection

Modern Next.js code often uses async functions:

```js
export default async function Page() {
  const data =
    await getData();

  return (
    <div>
      {data.name}
    </div>
  );
}
```

Server Actions and server-side utilities also commonly use async/await.

Understanding the underlying Promise behavior prevents accidental serial requests and error-handling mistakes.

---

# Interview Questions

### What does an async function return?

Always a Promise.

### Does await block JavaScript?

No. It pauses the continuation of the current async function while allowing other JavaScript work to proceed.

### What happens if an awaited Promise rejects?

It behaves like a thrown error at the await point.

### Does async make CPU-heavy code asynchronous?

No.

---

# Key Takeaways

- `async` functions always return Promises.
- Returning a value fulfills the async function's Promise.
- Throwing rejects it.
- `await` unwraps fulfilled Promise values.
- Rejected awaited Promises throw inside the async function.
- Code before the first `await` runs synchronously.
- `await` does not block the JavaScript thread.
- Multiple awaits can accidentally make independent tasks sequential.
- Async/await is Promise syntax, not a separate async system.
