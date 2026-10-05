# Lesson 29 — Object Destructuring

Object destructuring extracts properties from an object into variables.

Unlike array destructuring, object destructuring is based on **property names**, not positions.

## Basic Object Destructuring

Without destructuring:

```js
const user = {
  name: "Vikash",
  role: "Developer",
};

const name = user.name;
const role = user.role;
```

With destructuring:

```js
const { name, role } = user;

console.log(name); // Vikash
console.log(role); // Developer
```

Mental model:

```text
{
  name: "Vikash",
  role: "Developer"
}
    ↓ property matching
name → "Vikash"
role → "Developer"
```

## Order Does Not Matter

```js
const {
  role,
  name,
} = user;
```

This still works because object destructuring matches property names.

Compare:

```text
Array destructuring  → position
Object destructuring → property name
```

## Renaming Variables

Sometimes you want a different local variable name.

```js
const user = {
  name: "Vikash",
};

const {
  name: userName,
} = user;

console.log(userName); // Vikash
```

Here:

```text
property name → name
variable name → userName
```

The syntax means:

```js
property: variableName
```

## Default Values

```js
const user = {
  name: "Vikash",
};

const {
  name,
  role = "Developer",
} = user;

console.log(role); // Developer
```

Defaults apply when the property value is `undefined`.

```js
const user = {
  role: null,
};

const {
  role = "Developer",
} = user;

console.log(role); // null
```

## Rename + Default

```js
const user = {};

const {
  name: userName = "Guest",
} = user;

console.log(userName); // Guest
```

## Rest Properties

```js
const user = {
  id: 1,
  name: "Vikash",
  role: "Developer",
  active: true,
};

const {
  id,
  ...details
} = user;

console.log(id); // 1

console.log(details);
```

Result:

```js
{
  name: "Vikash",
  role: "Developer",
  active: true
}
```

This is useful for creating a new object without selected properties.

## Nested Destructuring

```js
const user = {
  name: "Vikash",

  address: {
    city: "Bengaluru",
    country: "India",
  },
};

const {
  address: {
    city,
    country,
  },
} = user;

console.log(city);    // Bengaluru
console.log(country); // India
```

Important:

```js
const {
  address: {
    city,
  },
} = user;
```

creates `city`, but it does not create a separate local variable named `address`.

## Destructuring Function Parameters

Very common in modern JavaScript:

```js
function printUser({
  name,
  role,
}) {
  console.log(name);
  console.log(role);
}

printUser({
  name: "Vikash",
  role: "Developer",
});
```

Instead of:

```js
function printUser(user) {
  console.log(user.name);
  console.log(user.role);
}
```

Both are valid. Parameter destructuring is useful when the function only needs specific properties.

## React Props

This pattern is everywhere in React.

Without destructuring:

```jsx
function UserCard(props) {
  return (
    <h2>{props.name}</h2>
  );
}
```

With destructuring:

```jsx
function UserCard({
  name,
  role,
}) {
  return (
    <>
      <h2>{name}</h2>
      <p>{role}</p>
    </>
  );
}
```

## Destructuring State or API Data

```js
const response = {
  data: {
    user: {
      id: 1,
      name: "Vikash",
    },
  },
};

const {
  data: {
    user: {
      id,
      name,
    },
  },
} = response;
```

Deep destructuring is possible, but avoid making code difficult to read.

Sometimes this is clearer:

```js
const user = response.data.user;

const {
  id,
  name,
} = user;
```

## Destructuring Does Not Deep Clone

This is important.

```js
const user = {
  profile: {
    name: "Vikash",
  },
};

const { profile } = user;

profile.name = "Kumar";

console.log(user.profile.name);
// Kumar
```

Why?

`profile` contains the same nested object reference.

```text
user.profile ──┐
               ├──→ same object
profile ───────┘
```

Destructuring extracts values. It does not automatically deep-copy them.

## Common Mistake: Missing Nested Property

This can fail:

```js
const user = {};

const {
  address: {
    city,
  },
} = user;
```

Because `address` is `undefined`, JavaScript cannot destructure `city` from it.

A default can help:

```js
const {
  address: {
    city,
  } = {},
} = user;
```

For safe property access, optional chaining is often more appropriate and will be covered in Lesson 31.

## Interview Perspective

**Difference between array and object destructuring?**

Array destructuring primarily matches by position. Object destructuring matches by property name.

**Does destructuring copy nested objects?**

It copies/extracts the property value. If that value is an object reference, the same referenced object remains shared.

## Key Takeaways

- Object destructuring extracts properties by name.
- Property order does not matter.
- Variables can be renamed.
- Defaults can be provided.
- Rest syntax can collect remaining properties.
- Nested destructuring is supported.
- Function parameters and React props commonly use destructuring.
- Destructuring does not deep-clone referenced objects.
