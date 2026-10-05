# Lesson 17 — First-Class Functions

JavaScript treats functions as **first-class values**.

This means functions can be used similarly to other values such as strings, numbers and objects.

A function can be:

1. Stored in a variable
2. Stored in an object or array
3. Passed to another function
4. Returned from another function

This ability is the foundation of callbacks, higher-order functions and many React patterns.

## 1. Store a Function in a Variable

```js
const greet = function () {
  return "Hello";
};

console.log(greet());
```

The function itself is a value assigned to `greet`.

Arrow functions work the same way:

```js
const add = (a, b) => a + b;
```

## Function Reference vs Function Call

This distinction is extremely important.

```js
function greet() {
  return "Hello";
}

const a = greet;
const b = greet();
```

Here:

```text
greet   → function reference
greet() → execute function and get result
```

Therefore:

```js
console.log(typeof a); // function
console.log(b);        // Hello
```

## 2. Pass a Function as an Argument

```js
function greet(name) {
  return `Hello ${name}`;
}

function processUser(callback) {
  console.log(callback("Vikash"));
}

processUser(greet);
```

Notice:

```js
processUser(greet);
```

not:

```js
processUser(greet());
```

We pass the function itself so another function can decide when to execute it.

## 3. Return a Function

A function can create and return another function.

```js
function createGreeting(greeting) {
  return function (name) {
    return `${greeting} ${name}`;
  };
}

const sayHello = createGreeting("Hello");

console.log(sayHello("Vikash"));
```

Flow:

```text
createGreeting("Hello")
        ↓
returns a function
        ↓
stored in sayHello
        ↓
sayHello("Vikash")
        ↓
"Hello Vikash"
```

This pattern connects directly to closures and currying.

## 4. Store Functions in Objects

```js
const calculator = {
  add(a, b) {
    return a + b;
  },

  subtract(a, b) {
    return a - b;
  },
};

console.log(calculator.add(10, 5));
```

A function stored as an object property is commonly called a **method**.

## 5. Store Functions in Arrays

```js
const operations = [
  (a, b) => a + b,
  (a, b) => a - b,
  (a, b) => a * b,
];

console.log(operations[0](10, 5)); // 15
```

## Why This Matters

Many JavaScript APIs depend on functions being values.

### Array methods

```js
const numbers = [1, 2, 3];

const doubled = numbers.map((number) => number * 2);
```

The arrow function is passed as a value to `map`.

### Event handlers

```js
button.addEventListener("click", handleClick);
```

`handleClick` is passed as a function value.

### Promise callbacks

```js
fetchData().then((data) => {
  console.log(data);
});
```

### React

```jsx
<button onClick={handleClick}>
  Click
</button>
```

React receives the function reference and can execute it when the event occurs.

Incorrect:

```jsx
<button onClick={handleClick()}>
```

That executes the function during rendering instead of passing the function for later execution.

## First-Class Functions vs Higher-Order Functions

These terms are related but different.

**First-class functions** describe a language feature:

> Functions can be treated as values.

A **higher-order function** is a function that uses this ability by accepting or returning functions.

We will study higher-order functions in the next lesson.

## Interview Perspective

**What does it mean that JavaScript has first-class functions?**

It means functions are values. They can be assigned to variables, passed as arguments, stored in data structures and returned from other functions.

## Key Takeaways

- Functions are values in JavaScript.
- A function reference and a function invocation are different.
- Functions can be passed as arguments.
- Functions can be returned from functions.
- Functions can be stored in objects and arrays.
- First-class functions enable callbacks and higher-order functions.
- This concept is heavily used in React and modern JavaScript.
