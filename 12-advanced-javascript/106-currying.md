# Lesson 106 — Currying

Currying transforms a function that accepts multiple arguments into a sequence of functions that each accept one argument.

Example:

```text
f(a, b, c)

becomes

f(a)(b)(c)
```

The key idea:

> Currying changes a function's shape into nested single-argument functions.

---

## 1. Normal Function

```js
function add(
  a,
  b
) {
  return a + b;
}
```

Call:

```js
add(2, 3);
```

---

## 2. Curried Version

```js
function add(
  a
) {
  return function (
    b
  ) {
    return a + b;
  };
}
```

Call:

```js
add(2)(3);
```

Output:

```text
5
```

---

## 3. Arrow Function Version

```js
const add =
  (a) =>
  (b) =>
    a + b;
```

Call:

```js
add(2)(3);
```

---

## 4. Closure Connection

When you call:

```js
const add2 =
  add(2);
```

the returned function remembers:

```text
a = 2
```

through closure.

Then:

```js
add2(5);
```

returns:

```text
7
```

Currying is built on closures.

---

## 5. Three Arguments

Normal:

```js
function multiply(
  a,
  b,
  c
) {
  return a * b * c;
}
```

Curried:

```js
const multiply =
  (a) =>
  (b) =>
  (c) =>
    a * b * c;
```

Call:

```js
multiply(2)(3)(4);
```

Output:

```text
24
```

---

## 6. Why Currying Is Useful

Currying lets you preconfigure functions incrementally.

Example:

```js
const multiplyBy =
  (factor) =>
  (value) =>
    factor * value;
```

Create:

```js
const double =
  multiplyBy(2);

const triple =
  multiplyBy(3);
```

Then:

```js
double(10);
// 20

triple(10);
// 30
```

---

## 7. Reusable Predicates

```js
const greaterThan =
  (min) =>
  (value) =>
    value > min;
```

Create:

```js
const greaterThan18 =
  greaterThan(18);
```

Use:

```js
[
  10,
  20,
  30,
].filter(
  greaterThan18
);
```

Result:

```js
[20, 30]
```

---

## 8. Configuration Functions

```js
const createLogger =
  (level) =>
  (message) => {
    console.log(
      `[${level}] ${message}`
    );
  };
```

Create:

```js
const errorLog =
  createLogger(
    "ERROR"
  );
```

Use:

```js
errorLog(
  "Request failed"
);
```

---

## 9. Currying and Function Composition

Curried functions work nicely with composition because each configured function can expose a simple one-input/one-output shape.

Example:

```js
const multiplyBy =
  (factor) =>
  (value) =>
    value * factor;

const add =
  (amount) =>
  (value) =>
    value + amount;
```

Configured:

```js
const double =
  multiplyBy(2);

const add10 =
  add(10);
```

These are easy to compose.

---

## 10. Manual Currying

You can write a utility:

```js
function curry(
  fn
) {
  return function curried(
    ...args
  ) {
    if (
      args.length >=
      fn.length
    ) {
      return fn.apply(
        this,
        args
      );
    }

    return function (
      ...nextArgs
    ) {
      return curried.apply(
        this,
        [
          ...args,
          ...nextArgs,
        ]
      );
    };
  };
}
```

Example:

```js
function sum(
  a,
  b,
  c
) {
  return a + b + c;
}

const curriedSum =
  curry(sum);

curriedSum(1)(2)(3);
```

---

## 11. fn.length

```js
fn.length
```

usually reports the number of parameters before the first one with a default/rest complication.

So a generic curry utility based purely on `fn.length` has limitations.

---

## 12. Currying Does Not Mean "Only One Call Syntax"

Some curry utilities allow:

```js
curriedSum(
  1,
  2
)(3);
```

or:

```js
curriedSum(
  1
)(
  2,
  3
);
```

Strict mathematical currying is one argument per function, while JavaScript helper libraries may be more flexible.

---

## 13. Currying vs Nested Closures

Not every nested function is currying.

Example:

```js
function outer(
  user
) {
  return function log() {
    console.log(
      user
    );
  };
}
```

This uses closure, but it is not necessarily transforming a multi-argument function into unary stages.

---

## 14. Event Handler Factory

```js
const handleAction =
  (action) =>
  (event) => {
    console.log(
      action,
      event.target
    );
  };
```

Usage:

```jsx
<button
  onClick={
    handleAction(
      "save"
    )
  }
>
  Save
</button>
```

This resembles currying/function factories and is common in UI code.

---

## 15. API Configuration Example

```js
const request =
  (baseUrl) =>
  (method) =>
  async (path) => {
    return fetch(
      baseUrl + path,
      {
        method,
      }
    );
  };
```

Then:

```js
const api =
  request(
    "/api"
  );

const get =
  api("GET");

get("/users");
```

This shows progressive configuration.

---

## 16. Currying and Readability

Currying can improve reuse, but overusing it can make ordinary JavaScript harder to read.

Compare:

```js
calculate(
  price,
  tax,
  discount
);
```

with:

```js
calculate(
  price
)(
  tax
)(
  discount
);
```

Use currying when it improves composition/configuration—not because it looks advanced.

---

## 17. Currying vs Partial Application Preview

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
new function expecting b, c
```

Lesson 107 will compare them carefully.

---

## 18. Interview Questions

### What is currying?

Transforming a multi-argument function into a sequence of functions that each accept one argument.

### What JavaScript feature makes currying possible?

Closures.

### Why use currying?

For reusable configuration, predicates, composition, and function specialization.

### Is every nested function curried?

No.

---

## Key Takeaways

- Currying transforms function shape.
- Typical form is `f(a)(b)(c)`.
- Closures preserve previously supplied arguments.
- Currying can create reusable specialized functions.
- Curried functions work well with composition.
- Generic curry helpers have edge cases.
- Currying should improve clarity, not reduce it.
