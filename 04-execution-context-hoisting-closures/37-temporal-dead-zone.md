# Lesson 37 — Temporal Dead Zone (TDZ)

The **Temporal Dead Zone (TDZ)** is the period in which a lexical binding exists but cannot yet be accessed.

It mainly matters for:

- `let`
- `const`
- `class`

Understanding TDZ completes the hoisting model from Lesson 36.

---

## 1. Basic Example

```js
console.log(name);

let name = "Vikash";
```

This throws:

```text
ReferenceError
```

A common but incomplete explanation is:

> `let` is not hoisted.

A more accurate mental model is:

```text
Enter scope
   ↓
name binding is created
   ↓
name is uninitialized
   ↓
TEMPORAL DEAD ZONE
   ↓
let name = "Vikash"
   ↓
binding initialized
   ↓
name can now be accessed
```

---

## 2. Why Is It Called "Temporal"?

The TDZ depends on **when execution reaches the declaration**, not simply the number of lines.

```js
{
  // TDZ for count begins

  console.log("Before");

  let count = 10;

  // TDZ has ended
  console.log(count);
}
```

The binding belongs to the scope from its beginning, but it cannot be accessed until initialization.

---

## 3. let vs var

### var

```js
console.log(a); // undefined

var a = 10;
```

Simplified:

```text
scope begins
   ↓
a initialized to undefined
   ↓
console.log(a)
   ↓
a = 10
```

### let

```js
console.log(b); // ReferenceError

let b = 20;
```

Simplified:

```text
scope begins
   ↓
b exists but is uninitialized
   ↓
TDZ
   ↓
let b = 20
   ↓
initialized
```

This difference is central to modern JavaScript declaration behavior.

---

## 4. const and TDZ

`const` behaves similarly before initialization:

```js
console.log(role);

const role = "Developer";
```

Result:

```text
ReferenceError
```

Unlike `let`, a `const` declaration must also provide an initializer:

```js
const role;
```

This is a syntax error.

---

## 5. TDZ Is Scope-Based

Consider:

```js
let value = "global";

{
  console.log(value);

  let value = "local";
}
```

This does **not** print:

```text
global
```

It throws a `ReferenceError`.

Why?

The inner block has its own `value` binding.

```text
Global Scope
value = "global"

        ↓ enter block

Block Scope
value = uninitialized
        ↓
      TDZ
```

When JavaScript resolves `value`, it finds the inner binding first.

Because that binding is in the TDZ, access fails. JavaScript does not skip it and continue to the global variable.

---

## 6. TDZ with Function Scope

```js
const name = "Global";

function printName() {
  console.log(name);

  const name = "Local";
}

printName();
```

Result:

```text
ReferenceError
```

Inside `printName`, the local `name` shadows the outer `name`.

Before its declaration executes, the local binding is in the TDZ.

---

## 7. typeof and TDZ

Normally:

```js
console.log(typeof unknownVariable);
```

returns:

```text
"undefined"
```

But:

```js
console.log(typeof count);

let count = 10;
```

throws a `ReferenceError`.

Why?

`count` is not undeclared. It is an existing lexical binding currently in the TDZ.

This is a common advanced interview question.

---

## 8. TDZ with class

Classes also have TDZ-like behavior.

```js
const user = new User();

class User {}
```

This throws a `ReferenceError`.

Correct:

```js
class User {}

const user = new User();
```

---

## 9. TDZ and Function Parameters

A subtle example:

```js
function test(a = b, b = 10) {
  console.log(a, b);
}

test();
```

This throws a `ReferenceError`.

When the default value for `a` is evaluated, `b` has not yet been initialized.

Parameter initialization occurs in order.

But this works:

```js
function test(a = 10, b = a) {
  console.log(a, b);
}

test();
```

Output:

```text
10 10
```

because `a` has already been initialized when `b`'s default expression is evaluated.

---

## 10. Why Does TDZ Exist?

TDZ makes accessing a lexical variable before initialization an explicit error.

Compare:

```js
console.log(user);

var user = getUser();
```

This silently gives:

```text
undefined
```

Whereas:

```js
console.log(user);

const user = getUser();
```

fails immediately.

This helps expose incorrect ordering rather than silently using an uninitialized value.

---

## 11. React Connection

```jsx
function UserCard() {
  console.log(handleClick);

  const handleClick = () => {
    console.log("clicked");
  };

  return null;
}
```

`handleClick` is in the TDZ when the first `console.log` executes.

The same JavaScript rules apply inside React components because a component is still JavaScript execution.

---

## Interview Perspective

**What is the Temporal Dead Zone?**

The TDZ is the period from entering a lexical scope until a `let`, `const`, or class binding is initialized. The binding exists but cannot be accessed during this period.

**Why doesn't JavaScript use an outer variable when an inner variable is in TDZ?**

Because scope resolution finds the inner binding first. Since it exists but is uninitialized, accessing it throws instead of continuing to an outer scope.

---

## Key Takeaways

- TDZ applies to lexical bindings such as `let`, `const`, and classes.
- Their bindings exist before their declaration executes.
- They remain uninitialized during the TDZ.
- Access during TDZ throws a `ReferenceError`.
- TDZ is determined by execution timing within a scope.
- Shadowing can place an identifier in TDZ even when an outer variable has the same name.
- `typeof` does not bypass TDZ.
