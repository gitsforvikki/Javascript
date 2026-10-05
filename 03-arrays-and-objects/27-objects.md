# Lesson 27 — Objects and Object Fundamentals

Objects are one of the most important parts of JavaScript.

They store related data as **key-value pairs**.

## Creating an Object

```js
const user = {
  name: "Vikash",
  role: "Developer",
  experience: 1.5,
};
```

Conceptually:

```text
user
 │
 ├── name       → "Vikash"
 ├── role       → "Developer"
 └── experience → 1.5
```

The keys are called **properties**.

## Accessing Properties

### Dot notation

```js
console.log(user.name);
console.log(user.role);
```

### Bracket notation

```js
console.log(user["name"]);
```

Both can access object properties.

## When Bracket Notation Is Necessary

Bracket notation is useful when the property name comes from a variable.

```js
const property = "role";

console.log(user[property]);
// Developer
```

This would mean something different:

```js
user.property
```

It looks for a literal property named `property`.

Bracket notation also handles property names that are not convenient for dot syntax.

```js
const person = {
  "full-name": "Vikash Kumar",
};

console.log(person["full-name"]);
```

## Adding Properties

```js
const user = {
  name: "Vikash",
};

user.role = "Developer";

console.log(user);
```

## Updating Properties

```js
user.role = "Full Stack Developer";
```

## Deleting Properties

```js
delete user.role;
```

Use deletion intentionally. In immutable application logic, creating a new object without the property is often preferable.

## const Objects Can Still Change

This is important:

```js
const user = {
  name: "Vikash",
};

user.name = "Kumar"; // allowed
```

But:

```js
user = {
  name: "Rahul",
}; // TypeError
```

Why?

`const` prevents reassignment of the variable. It does **not** make the object immutable.

Mental model:

```text
const user ──→ object
     │
     └── reference cannot be reassigned

object properties
     └── can still be mutated
```

## Nested Objects

Objects can contain other objects.

```js
const user = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
    country: "India",
  },
};

console.log(user.address.city);
```

## Arrays Inside Objects

```js
const developer = {
  name: "Vikash",
  skills: [
    "JavaScript",
    "React",
    "Next.js",
  ],
};

console.log(developer.skills[0]);
```

## Objects Inside Arrays

Very common API structure:

```js
const users = [
  {
    id: 1,
    name: "Vikash",
  },
  {
    id: 2,
    name: "Rahul",
  },
];
```

Then:

```js
const names = users.map(
  (user) => user.name
);
```

## Methods

Objects can store functions.

```js
const user = {
  name: "Vikash",

  greet() {
    return "Hello";
  },
};

console.log(user.greet());
```

A function stored as an object property is commonly called a **method**.

We will study object methods and `this` in more detail separately.

## Computed Property Names

Property names can be calculated dynamically.

```js
const property = "role";

const user = {
  name: "Vikash",
  [property]: "Developer",
};

console.log(user.role);
// Developer
```

This is useful when property names are dynamic.

## Property Shorthand

If a variable name matches the desired property name:

```js
const name = "Vikash";
const role = "Developer";

const user = {
  name,
  role,
};
```

Equivalent to:

```js
const user = {
  name: name,
  role: role,
};
```

## Objects Are Reference Values

```js
const user1 = {
  name: "Vikash",
};

const user2 = user1;

user2.name = "Kumar";

console.log(user1.name); // Kumar
```

Both variables hold the same object reference value.

Conceptually:

```text
user1 ──┐
        ├──→ { name: "Kumar" }
user2 ──┘
```

## Object Equality

```js
const a = {
  name: "Vikash",
};

const b = {
  name: "Vikash",
};

console.log(a === b); // false
```

They have identical content but are different objects.

```js
const a = {
  name: "Vikash",
};

const b = a;

console.log(a === b); // true
```

Now both variables contain the same reference.

## React Connection

React state commonly contains objects.

Avoid direct mutation:

```js
user.name = "Kumar";
setUser(user);
```

Prefer creating a new object:

```js
setUser({
  ...user,
  name: "Kumar",
});
```

Reference identity is extremely important for React rendering and memoization.

## Interview Perspective

**Why can a const object's properties change?**

Because `const` prevents reassignment of the variable binding; it does not freeze or make the referenced object immutable.

**How are objects compared with ===?**

Objects are compared by reference identity, not by recursively comparing their contents.

## Key Takeaways

- Objects store key-value pairs.
- Properties can be accessed with dot or bracket notation.
- Bracket notation supports dynamic property names.
- Objects can contain arrays, functions and other objects.
- `const` does not make an object immutable.
- Objects are reference values.
- Object equality is based on reference identity.
- Object reference behavior is fundamental to React state.
