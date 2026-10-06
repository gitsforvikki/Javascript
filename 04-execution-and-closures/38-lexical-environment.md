# Lesson 38 — Lexical Environment

A **Lexical Environment** is the mechanism JavaScript uses to store bindings and resolve variables according to where code is written.

It explains:

- scope
- scope chain
- shadowing
- closures
- nested functions

The key idea:

> A lexical environment contains bindings for the current scope and a reference to an outer lexical environment.

---

## 1. Basic Structure

Conceptually:

```text
Lexical Environment
│
├── Environment Record
│   └── bindings
│
└── Outer Environment Reference
```

The environment record stores names and values.

The outer reference links to the surrounding lexical scope.

---

## 2. Example

```js
const globalValue =
  "global";

function outer() {
  const outerValue =
    "outer";

  function inner() {
    const innerValue =
      "inner";

    console.log(
      innerValue,
      outerValue,
      globalValue
    );
  }

  inner();
}

outer();
```

Inside `inner()`, lookup proceeds outward.

---

## 3. Why It Is Called Lexical

"Lexical" means based on where code is written.

```js
const value =
  "global";

function show() {
  console.log(value);
}

function caller() {
  const value =
    "caller";

  show();
}

caller();
```

Output:

```text
global
```

`show` uses the environment where it was defined, not the caller's local scope.

---

## 4. JavaScript Is Lexically Scoped

JavaScript does not normally resolve identifiers based on the runtime caller.

It resolves them based on lexical nesting.

That is the difference between lexical scope and dynamic scope.

---

## 5. Environment Record

For:

```js
function test(
  a
) {
  let b = 20;
  const c = 30;
}
```

the function environment conceptually contains bindings for:

```text
a
b
c
```

---

## 6. Outer Environment Reference

```js
const x = 1;

function outer() {
  const y = 2;

  function inner() {
    const z = 3;

    console.log(
      x + y + z
    );
  }

  inner();
}
```

Conceptually:

```text
inner
├── z
└── outer → outer environment

outer
├── y
└── outer → global environment

global
└── x
```

---

## 7. Scope Chain

The linked outer references form the scope chain.

Lookup:

```text
current environment
↓
outer environment
↓
next outer environment
↓
global
↓
not found
→ ReferenceError
```

---

## 8. Shadowing

```js
const value =
  "global";

function test() {
  const value =
    "local";

  console.log(value);
}

test();
```

Output:

```text
local
```

Lookup stops at the first matching binding.

---

## 9. No Inward Lookup

```js
function outer() {
  function inner() {
    const secret =
      "inside";
  }

  console.log(secret);
}
```

This fails.

Outer scopes cannot search into child scopes.

---

## 10. Block Environments

`let` and `const` create block-scoped bindings.

```js
{
  let a = 1;
  const b = 2;
}
```

That block has its own lexical environment linked to the surrounding environment.

---

## 11. var Is Different

`var` is function-scoped.

```js
function test() {
  if (true) {
    var value = 10;
  }

  console.log(value);
}
```

The binding belongs to the function environment, not the block.

---

## 12. Lexical Environment vs Execution Context

These concepts are related but not identical.

```text
Execution Context
→ active runtime execution state

Lexical Environment
→ bindings + outer links
```

A function execution context uses lexical-environment information for identifier resolution.

---

## 13. Lexical Environment vs Call Stack

```text
Call Stack
→ who called whom

Lexical Environment
→ where variables are searched
```

This distinction is extremely important.

---

## 14. Proof: Caller Is Not Lexical Parent

```js
const name =
  "Global";

function showName() {
  console.log(name);
}

function caller() {
  const name =
    "Caller";

  showName();
}

caller();
```

Call stack:

```text
global
↓
caller
↓
showName
```

Lexical chain for `showName`:

```text
showName scope
↓
global scope
```

Output:

```text
Global
```

---

## 15. Closures Depend on Lexical Environments

```js
function outer() {
  let count = 0;

  return function () {
    count++;

    return count;
  };
}
```

The returned function retains access to the lexical environment where it was created.

That is the foundation of closures.

---

## 16. Separate Calls Create Separate Environments

```js
function createCounter() {
  let count = 0;

  return () =>
    ++count;
}

const a =
  createCounter();

const b =
  createCounter();
```

`a` and `b` have independent `count` bindings.

---

## 17. TDZ Connection

A lexical environment can contain a binding that exists but is still uninitialized.

That is exactly how TDZ behavior is modeled.

---

## 18. React Connection

Each render creates new local bindings.

```jsx
function Counter({
  count
}) {
  function handleClick() {
    console.log(count);
  }

  return (
    <button
      onClick={
        handleClick
      }
    >
      Click
    </button>
  );
}
```

`handleClick` closes over the lexical environment of that render.

This is why stale closures can happen.

---

## Interview Answer — 30 Seconds

> A lexical environment stores the bindings for a scope and a reference to the outer lexical environment. JavaScript follows those linked environments to resolve identifiers. Because those links are based on where code is written, JavaScript uses lexical scope rather than dynamic scope. Closures work because functions retain access to the lexical environment where they were created.

---

## Key Takeaways

- Lexical environments store bindings and outer references.
- JavaScript uses lexical scope.
- Variable lookup moves outward only.
- The first matching binding wins.
- Call stack order does not determine scope lookup.
- Each function call can create a distinct environment.
- Closures rely on lexical environments.
