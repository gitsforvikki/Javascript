# Lesson 30 — Shallow Copy vs Deep Copy

Shallow copy vs deep copy is one of the most important concepts when working with objects, arrays, React state and nested application data.

To understand it correctly, remember:

> Objects and arrays can contain references to other objects and arrays.

## Starting Example

```js
const user = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
  },
};
```

Conceptually:

```text
user
 │
 └──→ Object A
       ├── name: "Vikash"
       │
       └── address ──→ Object B
                       └── city: "Bengaluru"
```

There are two different objects:

- outer user object
- nested address object

---

# Assignment Is Not a Copy

Consider:

```js
const user1 = {
  name: "Vikash",
};

const user2 = user1;
```

No new object is created.

```text
user1 ──┐
        ├──→ Object A
user2 ──┘
```

Therefore:

```js
user2.name = "Kumar";

console.log(user1.name);
// Kumar
```

Both variables access the same object.

---

# What is a Shallow Copy?

A shallow copy creates a **new outer object or array**, but nested object references remain shared.

Using object spread:

```js
const original = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
  },
};

const copy = {
  ...original,
};
```

Now:

```js
console.log(
  original === copy
); // false
```

The outer objects are different.

But:

```js
console.log(
  original.address === copy.address
); // true
```

The nested object is still shared.

Visual:

```text
original ──→ Object A
              │
              └── address ──┐
                             │
                             ├──→ Object C
                             │
copy ─────→ Object B         │
              │              │
              └── address ───┘
```

## Consequence

```js
copy.address.city = "Pune";

console.log(
  original.address.city
); // Pune
```

Changing the nested object through `copy` also affects `original`.

That is the key idea behind a shallow copy.

## Common Shallow-Copy Techniques

### Object spread

```js
const copy = {
  ...original,
};
```

### Object.assign()

```js
const copy = Object.assign(
  {},
  original
);
```

### Array spread

```js
const copy = [...originalArray];
```

### slice()

```js
const copy = originalArray.slice();
```

These create new outer containers, not recursive copies of every nested value.

---

# What is a Deep Copy?

A deep copy recursively creates independent copies of nested structures.

Conceptually:

```text
original ──→ Object A
              └── address ──→ Object B

deepCopy ──→ Object C
              └── address ──→ Object D
```

No nested object is shared.

Therefore:

```js
deepCopy.address.city = "Pune";
```

does not modify:

```js
original.address.city
```

---

# structuredClone()

Modern JavaScript provides `structuredClone()` for deep-cloning many structured values.

```js
const original = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
  },

  skills: [
    "JavaScript",
    "React",
  ],
};

const copy = structuredClone(
  original
);

copy.address.city = "Pune";

console.log(
  original.address.city
); // Bengaluru

console.log(
  copy.address.city
); // Pune
```

References:

```js
console.log(
  original === copy
); // false

console.log(
  original.address === copy.address
); // false
```

## structuredClone Supports More Than JSON

It can clone many built-in structured data types, including common cases such as:

- objects
- arrays
- Date
- Map
- Set
- typed arrays
- circular references

However, it does not clone every JavaScript value. Functions, for example, cannot be cloned with `structuredClone()`.

---

# JSON.stringify / JSON.parse Copy

You may see:

```js
const copy = JSON.parse(
  JSON.stringify(original)
);
```

This can appear to deep-copy simple JSON-compatible data.

But it has limitations and should not be treated as a general-purpose deep-cloning solution.

For example:

- `undefined` properties can be lost
- functions are not preserved
- Symbol values are not preserved normally
- Date becomes a string
- Map and Set are not represented normally
- BigInt causes serialization problems
- circular references throw an error

Example:

```js
const original = {
  createdAt: new Date(),
};

const copy = JSON.parse(
  JSON.stringify(original)
);

console.log(
  typeof copy.createdAt
); // "string"
```

Prefer `structuredClone()` when its supported semantics match what you need.

---

# React State and Nested Objects

Suppose state looks like:

```js
const user = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
  },
};
```

This creates a new outer object:

```js
const updated = {
  ...user,
};
```

But this mutation is still dangerous:

```js
updated.address.city = "Pune";
```

because `address` is shared.

## Correct Nested Immutable Update

Create new objects along the path being changed:

```js
const updated = {
  ...user,

  address: {
    ...user.address,
    city: "Pune",
  },
};
```

Now:

```text
user ──→ old outer object
          └── old address

updated ──→ new outer object
             └── new address
```

This is a fundamental React state-update pattern.

## Do You Always Need a Full Deep Clone?

No.

This is important.

If only one nested branch changes, you usually copy the objects along that changed path rather than cloning the entire application state.

Example:

```js
const updated = {
  ...user,
  address: {
    ...user.address,
    city: "Pune",
  },
};
```

This preserves unchanged references where appropriate and creates new references where data changed.

A full deep clone can be unnecessary and expensive for large data structures.

---

# Arrays with Nested Objects

```js
const users = [
  {
    id: 1,
    name: "Vikash",
  },
];
```

This:

```js
const copy = [...users];
```

creates a new array but shares the user object.

```js
console.log(
  users === copy
); // false

console.log(
  users[0] === copy[0]
); // true
```

To update one user immutably:

```js
const updatedUsers = users.map(
  (user) =>
    user.id === 1
      ? {
          ...user,
          name: "Kumar",
        }
      : user
);
```

Now the changed user receives a new object reference.

---

# Shallow vs Deep Copy

| Shallow Copy | Deep Copy |
| --- | --- |
| New outer container | New outer container |
| Nested references may be shared | Nested structures are independently copied |
| Usually cheaper | Usually more expensive |
| Spread/Object.assign commonly used | structuredClone for supported data |
| Often enough for targeted immutable updates | Useful when truly independent nested data is needed |

## Interview Perspective

**What is a shallow copy?**

A shallow copy creates a new outer object or array while nested object references remain shared.

**What is a deep copy?**

A deep copy creates independent copies of nested structures so mutations to the copied nested data do not affect the original.

**Does spread create a deep copy?**

No. Object and array spread create shallow copies.

**Should React state always be deep-cloned?**

No. Usually you create new references only along the path being updated. Deep-cloning the entire state is often unnecessary.

## Key Takeaways

- Assignment of an object does not create a copy.
- Spread creates a shallow copy.
- Shallow copies share nested object references.
- Deep copies create independent nested structures.
- `structuredClone()` supports many modern deep-cloning use cases.
- JSON serialization is not a general-purpose cloning solution.
- React updates should copy every changed level of nested state.
- Full deep cloning is not required for every immutable update.
