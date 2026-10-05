# Lesson 26 — Array Destructuring

Array destructuring lets us extract values from an array into variables using a compact syntax.

## Basic Syntax

Without destructuring:

```js
const skills = ["JavaScript", "React", "Next.js"];

const first = skills[0];
const second = skills[1];
```

With destructuring:

```js
const [first, second] = skills;

console.log(first);  // JavaScript
console.log(second); // React
```

Mental model:

```text
["JavaScript", "React", "Next.js"]
       ↓          ↓
     first      second
```

Array destructuring works by **position**.

## Skipping Elements

```js
const skills = ["JavaScript", "React", "Next.js"];

const [first, , third] = skills;

console.log(first); // JavaScript
console.log(third); // Next.js
```

## Default Values

```js
const values = ["JavaScript"];

const [language, framework = "React"] = values;

console.log(language);  // JavaScript
console.log(framework); // React
```

The default is used when the destructured value is `undefined`.

```js
const values = ["JavaScript", null];

const [language, framework = "React"] = values;

console.log(framework); // null
```

## Rest with Destructuring

```js
const skills = [
  "JavaScript",
  "React",
  "Next.js",
  "Node.js",
];

const [primary, ...remaining] = skills;

console.log(primary);
// JavaScript

console.log(remaining);
// ["React", "Next.js", "Node.js"]
```

The rest element must be last.

## Swapping Variables

Destructuring makes swapping values simple.

```js
let a = 10;
let b = 20;

[a, b] = [b, a];

console.log(a); // 20
console.log(b); // 10
```

## Destructuring Function Results

A function can return an array:

```js
function getCoordinates() {
  return [10, 20];
}

const [x, y] = getCoordinates();

console.log(x); // 10
console.log(y); // 20
```

## Nested Destructuring

```js
const data = [
  "Vikash",
  ["React", "Next.js"],
];

const [name, [frontend, framework]] = data;

console.log(name);      // Vikash
console.log(frontend);  // React
console.log(framework); // Next.js
```

Avoid overly complex nested destructuring when it hurts readability.

## React Connection

React hooks commonly return arrays.

```jsx
const [count, setCount] = useState(0);
```

Conceptually, the hook returns two values:

```text
[value, updaterFunction]
   ↓          ↓
 count     setCount
```

Array destructuring lets us choose our own variable names.

```jsx
const [name, setName] = useState("");
const [user, setUser] = useState(null);
```

## Array vs Object Destructuring

Array destructuring is based primarily on position:

```js
const [first, second] = values;
```

Object destructuring, which we will study later, is based on property names.

## Interview Perspective

**How does array destructuring work?**

It extracts array values into variables based on their positions.

**Can you skip array values while destructuring?**

Yes. Leave an empty position using a comma.

## Key Takeaways

- Array destructuring extracts values by position.
- Values can be skipped.
- Default values can be provided.
- Rest syntax collects remaining elements.
- Destructuring can swap variables.
- React hooks commonly use array destructuring.
