# Lesson 14 — Rest Parameters and Spread Syntax

Rest and spread use the same syntax:

```js
...
```

But their purpose depends on the context.

A useful mental model is:

```text
REST   → collect multiple values
SPREAD → expand values
```

## Rest Parameters

Rest parameters collect remaining function arguments into an array.

```js
function sum(...numbers) {
  console.log(numbers);
}

sum(10, 20, 30);
```

Output:

```js
[10, 20, 30]
```

Because `numbers` is a real array, array methods can be used directly.

```js
function sum(...numbers) {
  return numbers.reduce((total, number) => total + number, 0);
}

console.log(sum(10, 20, 30)); // 60
```

## Normal Parameters with Rest

```js
function createUser(name, role, ...skills) {
  console.log(name);
  console.log(role);
  console.log(skills);
}

createUser(
  "Vikash",
  "Developer",
  "JavaScript",
  "React",
  "Next.js"
);
```

Result:

```text
name   → "Vikash"
role   → "Developer"
skills → ["JavaScript", "React", "Next.js"]
```

The rest parameter must be the last parameter.

Invalid:

```js
// function test(...values, last) {}
```

## Spread Syntax with Arrays

Spread expands an iterable into individual values.

```js
const frontend = ["JavaScript", "React"];
const backend = ["Node.js", "Express"];

const skills = [...frontend, ...backend];

console.log(skills);
```

Result:

```js
["JavaScript", "React", "Node.js", "Express"]
```

## Copying an Array

```js
const original = [1, 2, 3];
const copy = [...original];

console.log(copy);
```

The outer array is new:

```js
console.log(original === copy); // false
```

However, spread performs only a **shallow copy**.

```js
const users = [
  { name: "Vikash" }
];

const copy = [...users];

copy[0].name = "Kumar";

console.log(users[0].name); // Kumar
```

The nested object reference is still shared.

We will cover shallow and deep copying in detail later.

## Spread Syntax with Objects

```js
const user = {
  name: "Vikash",
  role: "Developer",
};

const updatedUser = {
  ...user,
  role: "Full Stack Developer",
};

console.log(updatedUser);
```

This pattern is extremely common in React state updates.

## Merging Objects

```js
const basicInfo = {
  name: "Vikash",
};

const professionalInfo = {
  role: "Developer",
};

const user = {
  ...basicInfo,
  ...professionalInfo,
};
```

If the same property appears multiple times, a later value overwrites an earlier one.

```js
const user = {
  name: "Vikash",
  ...{ name: "Kumar" },
};

console.log(user.name); // Kumar
```

## Spread in Function Calls

```js
const numbers = [10, 20, 30];

function add(a, b, c) {
  return a + b + c;
}

console.log(add(...numbers)); // 60
```

Conceptually:

```text
add(...[10, 20, 30])

becomes

add(10, 20, 30)
```

## Rest vs Spread

### Rest

Collects:

```js
function test(...values) {}
```

```text
10, 20, 30
    ↓
[10, 20, 30]
```

### Spread

Expands:

```js
const values = [10, 20, 30];

test(...values);
```

```text
[10, 20, 30]
     ↓
10, 20, 30
```

## React Connection

Spread is heavily used for immutable state updates.

```js
setUser({
  ...user,
  name: "Kumar",
});
```

Arrays:

```js
setSkills([
  ...skills,
  "Node.js",
]);
```

Understanding that these are shallow copies is important for nested React state.

## Interview Perspective

**What is the difference between rest and spread?**

Both use `...`. Rest collects multiple values into one array, while spread expands an iterable or object into individual elements/properties depending on context.

**Does spread create a deep copy?**

No. Spread creates a shallow copy.

## Key Takeaways

- Rest collects values.
- Spread expands values.
- Rest parameters create a real array.
- A rest parameter must be last.
- Spread is commonly used to copy and merge arrays and objects.
- Spread performs shallow copying.
- Spread syntax is fundamental to immutable React state updates.
