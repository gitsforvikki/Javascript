# Lesson 22 — Array Mutation vs Non-Mutation Methods

Understanding mutation is extremely important in modern JavaScript and React.

## What is Mutation?

Mutation means changing an existing value directly.

Example:

```js
const numbers = [1, 2, 3];

numbers.push(4);

console.log(numbers);
// [1, 2, 3, 4]
```

The original array itself changed.

Conceptually:

```text
Before
numbers ──→ [1, 2, 3]

push(4)

After
numbers ──→ [1, 2, 3, 4]
             same array
```

## Non-Mutating Operation

A non-mutating operation leaves the original array unchanged and produces another value or array.

```js
const numbers = [1, 2, 3];

const updated = [...numbers, 4];

console.log(numbers); // [1, 2, 3]
console.log(updated); // [1, 2, 3, 4]
```

Conceptually:

```text
numbers ──→ [1, 2, 3]

updated ──→ [1, 2, 3, 4]
```

They are different array references.

## Common Mutating Array Methods

These methods modify the original array:

```text
push()
pop()
shift()
unshift()
splice()
sort()
reverse()
fill()
copyWithin()
```

Example:

```js
const numbers = [3, 1, 2];

numbers.sort();

console.log(numbers); // [1, 2, 3]
```

The original `numbers` array has changed.

## Common Non-Mutating Methods

These do not directly modify the original array:

```text
map()
filter()
reduce()
slice()
concat()
flat()
flatMap()
find()
findIndex()
some()
every()
includes()
```

Example:

```js
const numbers = [1, 2, 3];

const doubled = numbers.map(
  (number) => number * 2
);

console.log(numbers); // [1, 2, 3]
console.log(doubled); // [2, 4, 6]
```

## Modern Non-Mutating Alternatives

Modern JavaScript also provides methods such as:

```text
toSorted()
toReversed()
toSpliced()
with()
```

### toSorted()

```js
const numbers = [3, 1, 2];

const sorted = numbers.toSorted(
  (a, b) => a - b
);

console.log(numbers); // [3, 1, 2]
console.log(sorted);  // [1, 2, 3]
```

Unlike `sort()`, `toSorted()` does not modify the original array.

### toReversed()

```js
const numbers = [1, 2, 3];

const reversed = numbers.toReversed();

console.log(numbers);  // [1, 2, 3]
console.log(reversed); // [3, 2, 1]
```

### toSpliced()

```js
const skills = ["JS", "React", "Node"];

const updated = skills.toSpliced(
  1,
  1,
  "Next.js"
);

console.log(skills);
// ["JS", "React", "Node"]

console.log(updated);
// ["JS", "Next.js", "Node"]
```

## Why Mutation Can Cause Bugs

Consider two variables sharing the same array:

```js
const original = ["JS", "React"];
const copy = original;

copy.push("Node.js");

console.log(original);
// ["JS", "React", "Node.js"]
```

Why?

```text
original ──┐
           ├──→ same array
copy ──────┘
```

Mutation through either reference affects the same array.

## React Connection

React state should generally be treated as immutable.

Problem:

```js
skills.push("Node.js");
setSkills(skills);
```

The same array reference is reused.

Better:

```js
setSkills([
  ...skills,
  "Node.js",
]);
```

Now:

```text
old state ──→ old array

new state ──→ new array
```

React can reason about changed references much more reliably.

## Updating an Item Without Mutation

Suppose:

```js
const users = [
  { id: 1, name: "Vikash" },
  { id: 2, name: "Rahul" },
];
```

Avoid:

```js
users[0].name = "Kumar";
```

A common immutable pattern:

```js
const updatedUsers = users.map((user) =>
  user.id === 1
    ? { ...user, name: "Kumar" }
    : user
);
```

This creates:

- a new outer array
- a new object for the changed user
- existing references for unchanged users

## Removing an Item Without Mutation

```js
const updatedUsers = users.filter(
  (user) => user.id !== 2
);
```

## Adding an Item Without Mutation

```js
const updatedUsers = [
  ...users,
  { id: 3, name: "Aman" },
];
```

## Important: Non-Mutating Does Not Mean Deep Copy

```js
const users = [
  {
    id: 1,
    profile: {
      name: "Vikash",
    },
  },
];

const copy = [...users];
```

The outer array is new, but nested references remain shared.

```text
users ──→ Array A ──┐
                    ├──→ same user object
copy ───→ Array B ──┘
```

This distinction becomes important with nested React state.

## Interview Perspective

**What is mutation?**

Mutation means directly changing an existing object or array rather than creating a new value.

**Why is immutability important in React?**

Creating new references makes state changes predictable and helps React correctly identify changes while avoiding accidental modification of existing state.

**Does map mutate the original array?**

No. `map()` returns a new array. However, objects contained inside arrays can still be shared and mutated if you modify them directly.

## Key Takeaways

- Mutation changes an existing array.
- Non-mutating operations preserve the original array.
- `push`, `splice`, `sort`, and `reverse` mutate.
- `map`, `filter`, `slice`, and `concat` do not mutate the original array.
- Modern JavaScript provides `toSorted`, `toReversed`, and `toSpliced`.
- Non-mutating operations do not automatically deep-clone nested values.
- Immutability is especially important for React state.
