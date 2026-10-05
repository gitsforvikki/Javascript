# Lesson 107 — Partial Application

Partial application creates a new function by fixing some arguments of an existing function in advance.

The core idea:

> Supply some arguments now, receive a new function that expects the remaining arguments later.

---

## 1. Normal Function

```js
function multiply(
  a,
  b
) {
  return a * b;
}
```

Partial application:

```js
function multiplyBy(
  factor
) {
  return function (
    value
  ) {
    return multiply(
      factor,
      value
    );
  };
}
```

Create:

```js
const double =
  multiplyBy(2);
```

Then:

```js
double(10);
// 20
```

---

## 2. Main Mental Model

Original:

```text
f(a, b, c)
```

Partially apply:

```text
fix a = 10
↓
g(b, c)
```

The new function may still accept multiple arguments.

---

## 3. Currying vs Partial Application

Currying:

```text
f(a, b, c)
↓
f(a)(b)(c)
```

Partial application:

```text
f(a, b, c)
↓
fix a
↓
g(b, c)
```

This is the most important distinction.

---

## 4. Partial with bind()

`bind()` can partially apply leading arguments.

```js
function add(
  a,
  b
) {
  return a + b;
}

const add10 =
  add.bind(
    null,
    10
  );
```

Then:

```js
add10(5);
// 15
```

---

## 5. bind() Also Binds this

Important:

```js
fn.bind(
  thisArg,
  arg1,
  arg2
);
```

does two things:

- fixes `this`
- can pre-fill arguments

If `this` is irrelevant:

```js
null
```

is often used.

---

## 6. Generic Partial Utility

```js
function partial(
  fn,
  ...presetArgs
) {
  return function (
    ...laterArgs
  ) {
    return fn.apply(
      this,
      [
        ...presetArgs,
        ...laterArgs,
      ]
    );
  };
}
```

Usage:

```js
function greet(
  greeting,
  name
) {
  return `${greeting}, ${name}`;
}

const sayHello =
  partial(
    greet,
    "Hello"
  );

sayHello(
  "Vikash"
);
```

---

## 7. Logger Example

```js
function log(
  level,
  module,
  message
) {
  console.log(
    `[${level}] [${module}] ${message}`
  );
}
```

Partially configure:

```js
const errorLogger =
  partial(
    log,
    "ERROR",
    "AUTH"
  );
```

Then:

```js
errorLogger(
  "Login failed"
);
```

---

## 8. URL Builder Example

```js
function buildUrl(
  baseUrl,
  version,
  path
) {
  return (
    baseUrl +
    "/" +
    version +
    path
  );
}
```

Partial:

```js
const apiV1 =
  partial(
    buildUrl,
    "https://api.example.com",
    "v1"
  );
```

Then:

```js
apiV1(
  "/users"
);
```

---

## 9. React Event Handler Example

```js
function handleAction(
  action,
  id,
  event
) {
  console.log(
    action,
    id,
    event.type
  );
}
```

Partial:

```js
const handleDelete =
  partial(
    handleAction,
    "delete",
    42
  );
```

Now the event system can supply the remaining event argument.

---

## 10. Function Factory vs Partial Application

These can look similar.

Function factory:

```js
function createMultiplier(
  factor
) {
  return (
    value
  ) =>
    value * factor;
}
```

Partial application often starts from an existing multi-argument function:

```js
function multiply(
  factor,
  value
) {
  return factor *
    value;
}

const double =
  partial(
    multiply,
    2
  );
```

The distinction is conceptual, not always visible from syntax alone.

---

## 11. Partial Application and Reuse

Without partial:

```js
users.map(
  (user) =>
    formatUser(
      "compact",
      "en",
      user
    )
);
```

With partial:

```js
const compactEnglish =
  partial(
    formatUser,
    "compact",
    "en"
  );

users.map(
  compactEnglish
);
```

This can improve readability.

---

## 12. Partial Application with Object Config

Sometimes named object configuration is clearer than positional partial application.

Instead of:

```js
partial(
  request,
  "GET",
  "/api",
  true
);
```

you may prefer:

```js
createRequest({
  method:
    "GET",
  baseUrl:
    "/api",
  credentials:
    true,
});
```

Use partial application when the argument order is stable and meaningful.

---

## 13. Placeholder-Based Partial Application

Some functional libraries support placeholders:

```text
f(_, 2, _)
```

meaning:

- preset the second argument
- fill others later

JavaScript has no built-in placeholder syntax for generic partial application.

Libraries implement it themselves.

---

## 14. Partial vs Default Parameters

Default parameter:

```js
function greet(
  greeting =
    "Hello",
  name
) {
}
```

Partial application:

```js
const sayHello =
  partial(
    greet,
    "Hello"
  );
```

Default parameters provide fallback values.

Partial application creates a specialized function.

---

## 15. Partial vs Closure

Partial application usually uses closure internally.

```js
function partial(
  fn,
  preset
) {
  return function (
    later
  ) {
    return fn(
      preset,
      later
    );
  };
}
```

The returned function remembers `preset`.

---

## 16. Partial Application and Composition

Specialized one-purpose functions are easy to compose.

Example:

```js
const addTax =
  partial(
    multiply,
    1.18
  );
```

Then `addTax` can participate in a transformation pipeline.

---

## 17. When Partial Application Helps

Good cases:

- repeated configuration
- reusable callbacks
- adapters
- logging
- request builders
- formatter specialization

---

## 18. When It Hurts

Avoid when:

- positional arguments are confusing
- configuration changes often
- object parameters are clearer
- extra abstraction hides intent

---

## 19. Interview Questions

### What is partial application?

Creating a new function by pre-filling some arguments of an existing function.

### Currying vs partial application?

Currying transforms a function into unary stages. Partial application fixes some arguments and returns a function for the remaining arguments.

### Can bind perform partial application?

Yes, for leading arguments, while also binding `this`.

---

## Key Takeaways

- Partial application pre-fills arguments.
- The returned function receives the remaining arguments.
- It commonly uses closures.
- `bind()` can partially apply leading arguments.
- Partial application is not the same as currying.
- It is useful for function specialization and reusable callbacks.
