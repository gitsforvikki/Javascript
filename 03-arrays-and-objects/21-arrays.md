# Lesson 21 — Arrays and Array Fundamentals

Arrays are one of the most commonly used data structures in JavaScript.

An array stores multiple values in a single ordered collection.

## Creating an Array

```js
const skills = ["JavaScript", "React", "Next.js"];
```

Each value has a numeric index.

```text
Index:    0             1          2
          ↓             ↓          ↓
      JavaScript      React      Next.js
```

JavaScript arrays use **zero-based indexing**.

```js
console.log(skills[0]); // JavaScript
console.log(skills[1]); // React
```

## Array Length

```js
console.log(skills.length); // 3
```

The last valid index is usually:

```js
skills.length - 1
```

Example:

```js
console.log(skills[skills.length - 1]); // Next.js
```

## Arrays Can Store Different Types

JavaScript arrays can contain different value types.

```js
const values = [
  "Vikash",
  25,
  true,
  null,
  { role: "Developer" },
  ["React", "Next.js"],
];
```

In real applications, arrays usually contain logically related values.

## Checking Whether a Value Is an Array

A common mistake is using only `typeof`.

```js
typeof []; // "object"
```

Use:

```js
Array.isArray([]);
```

Example:

```js
console.log(Array.isArray([])); // true
console.log(Array.isArray({})); // false
```

## Accessing and Updating Elements

```js
const skills = ["JavaScript", "React"];

skills[1] = "Next.js";

console.log(skills);
// ["JavaScript", "Next.js"]
```

Arrays are mutable objects, so their contents can be changed.

## Adding and Removing Elements

### push()

Adds an item to the end.

```js
const skills = ["JavaScript"];

skills.push("React");

console.log(skills);
// ["JavaScript", "React"]
```

`push()` returns the new array length.

### pop()

Removes the last item.

```js
const skills = ["JavaScript", "React"];

const removed = skills.pop();

console.log(removed); // React
console.log(skills);  // ["JavaScript"]
```

### unshift()

Adds an item to the beginning.

```js
const skills = ["React"];

skills.unshift("JavaScript");
```

### shift()

Removes the first item.

```js
const skills = ["JavaScript", "React"];

skills.shift();

console.log(skills); // ["React"]
```

## includes()

Checks whether an array contains a value.

```js
const skills = ["JavaScript", "React", "Next.js"];

console.log(skills.includes("React")); // true
console.log(skills.includes("Vue"));   // false
```

## indexOf()

Returns the first index of a value.

```js
const skills = ["JavaScript", "React"];

console.log(skills.indexOf("React")); // 1
console.log(skills.indexOf("Vue"));   // -1
```

## slice()

`slice()` returns a portion of an array without changing the original array.

```js
const numbers = [10, 20, 30, 40, 50];

const result = numbers.slice(1, 4);

console.log(result);  // [20, 30, 40]
console.log(numbers); // [10, 20, 30, 40, 50]
```

The end index is excluded.

```text
slice(start, end)

start → included
end   → excluded
```

## splice()

`splice()` can add, remove or replace elements, and it **mutates the original array**.

```js
const skills = ["JavaScript", "React", "Node.js"];

skills.splice(1, 1);

console.log(skills);
// ["JavaScript", "Node.js"]
```

Syntax:

```js
array.splice(startIndex, deleteCount, ...items);
```

Example replacing an item:

```js
const skills = ["JavaScript", "React", "Node.js"];

skills.splice(1, 1, "Next.js");

console.log(skills);
// ["JavaScript", "Next.js", "Node.js"]
```

## Iterating Over an Array

### for...of

```js
const skills = ["JavaScript", "React"];

for (const skill of skills) {
  console.log(skill);
}
```

### forEach()

```js
skills.forEach((skill, index) => {
  console.log(index, skill);
});
```

`forEach()` is useful when you want to perform an action for each element.

It does not build and return a transformed array like `map()`.

## Arrays Are Objects and Reference Values

```js
const a = [1, 2, 3];
const b = a;

b.push(4);

console.log(a); // [1, 2, 3, 4]
```

Both variables refer to the same array.

```text
a ──┐
    ├──→ [1, 2, 3]
b ──┘
```

This is why mutation is important when working with arrays.

## Copying an Array

A common shallow-copy approach is spread syntax:

```js
const original = [1, 2, 3];
const copy = [...original];

console.log(original === copy); // false
```

But nested objects remain shared references.

```js
const users = [{ name: "Vikash" }];
const copy = [...users];

copy[0].name = "Kumar";

console.log(users[0].name); // Kumar
```

We will study shallow and deep copying later in this section.

## React Connection

Arrays frequently represent UI data.

```jsx
const users = [
  { id: 1, name: "Vikash" },
  { id: 2, name: "Rahul" },
];

return users.map((user) => (
  <UserCard key={user.id} user={user} />
));
```

For React state, avoid directly mutating the existing array.

Instead of:

```js
skills.push("Node.js");
setSkills(skills);
```

prefer creating a new array:

```js
setSkills([...skills, "Node.js"]);
```

## Interview Perspective

**How do you check whether a value is an array?**

Use `Array.isArray(value)`, because `typeof []` returns `"object"`.

**What is the difference between slice and splice?**

`slice()` returns a selected portion without mutating the original array. `splice()` changes the original array by adding, removing or replacing elements.

## Key Takeaways

- Arrays are ordered, zero-indexed collections.
- Arrays are objects and reference values.
- `Array.isArray()` reliably identifies arrays.
- `push`, `pop`, `shift`, `unshift` and `splice` change arrays.
- `slice` does not mutate the original array.
- Reference behavior matters when copying and updating arrays.
- Immutable array updates are especially important in React.
