# Lesson 24 — find, findIndex, some and every

These array methods help answer different questions about array contents.

Mental model:

```text
find      → give me the first matching value
findIndex → give me the index of the first match
some      → does at least one match?
every     → do all match?
```

All four accept callback functions and do not mutate the original array.

---

# 1. find()

`find()` returns the **first element** for which the callback returns a truthy value.

```js
const users = [
  { id: 1, name: "Vikash" },
  { id: 2, name: "Rahul" },
  { id: 3, name: "Aman" },
];

const user = users.find(
  (user) => user.id === 2
);

console.log(user);
```

Result:

```js
{
  id: 2,
  name: "Rahul"
}
```

If no element matches:

```js
const user = users.find(
  (user) => user.id === 100
);

console.log(user); // undefined
```

## find vs filter

This is an important distinction.

```js
const numbers = [1, 2, 4, 6];
```

`find`:

```js
numbers.find(
  (number) => number % 2 === 0
);
```

returns:

```text
2
```

Only the first match.

`filter`:

```js
numbers.filter(
  (number) => number % 2 === 0
);
```

returns:

```js
[2, 4, 6]
```

Use:

```text
find   → first matching element
filter → all matching elements
```

---

# 2. findIndex()

`findIndex()` returns the index of the first matching element.

```js
const users = [
  { id: 10, name: "Vikash" },
  { id: 20, name: "Rahul" },
];

const index = users.findIndex(
  (user) => user.id === 20
);

console.log(index); // 1
```

If no match exists:

```js
console.log(
  users.findIndex((user) => user.id === 100)
); // -1
```

Notice the difference:

```text
find()      → undefined when missing
findIndex() → -1 when missing
```

## Practical Example

```js
const index = users.findIndex(
  (user) => user.id === 20
);

if (index !== -1) {
  console.log("User exists");
}
```

---

# 3. some()

`some()` checks whether **at least one** element satisfies a condition.

It returns a boolean.

```js
const users = [
  { name: "A", active: false },
  { name: "B", active: true },
  { name: "C", active: false },
];

const hasActiveUser = users.some(
  (user) => user.active
);

console.log(hasActiveUser); // true
```

Mental model:

```text
Does ANY item match?

[false, true, false]
        ↓
       true
```

## Practical Permission Example

```js
const roles = ["user", "editor"];

const canEdit = roles.some(
  (role) => role === "admin" || role === "editor"
);

console.log(canEdit); // true
```

---

# 4. every()

`every()` checks whether **all elements** satisfy a condition.

```js
const numbers = [2, 4, 6, 8];

const allEven = numbers.every(
  (number) => number % 2 === 0
);

console.log(allEven); // true
```

Another example:

```js
const forms = [
  { valid: true },
  { valid: true },
  { valid: false },
];

const formIsValid = forms.every(
  (field) => field.valid
);

console.log(formIsValid); // false
```

Mental model:

```text
Do ALL items match?

[true, true, false]
         ↓
        false
```

## Short-Circuit Behavior

`some()` and `every()` can stop early.

For `some()`:

```text
false
false
true  ← match found
STOP
```

Once one truthy result is found, the final answer must be `true`.

For `every()`:

```text
true
true
false ← failure found
STOP
```

Once one falsy result is found, the final answer must be `false`.

`find()` also stops after finding its first matching element.

This is different from `filter()`, which must determine all matches.

## Empty Array Behavior

An interesting interview case:

```js
[].some(() => true);  // false
[].every(() => false); // true
```

Why?

`some` asks:

> Is there at least one matching element?

There are no elements, so the answer is false.

`every` asks:

> Is there an element that violates the condition?

There is no violating element, so `every` returns true for an empty array.

You do not need to memorize the mathematical terminology; remember the behavior.

## React Connection

### Check whether an item is selected

```js
const isSelected = selectedItems.some(
  (item) => item.id === product.id
);
```

### Validate all fields

```js
const isValid = fields.every(
  (field) => field.isValid
);
```

### Find an item

```js
const selectedUser = users.find(
  (user) => user.id === selectedId
);
```

## Interview Perspective

**Difference between find and filter?**

`find()` returns the first matching element or `undefined`. `filter()` returns a new array containing all matching elements.

**Difference between some and every?**

`some()` returns true when at least one element passes the test. `every()` returns true only when all elements pass the test.

## Key Takeaways

- `find` returns the first matching element.
- `findIndex` returns the first matching index.
- `some` checks whether at least one item matches.
- `every` checks whether all items match.
- `find`, `some`, and `every` can stop once their result is known.
- These methods do not mutate the original array.
