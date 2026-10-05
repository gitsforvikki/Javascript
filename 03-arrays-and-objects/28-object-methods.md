# Lesson 28 — Object Methods and Property Access

JavaScript provides useful built-in methods for inspecting, transforming and working with objects.

This lesson focuses on the object operations you will use most often.

## Property Access Recap

Given:

```js
const user = {
  name: "Vikash",
  role: "Developer",
};
```

Dot notation:

```js
user.name;
```

Bracket notation:

```js
user["name"];
```

Dynamic access:

```js
const key = "role";

user[key];
```

## Checking Whether a Property Exists

### in Operator

```js
const user = {
  name: "Vikash",
};

console.log("name" in user); // true
console.log("role" in user); // false
```

The `in` operator also considers inherited properties.

### Object.hasOwn()

To check whether the object itself owns a property:

```js
Object.hasOwn(user, "name");
```

Example:

```js
console.log(
  Object.hasOwn(user, "name")
); // true
```

For modern code, `Object.hasOwn()` is a clear way to check own properties.

## Object.keys()

Returns an array containing the object's own enumerable string property names.

```js
const user = {
  name: "Vikash",
  role: "Developer",
  active: true,
};

const keys = Object.keys(user);

console.log(keys);
```

Output:

```js
["name", "role", "active"]
```

Because the result is an array, array methods can be used.

```js
Object.keys(user).forEach((key) => {
  console.log(key);
});
```

## Object.values()

Returns the corresponding property values.

```js
const values = Object.values(user);

console.log(values);
```

Output:

```js
["Vikash", "Developer", true]
```

## Object.entries()

Returns key-value pairs as nested arrays.

```js
const entries = Object.entries(user);

console.log(entries);
```

Result:

```js
[
  ["name", "Vikash"],
  ["role", "Developer"],
  ["active", true]
]
```

This works nicely with destructuring:

```js
Object.entries(user).forEach(
  ([key, value]) => {
    console.log(key, value);
  }
);
```

Mental model:

```text
Object
{
  name: "Vikash",
  role: "Developer"
}

       ↓ Object.entries()

[
  ["name", "Vikash"],
  ["role", "Developer"]
]
```

## Object.fromEntries()

`Object.fromEntries()` performs the reverse kind of transformation.

```js
const entries = [
  ["name", "Vikash"],
  ["role", "Developer"],
];

const user = Object.fromEntries(entries);

console.log(user);
```

Result:

```js
{
  name: "Vikash",
  role: "Developer"
}
```

## Useful Transformation Pattern

Suppose:

```js
const prices = {
  laptop: 1000,
  phone: 500,
};
```

Increase every price:

```js
const updatedPrices = Object.fromEntries(
  Object.entries(prices).map(
    ([product, price]) => [
      product,
      price * 1.1,
    ]
  )
);
```

Flow:

```text
Object
  ↓ Object.entries()
Array of pairs
  ↓ map()
transformed pairs
  ↓ Object.fromEntries()
New Object
```

## Object.assign()

`Object.assign()` copies enumerable own properties from source objects into a target object.

```js
const user = {
  name: "Vikash",
};

const details = {
  role: "Developer",
};

const result = Object.assign(
  {},
  user,
  details
);

console.log(result);
```

Result:

```js
{
  name: "Vikash",
  role: "Developer"
}
```

In modern application code, spread syntax is often more readable:

```js
const result = {
  ...user,
  ...details,
};
```

Both approaches here create only a **shallow copy**.

## Object.freeze()

`Object.freeze()` prevents common direct additions, removals and modifications of the object's own properties.

```js
const config = Object.freeze({
  api: "/api",
});

config.api = "/new-api";
```

In strict mode, attempting prohibited mutations can throw a `TypeError`; otherwise they may fail silently.

Important: `Object.freeze()` is **shallow**.

```js
const config = Object.freeze({
  user: {
    name: "Vikash",
  },
});

config.user.name = "Kumar";
```

The nested object is not automatically frozen.

## Object.seal()

`Object.seal()` prevents adding or deleting properties, while existing writable properties can still be changed.

```js
const user = Object.seal({
  name: "Vikash",
});

user.name = "Kumar"; // allowed
```

But adding a new property is prevented:

```js
user.role = "Developer";
```

## Methods Defined Inside Objects

```js
const user = {
  name: "Vikash",

  greet() {
    return `Hello ${this.name}`;
  },
};

console.log(user.greet());
```

Here, `greet` is a method.

The value of `this` depends on **how a function is called**, not simply where the method text appears.

We will study `this` deeply in Section 5.

## React / API Connection

API responses often contain objects whose keys or values need transformation.

Example:

```js
const formData = {
  name: "Vikash",
  email: "test@example.com",
};

const hasEmptyField = Object.values(
  formData
).some((value) => !value);
```

Or rendering key-value data:

```jsx
Object.entries(user).map(
  ([key, value]) => (
    <p key={key}>
      {key}: {String(value)}
    </p>
  )
);
```

## Interview Perspective

**Difference between Object.keys, Object.values and Object.entries?**

- `Object.keys()` returns property names.
- `Object.values()` returns property values.
- `Object.entries()` returns key-value pairs.

**What does Object.fromEntries do?**

It creates an object from an iterable of key-value pairs.

**Is Object.freeze deep?**

No. `Object.freeze()` is shallow unless nested objects are separately frozen.

## Key Takeaways

- Dot notation is simple for known property names.
- Bracket notation supports dynamic keys.
- `Object.keys()` returns keys.
- `Object.values()` returns values.
- `Object.entries()` returns key-value pairs.
- `Object.fromEntries()` creates an object from entries.
- `Object.assign()` and spread commonly create shallow copies.
- `Object.freeze()` and `Object.seal()` are shallow controls.
