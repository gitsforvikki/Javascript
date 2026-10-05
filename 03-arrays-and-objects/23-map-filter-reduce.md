# Lesson 23 — map, filter and reduce

`map()`, `filter()`, and `reduce()` are among the most important JavaScript array methods.

All three are **higher-order functions** because they receive callback functions.

A useful mental model:

```text
map    → transform
filter → select
reduce → combine
```

---

# 1. map()

`map()` transforms every element and returns a **new array**.

## Syntax

```js
array.map((element, index, array) => {
  return transformedValue;
});
```

Example:

```js
const numbers = [1, 2, 3, 4];

const doubled = numbers.map(
  (number) => number * 2
);

console.log(doubled);
// [2, 4, 6, 8]
```

Flow:

```text
[1, 2, 3, 4]
      ↓ map
[2, 4, 6, 8]
```

The output array has the same number of positions as the input array.

## Transforming Objects

```js
const users = [
  { id: 1, name: "Vikash" },
  { id: 2, name: "Rahul" },
];

const names = users.map(
  (user) => user.name
);

console.log(names);
// ["Vikash", "Rahul"]
```

## Updating One Object Immutably

```js
const updatedUsers = users.map((user) =>
  user.id === 1
    ? { ...user, name: "Vikash Kumar" }
    : user
);
```

This pattern is common in React state updates.

## Common map Mistake

Forgetting `return` with braces:

```js
const doubled = numbers.map((number) => {
  number * 2;
});

console.log(doubled);
// [undefined, undefined, undefined, undefined]
```

Correct:

```js
const doubled = numbers.map((number) => {
  return number * 2;
});
```

or:

```js
const doubled = numbers.map(
  (number) => number * 2
);
```

---

# 2. filter()

`filter()` selects elements whose callback returns a truthy value.

It returns a **new array**.

## Syntax

```js
array.filter((element, index, array) => {
  return condition;
});
```

Example:

```js
const numbers = [1, 2, 3, 4, 5, 6];

const evenNumbers = numbers.filter(
  (number) => number % 2 === 0
);

console.log(evenNumbers);
// [2, 4, 6]
```

Flow:

```text
[1, 2, 3, 4, 5, 6]
        ↓ filter
condition: number % 2 === 0
        ↓
[2, 4, 6]
```

## Filtering Objects

```js
const users = [
  { id: 1, active: true },
  { id: 2, active: false },
  { id: 3, active: true },
];

const activeUsers = users.filter(
  (user) => user.active
);
```

## Removing an Item

```js
const remainingUsers = users.filter(
  (user) => user.id !== 2
);
```

This is a common immutable deletion pattern in React.

---

# 3. reduce()

`reduce()` processes an array and combines its elements into a final accumulated result.

That result can be:

- number
- string
- object
- array
- Map
- almost any other value

## Syntax

```js
array.reduce(
  (accumulator, currentValue, index, array) => {
    return updatedAccumulator;
  },
  initialValue
);
```

## Sum Example

```js
const numbers = [10, 20, 30];

const total = numbers.reduce(
  (accumulator, number) => {
    return accumulator + number;
  },
  0
);

console.log(total); // 60
```

Execution:

```text
Initial accumulator = 0

Step 1:
0 + 10 = 10

Step 2:
10 + 20 = 30

Step 3:
30 + 30 = 60

Final result = 60
```

## Short Version

```js
const total = numbers.reduce(
  (sum, number) => sum + number,
  0
);
```

## Why the Initial Value Matters

Prefer supplying an appropriate initial value.

```js
const total = numbers.reduce(
  (sum, number) => sum + number,
  0
);
```

Without an initial value, `reduce` uses the first array element as the initial accumulator and begins from the next element.

This also means:

```js
[].reduce((a, b) => a + b);
```

throws a `TypeError`.

But:

```js
[].reduce((a, b) => a + b, 0);
```

returns:

```text
0
```

## reduce to an Object

Suppose:

```js
const users = [
  { name: "A", role: "frontend" },
  { name: "B", role: "backend" },
  { name: "C", role: "frontend" },
];
```

Count users by role:

```js
const counts = users.reduce(
  (acc, user) => {
    acc[user.role] =
      (acc[user.role] ?? 0) + 1;

    return acc;
  },
  {}
);

console.log(counts);
```

Result:

```js
{
  frontend: 2,
  backend: 1
}
```

This example mutates the local accumulator object for efficiency. That does not mutate the original `users` array.

## Choosing Between map, filter and reduce

Ask what result you want.

### Transform every item?

Use `map`.

```js
const names = users.map(
  (user) => user.name
);
```

### Keep only matching items?

Use `filter`.

```js
const activeUsers = users.filter(
  (user) => user.active
);
```

### Produce one accumulated result?

Use `reduce`.

```js
const total = prices.reduce(
  (sum, price) => sum + price,
  0
);
```

Mental model:

```text
MAP
[A, B, C]
   ↓
[A', B', C']

FILTER
[A, B, C]
   ↓
[A, C]

REDUCE
[A, B, C]
   ↓
single accumulated result
```

## Chaining Array Methods

Methods can be combined.

```js
const products = [
  { name: "Laptop", price: 80000, active: true },
  { name: "Phone", price: 40000, active: false },
  { name: "Monitor", price: 20000, active: true },
];

const total = products
  .filter((product) => product.active)
  .map((product) => product.price)
  .reduce((sum, price) => sum + price, 0);

console.log(total); // 100000
```

Flow:

```text
products
   ↓ filter active
active products
   ↓ map price
[80000, 20000]
   ↓ reduce
100000
```

## React Connection

### Rendering lists

```jsx
users.map((user) => (
  <UserCard
    key={user.id}
    user={user}
  />
));
```

### Removing state data

```js
setUsers(
  users.filter((user) => user.id !== id)
);
```

### Updating state data

```js
setUsers(
  users.map((user) =>
    user.id === id
      ? { ...user, active: true }
      : user
  )
);
```

These patterns are extremely common in frontend development.

## Interview Perspective

**Difference between map and forEach?**

`map()` creates a new array from callback results. `forEach()` is primarily used to execute an operation for each item and returns `undefined`.

**Difference between map and filter?**

`map()` transforms elements. `filter()` selects elements based on a condition.

**What does reduce do?**

`reduce()` processes array elements while carrying an accumulator and returns the final accumulated value.

## Key Takeaways

- `map` transforms.
- `filter` selects.
- `reduce` accumulates.
- All three receive callback functions.
- `map` and `filter` return new arrays.
- `reduce` can produce many kinds of results.
- Supplying an initial value to `reduce` is usually clearer and safer.
- These methods are heavily used in React applications.
