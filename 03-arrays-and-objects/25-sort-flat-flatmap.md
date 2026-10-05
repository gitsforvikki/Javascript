# Lesson 25 — sort, flat and flatMap

This lesson covers three useful array operations:

- `sort()` — reorder elements
- `flat()` — flatten nested arrays
- `flatMap()` — map and then flatten one level

The most important concept here is understanding how `sort()` behaves.

---

# 1. sort()

`sort()` sorts the elements of an array.

```js
const fruits = [
  "banana",
  "apple",
  "mango",
];

fruits.sort();

console.log(fruits);
// ["apple", "banana", "mango"]
```

## Important: sort() Mutates

`sort()` changes the original array.

```js
const numbers = [3, 1, 2];

const sorted = numbers.sort();

console.log(numbers); // [1, 2, 3]
console.log(sorted);  // [1, 2, 3]

console.log(numbers === sorted); // true
```

`sort()` returns the same array reference after sorting it.

Conceptually:

```text
numbers ──→ [3, 1, 2]

sort()

numbers ──→ [1, 2, 3]
             same array
```

## Default sort() Can Surprise You

Consider:

```js
const numbers = [1, 10, 2, 20, 3];

numbers.sort();

console.log(numbers);
```

You might expect:

```js
[1, 2, 3, 10, 20]
```

But default sorting compares values as strings.

The result is effectively based on string ordering:

```js
[1, 10, 2, 20, 3]
```

Therefore, use a comparator for numeric sorting.

## Numeric Ascending Sort

```js
const numbers = [10, 2, 30, 5];

numbers.sort((a, b) => a - b);

console.log(numbers);
// [2, 5, 10, 30]
```

## Numeric Descending Sort

```js
numbers.sort((a, b) => b - a);
```

## Understanding the Comparator

The comparator:

```js
(a, b) => a - b
```

conceptually tells JavaScript:

```text
result < 0 → a comes before b
result > 0 → b comes before a
result = 0 → considered equal for ordering
```

For ascending numbers:

```js
(a, b) => a - b
```

For descending numbers:

```js
(a, b) => b - a
```

## Sorting Objects

```js
const users = [
  { name: "A", age: 30 },
  { name: "B", age: 20 },
  { name: "C", age: 25 },
];

users.sort(
  (a, b) => a.age - b.age
);
```

Result order:

```text
B → 20
C → 25
A → 30
```

## Sorting Strings More Intentionally

For user-facing strings, `localeCompare()` is useful.

```js
const users = [
  { name: "Vikash" },
  { name: "Aman" },
  { name: "Rahul" },
];

users.sort(
  (a, b) => a.name.localeCompare(b.name)
);
```

## Non-Mutating Sort

Because `sort()` mutates, older/common code often copies first:

```js
const sorted = [...numbers].sort(
  (a, b) => a - b
);
```

Modern JavaScript provides `toSorted()`:

```js
const numbers = [3, 1, 2];

const sorted = numbers.toSorted(
  (a, b) => a - b
);

console.log(numbers); // [3, 1, 2]
console.log(sorted);  // [1, 2, 3]
```

For immutable application state, `toSorted()` is often clearer when supported by your target runtime.

---

# 2. flat()

`flat()` creates a new array with nested array elements flattened to a specified depth.

Example:

```js
const numbers = [
  1,
  2,
  [3, 4],
];

const result = numbers.flat();

console.log(result);
// [1, 2, 3, 4]
```

Default depth:

```js
1
```

## Nested More Than One Level

```js
const numbers = [
  1,
  [2, [3, [4]]],
];
```

Using:

```js
numbers.flat();
```

produces:

```js
[1, 2, [3, [4]]]
```

Using depth 2:

```js
numbers.flat(2);
```

produces:

```js
[1, 2, 3, [4]]
```

## Flatten All Nested Levels

```js
numbers.flat(Infinity);
```

Result:

```js
[1, 2, 3, 4]
```

## Does flat Mutate?

No.

```js
const nested = [1, [2, 3]];

const flattened = nested.flat();

console.log(nested);
// [1, [2, 3]]

console.log(flattened);
// [1, 2, 3]
```

Like other copying operations, `flat()` does not deep-clone objects inside the array.

---

# 3. flatMap()

`flatMap()` performs:

```text
map()
  +
flat(1)
```

Conceptually:

```js
array.flatMap(callback)
```

is similar to:

```js
array.map(callback).flat(1)
```

## Example

```js
const numbers = [1, 2, 3];

const result = numbers.flatMap(
  (number) => [number, number * 2]
);

console.log(result);
```

Output:

```js
[1, 2, 2, 4, 3, 6]
```

Without flattening, `map` would produce:

```js
[
  [1, 2],
  [2, 4],
  [3, 6]
]
```

`flatMap` flattens those results by one level.

## Practical Example

Suppose each developer has multiple skills.

```js
const developers = [
  {
    name: "A",
    skills: ["React", "JavaScript"],
  },
  {
    name: "B",
    skills: ["Node.js", "MongoDB"],
  },
];

const skills = developers.flatMap(
  (developer) => developer.skills
);

console.log(skills);
```

Result:

```js
[
  "React",
  "JavaScript",
  "Node.js",
  "MongoDB"
]
```

## flatMap Only Flattens One Level

```js
const values = [1, 2];

const result = values.flatMap(
  (value) => [[value]]
);

console.log(result);
// [[1], [2]]
```

It is not equivalent to `flat(Infinity)`.

## React Connection

Suppose categories contain products:

```js
const categories = [
  {
    name: "Electronics",
    products: ["Laptop", "Phone"],
  },
  {
    name: "Accessories",
    products: ["Mouse", "Keyboard"],
  },
];
```

Getting one product list:

```js
const products = categories.flatMap(
  (category) => category.products
);
```

For sorting state data, remember not to mutate existing state accidentally.

Avoid:

```js
users.sort((a, b) => a.name.localeCompare(b.name));
```

Prefer a non-mutating approach:

```js
const sortedUsers = users.toSorted(
  (a, b) => a.name.localeCompare(b.name)
);
```

or copy first when appropriate:

```js
const sortedUsers = [...users].sort(
  (a, b) => a.name.localeCompare(b.name)
);
```

## Interview Perspective

**Why can sort() give unexpected results for numbers?**

Without a comparator, `sort()` compares elements using string-based ordering. For numeric ordering, provide a comparator such as `(a, b) => a - b`.

**Does sort mutate the array?**

Yes. `sort()` sorts the original array in place. `toSorted()` is the modern non-mutating alternative.

**What is the difference between flat and flatMap?**

`flat()` flattens existing nested arrays to a chosen depth. `flatMap()` maps each element and then flattens the mapped result by one level.

## Key Takeaways

- `sort()` mutates the original array.
- Default `sort()` is not reliable for numeric ordering without a comparator.
- Use `a - b` for ascending numeric order.
- Use `b - a` for descending numeric order.
- `toSorted()` provides non-mutating sorting.
- `flat()` flattens nested arrays to a specified depth.
- `flatMap()` combines mapping with one-level flattening.
