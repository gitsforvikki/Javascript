# Lesson 05 — Primitive vs Reference Values

Understanding how values are copied is important when working with JavaScript.

## Primitive Values

Primitive values are copied **by value**.

```js
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
console.log(b); // 20
```

Changing `b` does not change `a`.

Conceptually:

```text
a → 10
b → 10

b = 20

a → 10
b → 20
```

## Objects and References

Objects are handled through references.

```js
const user1 = {
  name: "Vikash",
};

const user2 = user1;

user2.name = "Kumar";

console.log(user1.name); // Kumar
```

Both variables refer to the same object.

Conceptually:

```text
user1 ──┐
        ├──→ { name: "Vikash" }
user2 ──┘
```

Therefore, changing the object through `user2` is visible through `user1`.

## Reference Equality

```js
const a = { name: "Vikash" };
const b = { name: "Vikash" };

console.log(a === b); // false
```

Although the contents look identical, they are two different objects.

But:

```js
const a = { name: "Vikash" };
const b = a;

console.log(a === b); // true
```

Both variables reference the same object.

## Why This Matters in React

React state frequently contains objects and arrays.

Mutating an existing object can cause problems because React commonly relies on reference changes when deciding whether data has changed.

Instead of:

```js
user.name = "Kumar";
```

you will commonly create a new object:

```js
const updatedUser = {
  ...user,
  name: "Kumar",
};
```

We will study references, immutability, shallow copies and React rendering much more deeply later.

## Important Terminology Note

You will often hear **"objects are passed by reference."**

More precisely, JavaScript is **pass-by-value**: for objects, the value being copied/passed is a reference to the object. This is why two variables can point to the same object.

## Key Takeaways

- Primitive values behave like independent copied values.
- Objects can be accessed through shared references.
- Two separate objects are not equal just because their contents match.
- Reference behavior is very important for arrays, objects and React state.

## Interview Quick Answer

**What is the difference between primitive and reference values?**

Primitive values are copied as independent values. With objects, the copied value is a reference to the same object, so multiple variables can access and modify that same object.
