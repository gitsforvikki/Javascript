# Lesson 07 — Operators and Expressions

Operators are symbols used to perform operations on values.

## Arithmetic Operators

```js
const a = 10;
const b = 3;

console.log(a + b); // 13
console.log(a - b); // 7
console.log(a * b); // 30
console.log(a / b); // 3.333...
console.log(a % b); // 1
console.log(a ** b); // 1000
```

## Assignment Operators

```js
let count = 10;

count += 5;
count -= 2;
count *= 2;
```

## Comparison Operators

```js
10 > 5;   // true
10 < 5;   // false
10 >= 10; // true
10 <= 5;  // false
10 === 10; // true
10 !== 5;  // true
```

## Logical Operators

### AND — &&

Both conditions must be truthy.

```js
const age = 25;
const hasId = true;

console.log(age >= 18 && hasId); // true
```

### OR — ||

At least one condition must be truthy.

```js
const isAdmin = false;
const isManager = true;

console.log(isAdmin || isManager); // true
```

### NOT — !

Reverses a boolean value.

```js
console.log(!true); // false
```

## Increment and Decrement

```js
let count = 1;

count++;
count--;

console.log(count); // 1
```

## Ternary Operator

A short form of a simple `if/else`.

```js
const age = 20;

const status = age >= 18 ? "Adult" : "Minor";

console.log(status);
```

## What is an Expression?

An expression is code that produces a value.

```js
10 + 20
age >= 18
isLoggedIn && isAdmin
```

## Key Takeaways

- Arithmetic operators perform calculations.
- Comparison operators compare values.
- Logical operators combine conditions.
- The ternary operator is useful for simple conditional values.
- An expression produces a value.

## Interview Quick Answer

**What are logical operators in JavaScript?**

The main logical operators are AND (`&&`), OR (`||`), and NOT (`!`). They are commonly used to combine or reverse conditions.
