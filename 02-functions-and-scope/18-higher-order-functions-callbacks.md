# Lesson 18 — Higher-Order Functions and Callbacks

Higher-order functions and callbacks are fundamental JavaScript concepts.

They appear in:

- array methods
- event handling
- asynchronous JavaScript
- promises
- React
- reusable application logic

## What is a Higher-Order Function?

A **higher-order function (HOF)** is a function that does at least one of these:

1. Accepts another function as an argument
2. Returns another function

Example:

```js
function calculate(a, b, operation) {
  return operation(a, b);
}
```

`calculate` receives another function, so it is a higher-order function.

## What is a Callback?

A **callback** is a function passed to another function so that the receiving function can execute it.

```js
function add(a, b) {
  return a + b;
}

function calculate(a, b, callback) {
  return callback(a, b);
}

console.log(calculate(10, 20, add)); // 30
```

Roles:

```text
calculate → higher-order function

add → callback function
```

## Execution Flow

For:

```js
calculate(10, 20, add);
```

the flow is:

```text
calculate receives:
10
20
add function
      ↓
callback(a, b)
      ↓
add(10, 20)
      ↓
30
```

## Anonymous Callback

The callback does not need to be declared separately.

```js
const result = calculate(10, 20, function (a, b) {
  return a * b;
});

console.log(result); // 200
```

With an arrow function:

```js
const result = calculate(
  10,
  20,
  (a, b) => a * b
);
```

## Why Higher-Order Functions Are Useful

They separate **what should happen** from **when/how it should happen**.

Example:

```js
function processUser(user, action) {
  action(user);
}

processUser(
  { name: "Vikash" },
  (user) => console.log(user.name)
);
```

The processing function is reusable because behavior can be supplied from outside.

## Array Methods Are Higher-Order Functions

### map

```js
const numbers = [1, 2, 3];

const doubled = numbers.map((number) => {
  return number * 2;
});
```

`map` is the higher-order function.

```js
(number) => number * 2
```

is the callback.

### filter

```js
const numbers = [1, 2, 3, 4];

const even = numbers.filter(
  (number) => number % 2 === 0
);
```

### forEach

```js
users.forEach((user) => {
  console.log(user.name);
});
```

We will study these array methods deeply in Section 3.

## Higher-Order Function Returning a Function

A HOF can also return a function.

```js
function multiplyBy(multiplier) {
  return function (number) {
    return number * multiplier;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(10)); // 20
console.log(triple(10)); // 30
```

Flow:

```text
multiplyBy(2)
      ↓
returns function
      ↓
double
      ↓
double(10)
      ↓
20
```

This also demonstrates closure behavior because the returned function remembers `multiplier`.

## Synchronous Callbacks

A callback does not automatically mean asynchronous.

Example:

```js
[1, 2, 3].map((number) => number * 2);
```

The callback is executed synchronously while `map` processes the array.

## Asynchronous Callbacks

Callbacks can also execute later.

```js
setTimeout(() => {
  console.log("Executed later");
}, 1000);
```

The callback is supplied now but executed later by the surrounding runtime mechanism.

We will cover asynchronous callbacks and the event loop deeply in later sections.

## React Connection

React relies heavily on callbacks.

Event handlers:

```jsx
function Button() {
  const handleClick = () => {
    console.log("Clicked");
  };

  return (
    <button onClick={handleClick}>
      Click
    </button>
  );
}
```

Array rendering:

```jsx
users.map((user) => (
  <UserCard key={user.id} user={user} />
));
```

State updater callback:

```js
setCount((previousCount) => previousCount + 1);
```

Understanding callbacks is essential before studying closures and asynchronous JavaScript.

## Common Mistake: Calling Instead of Passing

Suppose:

```js
function handleClick() {
  console.log("Clicked");
}
```

Passing:

```js
button.addEventListener("click", handleClick);
```

Calling immediately:

```js
button.addEventListener("click", handleClick());
```

The second version executes `handleClick` immediately and passes its return value instead of the intended function reference.

## Interview Perspective

**What is a higher-order function?**

A higher-order function accepts one or more functions as arguments, returns a function, or both.

**What is a callback function?**

A callback is a function passed to another function so that the receiving function can execute it.

**Are callbacks always asynchronous?**

No. Callbacks can be synchronous, such as callbacks used by `map`, or asynchronous, such as a callback supplied to `setTimeout`.

## Key Takeaways

- JavaScript functions are first-class values.
- A higher-order function accepts or returns functions.
- A callback is a function passed to another function.
- Callbacks can be synchronous or asynchronous.
- Array methods heavily use callbacks.
- React heavily uses callbacks for events, rendering and state updates.
- Passing a function and calling a function are different operations.
