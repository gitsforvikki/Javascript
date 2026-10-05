# Lesson 108 — Function Composition

Function composition means combining small functions so the output of one becomes the input of another.

The core idea:

> Build complex behavior by connecting simple transformations.

---

## 1. Basic Example

```js
function double(
  value
) {
  return value * 2;
}

function addOne(
  value
) {
  return value + 1;
}
```

Manual composition:

```js
const result =
  addOne(
    double(5)
  );
```

Flow:

```text
5
↓ double
10
↓ addOne
11
```

---

## 2. compose()

A simple compose utility:

```js
function compose(
  ...functions
) {
  return function (
    value
  ) {
    return functions
      .reduceRight(
        (
          result,
          fn
        ) =>
          fn(result),
        value
      );
  };
}
```

Use:

```js
const transform =
  compose(
    addOne,
    double
  );

transform(5);
// 11
```

Composition applies right-to-left.

---

## 3. pipe()

Many developers prefer left-to-right flow.

```js
function pipe(
  ...functions
) {
  return function (
    value
  ) {
    return functions
      .reduce(
        (
          result,
          fn
        ) =>
          fn(result),
        value
      );
  };
}
```

Use:

```js
const transform =
  pipe(
    double,
    addOne
  );
```

Flow reads top-to-bottom/left-to-right.

---

## 4. compose vs pipe

```text
compose
→ right to left

pipe
→ left to right
```

Mathematically:

```text
compose(f, g)(x)
= f(g(x))
```

---

## 5. Why Small Functions Help

Instead of:

```js
function processPrice(
  price
) {
  return (
    Math.round(
      (
        price *
        1.18 -
        50
      ) *
        100
    ) / 100
  );
}
```

split:

```js
const addTax =
  (value) =>
    value * 1.18;

const subtractDiscount =
  (value) =>
    value - 50;

const roundMoney =
  (value) =>
    Math.round(
      value * 100
    ) / 100;
```

Then:

```js
const processPrice =
  pipe(
    addTax,
    subtractDiscount,
    roundMoney
  );
```

---

## 6. One Input, One Output Works Best

Composition is easiest when each function has a simple shape:

```text
input
↓
function
↓
output
```

Currying and partial application can help create such functions.

---

## 7. Composition with Curried Functions

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

Configure:

```js
const double =
  multiplyBy(2);

const add10 =
  add(10);
```

Compose:

```js
const transform =
  pipe(
    double,
    add10
  );
```

---

## 8. String Pipeline

```js
const trim =
  (value) =>
    value.trim();

const lower =
  (value) =>
    value.toLowerCase();

const removeSpaces =
  (value) =>
    value.replaceAll(
      " ",
      "-"
    );
```

Then:

```js
const slugify =
  pipe(
    trim,
    lower,
    removeSpaces
  );
```

Use:

```js
slugify(
  "  React Developer  "
);
```

Result:

```text
react-developer
```

---

## 9. Composition Encourages Pure Functions

Functions that:

- depend only on inputs
- return outputs
- avoid hidden mutation

are easier to compose.

Example:

```js
const increment =
  (value) =>
    value + 1;
```

Much easier to compose than a function that mutates global state.

---

## 10. Composition vs Nested Calls

Nested:

```js
format(
  validate(
    normalize(
      input
    )
  )
);
```

Pipeline:

```js
const process =
  pipe(
    normalize,
    validate,
    format
  );
```

The pipeline often communicates order more clearly.

---

## 11. Function Composition with Arrays

You already use composition-like thinking:

```js
const result =
  users
    .filter(
      isActive
    )
    .map(
      toViewModel
    )
    .sort(
      compareByName
    );
```

Each step transforms the previous result.

---

## 12. Async Composition Problem

Simple `pipe()` assumes synchronous values.

This fails conceptually if one stage returns a Promise and the next expects a resolved value.

---

## 13. Async Pipe

```js
function pipeAsync(
  ...functions
) {
  return function (
    value
  ) {
    return functions.reduce(
      (
        promise,
        fn
      ) =>
        promise.then(
          fn
        ),
      Promise.resolve(
        value
      )
    );
  };
}
```

Use:

```js
const process =
  pipeAsync(
    fetchUser,
    loadProfile,
    formatProfile
  );
```

Each stage waits for the previous Promise.

---

## 14. Error Handling in Async Composition

A rejected stage rejects the whole Promise chain unless recovered.

So:

```js
process(id)
  .catch(
    handleError
  );
```

This directly connects to Promise chaining.

---

## 15. Composition and Validation

Example:

```js
const normalizeEmail =
  (email) =>
    email
      .trim()
      .toLowerCase();

const ensureEmail =
  (email) => {
    if (
      !email.includes(
        "@"
      )
    ) {
      throw new Error(
        "Invalid email"
      );
    }

    return email;
  };
```

Compose:

```js
const prepareEmail =
  pipe(
    normalizeEmail,
    ensureEmail
  );
```

---

## 16. Composition Is Not Always Better

For complicated branching:

```text
if A
then B
else C
with side effects
and retries
```

plain imperative code may be clearer.

Use composition where transformations naturally form a pipeline.

---

## 17. React Connection

React itself encourages composition:

```jsx
<Page>
  <Header />
  <Content />
</Page>
```

This is component composition rather than function composition, but the design principle is similar:

> build larger behavior from smaller reusable units.

---

## 18. Utility Libraries

Libraries such as:

- Lodash/fp
- Ramda

provide:

- compose
- pipe
- curry
- partial utilities

You do not need a library to understand the concepts.

---

## 19. Interview Questions

### What is function composition?

Combining functions so the output of one becomes the input of the next.

### compose vs pipe?

Compose applies right-to-left. Pipe applies left-to-right.

### Why do pure functions compose well?

They have predictable input/output behavior and minimal hidden side effects.

---

## Key Takeaways

- Composition connects small transformations.
- `compose` usually runs right-to-left.
- `pipe` runs left-to-right.
- Pure functions are easiest to compose.
- Currying/partial application can create composition-friendly functions.
- Async composition requires Promise-aware sequencing.
- Use composition when it makes data flow clearer.
