# Lesson 37 — Temporal Dead Zone

The **Temporal Dead Zone (TDZ)** is the period in which a lexical binding exists but cannot yet be accessed.

It mainly applies to:

- `let`
- `const`
- `class`

The key idea is:

> The binding exists from the beginning of its scope, but remains uninitialized until execution reaches its declaration.

---

## 1. Basic Example

```js
console.log(value);

let value = 10;
```

Result:

```text
ReferenceError
```

The binding exists, but it has not been initialized yet.

---

## 2. TDZ Starts at Scope Entry

```js
{
  // TDZ starts here

  console.log(value);

  let value = 10;

  // TDZ ends after initialization
}
```

TDZ is about execution timing inside a lexical scope.

---

## 3. Why "Temporal"?

Consider:

```js
{
  function read() {
    console.log(value);
  }

  let value = 10;

  read();
}
```

This works because the access happens after initialization.

So TDZ is not merely "code written above the declaration".

It depends on when the access occurs.

---

## 4. let and const Are Hoisted

Do not say:

> let and const are not hoisted.

A better explanation is:

> Their bindings are created during scope setup, but remain uninitialized in the TDZ.

---

## 5. Shadowing Trap

```js
let value = "outer";

{
  console.log(value);

  let value = "inner";
}
```

Result:

```text
ReferenceError
```

The inner `value` shadows the outer one for the entire block.

JavaScript does not fall back to the outer binding.

---

## 6. const Has the Same TDZ Behavior

```js
{
  console.log(user);

  const user = {
    name: "Vikash",
  };
}
```

This throws a `ReferenceError`.

---

## 7. Class TDZ

```js
const user =
  new User();

class User {}
```

Also throws:

```text
ReferenceError
```

---

## 8. typeof and TDZ

Usually:

```js
typeof unknownName;
```

returns:

```text
"undefined"
```

But:

```js
{
  console.log(
    typeof value
  );

  let value = 10;
}
```

throws `ReferenceError`.

Why?

Because `value` is a real binding currently in TDZ.

---

## 9. TDZ Is Scope-Based

```js
let value = 1;

function test() {
  console.log(value);
}

test();
```

This works.

But:

```js
let value = 1;

function test() {
  console.log(value);

  let value = 2;
}

test();
```

throws because the local binding shadows the outer binding and is still uninitialized.

---

## 10. Default Parameter Ordering

A subtle interview example:

```js
function test(
  a = b,
  b = 10
) {
  return a + b;
}

test();
```

This throws because `b` is not initialized when `a`'s default expression runs.

But:

```js
function test(
  a = 10,
  b = a
) {
  return a + b;
}

test();
```

works.

---

## 11. Why TDZ Exists

TDZ catches bugs early.

Compare:

```js
console.log(a);
var a = 10;
```

which silently prints `undefined`

with:

```js
console.log(a);
let a = 10;
```

which throws immediately.

---

## 12. TDZ Is Not "No Variable Exists"

Avoid saying:

> The variable does not exist before declaration.

A better mental model:

```text
binding exists
↓
binding is uninitialized
↓
access throws
```

---

## 13. Closures and TDZ

```js
function outer() {
  const read =
    () => value;

  let value = 10;

  return read;
}

const fn =
  outer();

console.log(
  fn()
);
```

Output:

```text
10
```

The closure reads the binding after initialization.

---

## 14. Calling Too Early

```js
function outer() {
  const read =
    () => value;

  console.log(
    read()
  );

  let value = 10;
}

outer();
```

This throws because the closure accesses `value` while it is still in TDZ.

---

## 15. Interview Output Question

```js
let a = 1;

{
  console.log(a);
  let a = 2;
}
```

Answer:

```text
ReferenceError
```

---

## 16. React Connection

React does not change JavaScript TDZ semantics.

```jsx
function Component() {
  const getValue =
    () => value;

  const value = 10;

  return (
    <div>
      {getValue()}
    </div>
  );
}
```

This works because `getValue()` runs after `value` initialization.

---

## Interview Answer — 30 Seconds

> The Temporal Dead Zone is the period from entering a lexical scope until a `let`, `const`, or class binding is initialized. The binding already exists, so it can shadow outer variables, but accessing it before initialization throws a ReferenceError.

---

## Key Takeaways

- TDZ applies to lexical bindings.
- The binding exists before initialization.
- Access during TDZ throws `ReferenceError`.
- TDZ is based on execution timing.
- Shadowing explains many tricky TDZ questions.
- Even `typeof` can throw when a binding is in TDZ.
