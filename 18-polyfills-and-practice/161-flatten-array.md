# Lesson 161 — Flatten Nested Arrays

Flattening nested arrays is a common interview problem because it tests:

- recursion
- iteration
- arrays
- depth control
- edge cases
- complexity

The goal is to transform:

```js
[
  1,
  [2, [3, 4]],
  5,
]
```

into:

```js
[
  1,
  2,
  3,
  4,
  5,
]
```

---

## 1. Native Solution

JavaScript provides:

```js
array.flat(
  Infinity
);
```

Example:

```js
const result =
  [
    1,
    [2, [3, 4]],
    5,
  ].flat(
    Infinity
  );
```

For interviews, you are usually asked to implement the behavior manually.

---

## 2. Recursive Mental Model

```text
for each item
↓
is array?
├── no → push value
└── yes → flatten that array
```

---

## 3. Basic Recursive Implementation

```js
function flatten(
  array
) {
  const result = [];

  for (
    const item
    of array
  ) {
    if (
      Array.isArray(
        item
      )
    ) {
      result.push(
        ...flatten(
          item
        )
      );
    } else {
      result.push(
        item
      );
    }
  }

  return result;
}
```

---

## 4. Example

```js
flatten([
  1,
  [2, [3]],
  4,
]);
```

Result:

```js
[
  1,
  2,
  3,
  4,
]
```

---

## 5. Avoid Repeated Spread for Large Data

This:

```js
result.push(
  ...flatten(item)
);
```

is simple, but can create extra temporary arrays and argument expansion.

A stronger version passes one result array through recursion.

---

## 6. More Efficient Recursive Version

```js
function flatten(
  array,
  result = []
) {
  for (
    const item
    of array
  ) {
    if (
      Array.isArray(
        item
      )
    ) {
      flatten(
        item,
        result
      );
    } else {
      result.push(
        item
      );
    }
  }

  return result;
}
```

Now we reuse the same result array.

---

## 7. Flatten to a Specific Depth

Native:

```js
array.flat(2);
```

Custom:

```js
function flattenDepth(
  array,
  depth = 1
) {
  const result = [];

  for (
    const item
    of array
  ) {
    if (
      Array.isArray(
        item
      ) &&
      depth > 0
    ) {
      result.push(
        ...flattenDepth(
          item,
          depth - 1
        )
      );
    } else {
      result.push(
        item
      );
    }
  }

  return result;
}
```

---

## 8. Depth Example

```js
flattenDepth(
  [
    1,
    [2, [3, [4]]],
  ],
  2
);
```

Result:

```js
[
  1,
  2,
  3,
  [4],
]
```

---

## 9. Iterative Solution

Recursion is not the only approach.

```js
function flattenIterative(
  array
) {
  const stack = [
    ...array,
  ];

  const result = [];

  while (
    stack.length
  ) {
    const item =
      stack.pop();

    if (
      Array.isArray(
        item
      )
    ) {
      stack.push(
        ...item
      );
    } else {
      result.push(
        item
      );
    }
  }

  return result.reverse();
}
```

---

## 10. Why reverse()?

Because stack processing is LIFO:

```text
last in
→ first out
```

So values are collected in reverse order.

---

## 11. Sparse Arrays

Native `flat()` removes empty slots at flattened levels.

Example:

```js
[
  1,
  ,
  [2, , 3],
].flat(
  Infinity
);
```

Conceptually becomes:

```js
[
  1,
  2,
  3,
]
```

A `for...of` based implementation may observe holes as `undefined`, so exact native sparse-array semantics require additional care.

This is worth mentioning in interviews.

---

## 12. Interview-Friendly Version

```js
function flatten(
  array
) {
  const result = [];

  function helper(
    items
  ) {
    for (
      let i = 0;
      i < items.length;
      i++
    ) {
      if (
        !(i in items)
      ) {
        continue;
      }

      const item =
        items[i];

      if (
        Array.isArray(
          item
        )
      ) {
        helper(item);
      } else {
        result.push(
          item
        );
      }
    }
  }

  helper(array);

  return result;
}
```

This skips sparse holes explicitly.

---

## 13. Deep Recursion Risk

Extremely deeply nested arrays can cause:

```text
Maximum call stack size exceeded
```

In such cases, the iterative stack solution is safer.

---

## 14. Complexity

If there are `n` total visited elements:

```text
Time:
O(n)

Output space:
O(n)
```

Recursive version also uses call-stack space proportional to nesting depth.

---

## 15. Common Wrong Solution

```js
function flatten(
  array
) {
  return array
    .toString()
    .split(",");
}
```

This is wrong because it:

- converts values to strings
- loses types
- breaks objects
- breaks nested special values

---

## Interview Explanation

> I recursively inspect each element. If it is an array, I recurse; otherwise I push it into one shared result array. For very deep nesting, I can switch to an explicit stack to avoid call-stack overflow. If exact native `flat` behavior matters, I also account for sparse-array holes and depth.

---

## Key Takeaways

- Flattening is naturally recursive.
- Iterative stacks avoid deep recursion limits.
- Depth can be controlled explicitly.
- Avoid string-based tricks.
- Exact native behavior includes sparse-array semantics.
