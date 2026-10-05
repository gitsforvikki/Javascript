# Lesson 20 — IIFE and Function Patterns

An **IIFE** is an **Immediately Invoked Function Expression**.

It is a function expression that executes immediately after it is created.

IIFEs were especially common before JavaScript modules became standard.

## Basic IIFE

```js
(function () {
  console.log("I run immediately");
})();
```

Break it into two ideas.

Function expression:

```js
(function () {
  console.log("Hello");
})
```

Immediately invoke it:

```js
(function () {
  console.log("Hello");
})();
```

## Arrow Function IIFE

An IIFE can also use an arrow function.

```js
(() => {
  console.log("Arrow IIFE");
})();
```

## Why the Parentheses?

A normal function declaration looks like:

```js
function greet() {
  console.log("Hello");
}
```

Wrapping a function in parentheses makes JavaScript treat it as an expression.

```js
(function () {
  console.log("Hello");
})
```

Then `()` invokes that expression immediately.

Mental model:

```text
(function () {
  ...
})
     ↓
function expression

(function () {
  ...
})()
  ↑
execute immediately
```

## Passing Arguments to an IIFE

```js
(function (name) {
  console.log(`Hello ${name}`);
})("Vikash");
```

Output:

```text
Hello Vikash
```

## Creating Private Scope

Historically, one major reason for IIFEs was to avoid putting variables into the global scope.

```js
(function () {
  const secret = "private value";

  console.log(secret);
})();

console.log(secret); // ReferenceError
```

The variable exists only inside the function scope.

Before `let`, `const`, and ES modules were widely available, this was especially useful.

## Old Module Pattern

IIFEs were commonly used to create encapsulated modules.

```js
const counter = (function () {
  let count = 0;

  return {
    increment() {
      count++;
    },

    getCount() {
      return count;
    },
  };
})();

counter.increment();
counter.increment();

console.log(counter.getCount()); // 2
```

External code cannot directly access:

```js
count
```

but returned methods can still access it.

This works because of **closures**.

Conceptually:

```text
IIFE executes
    ↓
creates private count
    ↓
returns object with functions
    ↓
functions retain access to count
    ↓
closure
```

We will study closures deeply in Section 4.

## Do We Still Need IIFEs?

Modern JavaScript provides better tools for many cases that previously required IIFEs.

### Block scope

```js
{
  const value = "private to this block";
}
```

### ES Modules

```js
export function calculate() {}
```

Modules naturally provide their own scope and are the standard approach for organizing modern applications.

Therefore, you usually do **not** need to build React or Next.js code around IIFEs.

However, understanding IIFEs is still useful because:

- they appear in older JavaScript code
- they demonstrate function expressions
- they demonstrate scope
- they help explain closures
- they may appear in interviews

## Function Patterns Recap

By this point, you should recognize several ways functions are used.

### Function declaration

```js
function add(a, b) {
  return a + b;
}
```

### Function expression

```js
const add = function (a, b) {
  return a + b;
};
```

### Arrow function

```js
const add = (a, b) => a + b;
```

### Callback

```js
numbers.map((number) => number * 2);
```

### Higher-order function

```js
function execute(callback) {
  callback();
}
```

### IIFE

```js
(function () {
  console.log("Runs immediately");
})();
```

These are not separate kinds of JavaScript values. They are different **ways of creating or using functions**.

## Interview Perspective

**What is an IIFE?**

An IIFE is an Immediately Invoked Function Expression: a function expression that executes immediately after it is created.

**Why were IIFEs commonly used?**

They were commonly used to create private function scope and avoid polluting the global scope, especially before block-scoped declarations and ES modules became standard.

**Are IIFEs commonly required in modern React applications?**

Usually no. Modern modules, block scope and framework tooling solve most of the problems for which IIFEs were historically used.

## Section 2 Connection

The concepts in this section connect directly:

```text
Functions
   ↓
Functions are values
   ↓
First-class functions
   ↓
Callbacks
   ↓
Higher-order functions

Scope
   ↓
Lexical scope
   ↓
Scope chain
   ↓
Closures
```

The next sections will build heavily on these foundations.

## Key Takeaways

- IIFE means Immediately Invoked Function Expression.
- An IIFE executes immediately after creation.
- IIFEs can create isolated function scope.
- They were heavily used before modern modules and block scope.
- Modern application code usually uses ES modules instead.
- IIFEs are still useful for understanding functions, scope and closures.
