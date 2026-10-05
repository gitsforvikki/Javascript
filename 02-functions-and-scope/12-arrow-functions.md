# Lesson 12 — Arrow Functions

Arrow functions provide a shorter syntax for writing functions.

They were introduced in ES6, but they are not only shorter syntax. They also behave differently from regular functions in important areas such as `this`.

## Basic Syntax

Regular function expression:

```js
const add = function (a, b) {
  return a + b;
};
```

Arrow function:

```js
const add = (a, b) => {
  return a + b;
};
```

## Implicit Return

If an arrow function contains only one expression, braces and `return` can be omitted.

```js
const add = (a, b) => a + b;

console.log(add(10, 20)); // 30
```

This is called an **implicit return**.

Compare:

```js
const square = (number) => {
  return number * number;
};
```

with:

```js
const square = number => number * number;
```

## Parameter Syntax

No parameters:

```js
const greet = () => console.log("Hello");
```

One parameter:

```js
const double = number => number * 2;
```

Multiple parameters:

```js
const add = (a, b) => a + b;
```

Using parentheses even for one parameter is often preferred for consistency:

```js
const double = (number) => number * 2;
```

## Returning an Object

This can be confusing:

```js
const createUser = () => {
  name: "Vikash";
};
```

The braces are interpreted as a function body.

To implicitly return an object, wrap it in parentheses:

```js
const createUser = () => ({
  name: "Vikash",
  role: "Developer",
});
```

## Arrow Functions and this

This is one of the most important differences.

Regular functions can receive their own `this` depending on how they are called.

Arrow functions **do not create their own `this` binding**. They use `this` from the surrounding lexical environment.

Conceptually:

```text
Regular function
    ↓
can have its own "this"

Arrow function
    ↓
uses "this" from surrounding scope
```

Example:

```js
const user = {
  name: "Vikash",

  regularMethod() {
    console.log(this.name);
  },
};

user.regularMethod(); // Vikash
```

Using an arrow function as an object method can produce unexpected behavior:

```js
const user = {
  name: "Vikash",

  showName: () => {
    console.log(this.name);
  },
};
```

Do not simply assume `this` means `user` here.

We will study `this` and lexical `this` deeply in Section 5.

## Arrow Functions Do Not Have Their Own arguments

Regular functions have access to an `arguments` object:

```js
function showArguments() {
  console.log(arguments);
}

showArguments(10, 20, 30);
```

Arrow functions do not create their own `arguments`.

Use rest parameters instead:

```js
const showArguments = (...args) => {
  console.log(args);
};

showArguments(10, 20, 30);
```

## Arrow Functions Cannot Be Used as Constructors

This does not work:

```js
const User = (name) => {
  this.name = name;
};

const user = new User("Vikash"); // TypeError
```

Arrow functions cannot be called with `new`.

## React Connection

Arrow functions are extremely common in React.

Event handlers:

```jsx
<button onClick={() => setCount(count + 1)}>
  Increment
</button>
```

Array transformations:

```js
users.map((user) => user.name);
```

Callbacks:

```js
fetchData().then((data) => {
  console.log(data);
});
```

Understanding arrow-function behavior is especially important when learning closures, callbacks and React rendering.

## Regular Function vs Arrow Function

| Feature | Regular Function | Arrow Function |
| --- | --- | --- |
| Short syntax | No | Yes |
| Own `this` behavior | Yes | No |
| Own `arguments` | Yes | No |
| Can use `new` | Yes, when suitable | No |
| Common for callbacks | Yes | Very common |

## Interview Perspective

**Are arrow functions just shorter regular functions?**

No. Arrow functions differ semantically. Most importantly, they do not create their own `this` or `arguments`, and they cannot be used as constructors.

## Key Takeaways

- Arrow functions provide concise function syntax.
- Single-expression arrows can use implicit return.
- Wrap an object in parentheses when returning it implicitly.
- Arrow functions do not have their own `this`.
- Arrow functions do not have their own `arguments`.
- Arrow functions cannot be used with `new`.
