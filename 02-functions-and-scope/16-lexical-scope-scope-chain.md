# Lesson 16 — Lexical Scope and Scope Chain

Lexical scope and the scope chain explain **how JavaScript decides which variable a function can access**.

These concepts are fundamental for understanding closures.

## What is Lexical Scope?

JavaScript uses **lexical scoping**.

Lexical means that scope is determined by **where code is written in the source code**, not by where a function is called.

Example:

```js
const language = "JavaScript";

function outer() {
  const framework = "React";

  function inner() {
    console.log(language);
    console.log(framework);
  }

  inner();
}

outer();
```

`inner()` can access both variables because of where it was defined.

```text
Global Scope
│
├── language
│
└── outer()
     │
     ├── framework
     │
     └── inner()
          │
          ├── can access framework
          └── can access language
```

## Scope Chain

When JavaScript tries to resolve a variable, it searches through available lexical scopes.

Consider:

```js
const a = 10;

function outer() {
  const b = 20;

  function inner() {
    const c = 30;

    console.log(a + b + c);
  }

  inner();
}

outer();
```

Inside `inner()`:

```text
Looking for c
   ↓
inner scope
   ↓
FOUND

Looking for b
   ↓
inner scope
   ↓
not found
   ↓
outer scope
   ↓
FOUND

Looking for a
   ↓
inner scope
   ↓
outer scope
   ↓
global scope
   ↓
FOUND
```

This chain of outer lexical environments is called the **scope chain**.

## Lookup Goes Outward, Not Inward

An inner scope can access outer variables.

But an outer scope cannot access variables declared only inside an inner scope.

```js
function outer() {
  const outerValue = "outer";

  function inner() {
    const innerValue = "inner";

    console.log(outerValue); // works
  }

  console.log(innerValue); // ReferenceError
}
```

## Where a Function Is Defined Matters

Consider:

```js
const name = "Global";

function printName() {
  console.log(name);
}

function run() {
  const name = "Local";

  printName();
}

run();
```

Output:

```text
Global
```

Why not `Local`?

Because `printName` was **defined in the global scope**.

Its lexical environment is based on its definition location.

```text
Global
│
├── name = "Global"
├── printName()
│
└── run()
     └── name = "Local"
```

Calling `printName()` inside `run()` does not make `run` its lexical parent.

This distinction is extremely important.

## Variable Shadowing

An inner scope can have a variable with the same name as an outer scope.

```js
const role = "Global Role";

function showRole() {
  const role = "Developer";

  console.log(role);
}

showRole(); // Developer
```

JavaScript finds the nearest matching variable first.

```text
Current scope
    ↓
role found?
    ↓ yes
STOP searching
```

The global `role` still exists but is shadowed inside the function.

## Lexical Environment — Simple Mental Model

Conceptually, each scope has:

```text
Lexical Environment
├── variables/functions in this scope
└── reference to outer lexical environment
```

For nested functions:

```text
inner environment
      ↓ outer reference
outer environment
      ↓ outer reference
global environment
      ↓
null
```

This structure enables scope-chain lookup.

We will study lexical environments more deeply in the execution-model section.

## Connection to Closures

Lexical scope makes closures possible.

```js
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message);
  }

  return inner;
}

const fn = outer();

fn(); // Hello
```

Even after `outer()` has finished, the returned function can still access `message`.

Why?

Because `inner` was lexically created inside `outer`.

This behavior is called a **closure**, which will receive its own deep lesson later.

## React Connection

Consider:

```jsx
function Counter() {
  const count = 10;

  function handleClick() {
    console.log(count);
  }

  return <button onClick={handleClick}>Show Count</button>;
}
```

`handleClick` can access `count` because it was created inside the component function's lexical scope.

This same mechanism later explains:

- closures in React
- stale closures
- event handlers
- effects
- callbacks

## Common Mistake

Do not think a function gets access to variables from wherever it is called.

Its lexical scope comes from **where it was defined**.

## Interview Perspective

**What is lexical scope?**

Lexical scope means a function's accessible outer scopes are determined by where that function is defined in the source code.

**What is the scope chain?**

The scope chain is the sequence of lexical environments JavaScript searches when resolving a variable, starting from the current scope and moving outward.

## Key Takeaways

- JavaScript uses lexical scoping.
- Scope depends on where code is defined.
- Variable lookup starts locally and moves outward.
- Inner scopes can access appropriate outer scopes.
- Outer scopes cannot directly access inner variables.
- JavaScript stops at the nearest matching variable.
- Lexical scope is the foundation of closures.
