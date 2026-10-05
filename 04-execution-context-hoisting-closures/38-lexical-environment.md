# Lesson 38 — Lexical Environment

A **lexical environment** is a core concept behind JavaScript scope resolution and closures.

The most important idea is:

> JavaScript determines outer scope relationships from where code is written, not from where a function is called.

---

## 1. Start with an Example

```js
const globalName = "JavaScript";

function outer() {
  const framework = "React";

  function inner() {
    const version = 19;

    console.log(
      globalName,
      framework,
      version
    );
  }

  inner();
}

outer();
```

Inside `inner`, JavaScript can access:

- `version` from its own environment
- `framework` from the outer function
- `globalName` from the outer/global environment

Why?

Because lexical environments are linked.

---

## 2. Simplified Lexical Environment Model

A lexical environment can be visualized as:

```text
Lexical Environment
│
├── Environment Record
│   └── local bindings
│
└── Outer Environment Reference
    └── points to surrounding lexical environment
```

For the previous example:

```text
inner environment
│
├── version = 19
└── outer ──────────────┐
                        ↓
outer environment
│
├── framework = "React"
└── outer ──────────────┐
                        ↓
global environment
│
└── globalName = "JavaScript"
```

These links form the basis of the **scope chain**.

---

## 3. What Does "Lexical" Mean?

Lexical means the relationship is determined by the **source-code structure**.

Example:

```js
const value = "global";

function outer() {
  const value = "outer";

  function inner() {
    console.log(value);
  }

  return inner;
}

const fn = outer();

fn();
```

Output:

```text
outer
```

Even though `fn()` is later called from global code, `inner` was **defined inside `outer`**.

Therefore its outer lexical environment is associated with where it was created.

This is lexical scoping.

---

## 4. Definition Location vs Call Location

This distinction is extremely important.

```js
const name = "Global";

function printName() {
  console.log(name);
}

function run() {
  const name = "Run";

  printName();
}

run();
```

Output:

```text
Global
```

Why not `Run`?

Because `printName` was defined in the global lexical environment.

Calling it inside `run` does not change its lexical parent.

```text
Where defined?

printName
   ↓
Global Environment

Where called?

run()
   ↓
does NOT redefine printName's lexical parent
```

This is one of the most important rules for understanding closures.

---

## 5. Environment Record

Conceptually, the environment record stores bindings associated with that lexical environment.

Example:

```js
function calculate(price) {
  const tax = 10;
  let total = price + tax;

  return total;
}
```

Simplified environment:

```text
calculate lexical environment
│
├── price
├── tax
└── total
```

Different declaration types have different initialization and scope rules, but they are resolved through environment structures.

---

## 6. Block Lexical Environments

`let` and `const` are block scoped.

```js
const value = 10;

if (true) {
  const message = "Hello";
  let count = 1;

  console.log(value);
}
```

Conceptually:

```text
Block Environment
│
├── message
├── count
└── outer ──→ Global Environment
                 └── value
```

Outside the block:

```js
console.log(message);
```

fails because the block environment is not an outer environment of the code outside that block.

---

## 7. Lexical Environment vs Execution Context

They are related but should not be treated as synonyms.

A useful learning model:

```text
Execution Context
│
├── currently executing function/script information
├── lexical environment information
└── other execution state
```

The lexical environment specifically helps model:

- identifier bindings
- lexical nesting
- outer environment references

The execution context is the broader execution mechanism.

---

## 8. Lexical Environment and Closures

Consider:

```js
function createCounter() {
  let count = 0;

  return function increment() {
    count++;

    return count;
  };
}

const counter = createCounter();

console.log(counter()); // 1
console.log(counter()); // 2
```

After `createCounter()` finishes, its call is no longer active on the call stack.

Yet `increment` can still access `count`.

Why?

Because the returned function retains access to the lexical environment needed by the closure.

Conceptually:

```text
counter
   │
   └──→ increment function
              │
              └── lexical reference
                       ↓
               count binding
```

This is the foundation of closures, which we will study fully in Lesson 41.

---

## 9. React Connection

```jsx
function Counter() {
  const message = "Current render";

  const handleClick = () => {
    console.log(message);
  };

  return (
    <button onClick={handleClick}>
      Click
    </button>
  );
}
```

`handleClick` is created inside the component invocation.

Its lexical environment gives it access to variables belonging to that render.

This is why closures are central to understanding React hooks and stale state.

---

## Interview Perspective

**What is a lexical environment?**

It is a specification-level structure used to associate identifier bindings with code and link that environment to its outer lexical environment.

**What determines a function's outer lexical environment?**

Its lexical relationship is determined by where the function is defined, not where it is later called.

---

## Key Takeaways

- Lexical environments store/associate identifier bindings.
- They have links to outer lexical environments.
- Lexical relationships follow source-code nesting.
- Function call location does not change lexical scope.
- Block scopes can create lexical environments.
- Scope chains are formed through outer-environment relationships.
- Closures depend on lexical environments.
