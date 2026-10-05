# Lesson 11 — Function Declarations and Function Expressions

Functions are one of the most important building blocks in JavaScript. A function groups reusable logic that can receive input, perform some work, and optionally return a result.

## 1. Function Declaration

A function declaration uses the `function` keyword followed by a function name.

```js
function add(a, b) {
  return a + b;
}

const result = add(10, 20);

console.log(result); // 30
```

Structure:

```text
function functionName(parameters) {
  // function body
  return value;
}
```

### Calling a Function

Defining a function does not execute its body.

```js
function greet() {
  console.log("Hello");
}
```

The function executes when it is invoked:

```js
greet();
```

## 2. Function Expression

A function can also be created as an expression and assigned to a variable.

```js
const add = function (a, b) {
  return a + b;
};

console.log(add(10, 20)); // 30
```

Here:

```text
function (a, b) { ... }
        ↓
function value
        ↓
stored in variable "add"
```

Functions are values in JavaScript. This idea becomes very important when we study first-class functions, callbacks and higher-order functions.

## Function Declaration vs Function Expression

```js
function greet() {
  console.log("Hello");
}
```

versus:

```js
const greet = function () {
  console.log("Hello");
};
```

A major behavioral difference involves **hoisting**.

A function declaration can normally be called before its declaration:

```js
greet();

function greet() {
  console.log("Hello");
}
```

But this does not work the same way with a function expression stored in `const`:

```js
greet(); // ReferenceError

const greet = function () {
  console.log("Hello");
};
```

Why this happens will be covered deeply in the execution context and hoisting section.

## Named Function Expressions

A function expression can also have its own name.

```js
const calculate = function add(a, b) {
  return a + b;
};
```

Most everyday application code commonly uses anonymous function expressions or arrow functions.

## return

A function can send a value back to the caller using `return`.

```js
function multiply(a, b) {
  return a * b;
}

const answer = multiply(4, 5);

console.log(answer); // 20
```

Once `return` executes, the function stops.

```js
function test() {
  return "Done";

  console.log("Never runs");
}
```

If no value is returned explicitly, the function returns `undefined`.

```js
function greet() {
  console.log("Hello");
}

console.log(greet()); // undefined
```

## Why Functions Matter in Real Applications

Functions are everywhere:

```js
function validateUser(user) {}
function calculateTotal(items) {}
function formatPrice(price) {}
function handleLogin(credentials) {}
```

React components, event handlers, callbacks, API utilities and server functions all depend heavily on JavaScript function behavior.

## Common Mistakes

### Forgetting return

```js
function add(a, b) {
  a + b;
}

console.log(add(2, 3)); // undefined
```

Correct:

```js
function add(a, b) {
  return a + b;
}
```

### Confusing definition with invocation

```js
function greet() {
  console.log("Hello");
}

greet;   // reference to function
greet(); // executes function
```

## Interview Perspective

**What is the difference between a function declaration and function expression?**

A function declaration defines a named function using the `function` statement. A function expression creates a function as a value and usually assigns it to a variable. Their important difference is how they behave during hoisting.

## Key Takeaways

- Functions contain reusable logic.
- Functions can accept input and return output.
- Function declarations and function expressions both create functions.
- Functions are values in JavaScript.
- Function declarations and expressions behave differently with hoisting.
- `return` sends a result back and ends function execution.
