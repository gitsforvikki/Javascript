# Lesson 04 — JavaScript Data Types

JavaScript values are broadly divided into **primitive** and **non-primitive/reference** values.

## Primitive Types

JavaScript has seven primitive types:

1. String
2. Number
3. BigInt
4. Boolean
5. Undefined
6. Null
7. Symbol

Examples:

```js
const name = "Vikash";       // string
const age = 25;              // number
const large = 123n;          // bigint
const isDeveloper = true;    // boolean
let city;                    // undefined
const value = null;          // null
const id = Symbol("id");     // symbol
```

## Non-Primitive Type

Objects are non-primitive values.

Examples include:

```js
const user = {
  name: "Vikash",
};

const skills = ["JavaScript", "React"];

function greet() {
  console.log("Hello");
}
```

Arrays and functions are special kinds of objects in JavaScript.

## typeof

Use `typeof` to inspect the type of many values.

```js
typeof "hello";     // "string"
typeof 10;          // "number"
typeof true;        // "boolean"
typeof undefined;   // "undefined"
typeof {};          // "object"
typeof [];          // "object"
typeof function(){}; // "function"
```

One famous JavaScript behavior:

```js
typeof null; // "object"
```

Despite this result, `null` is a primitive value. The `typeof null` result is a historical JavaScript behavior.

## Dynamic Typing

JavaScript is dynamically typed.

```js
let value = 10;

value = "JavaScript";

value = true;
```

The same variable can hold values of different types during execution.

## Key Takeaways

- JavaScript has seven primitive data types.
- Objects are non-primitive values.
- Arrays and functions are object-related values.
- JavaScript is dynamically typed.
- `typeof null` returning `"object"` is a historical quirk.

## Interview Quick Answer

**What are JavaScript primitive types?**

String, Number, BigInt, Boolean, Undefined, Null and Symbol.
