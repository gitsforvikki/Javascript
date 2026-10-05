# Lesson 150 — Implement Custom `map`

`map()` creates a new array by transforming each present element of the source array.

This is one of the most common JavaScript interview polyfill questions.

It tests:

- callback execution
- immutable transformation
- prototype methods
- sparse arrays
- `thisArg`
- return values

---

## 1. Native map Behavior

```js
const numbers = [
  1,
  2,
  3,
];

const doubled =
  numbers.map(
    (
      value,
      index,
      array
    ) =>
      value * 2
  );
```

Result:

```js
[2, 4, 6]
```

Original array remains unchanged.

---

## 2. Main Difference from forEach

```text
forEach
→ executes callback
→ returns undefined

map
→ executes callback
→ stores callback result
→ returns new array
```

---

## 3. Basic Implementation

```js
Array.prototype.customMap =
  function (
    callback
  ) {
    const result = [];

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      result.push(
        callback(
          this[i],
          i,
          this
        )
      );
    }

    return result;
  };
```

This works for normal dense arrays.

But it has a sparse-array problem.

---

## 4. Sparse Array Behavior

```js
const arr = [
  10,
  ,
  30,
];
```

Native:

```js
const result =
  arr.map(
    (value) =>
      value * 2
  );
```

The result keeps the hole.

Conceptually:

```text
source:
[10, <hole>, 30]

result:
[20, <hole>, 60]
```

---

## 5. Why push() Is Not Accurate for Sparse Arrays

If we do:

```js
result.push(
  transformed
);
```

we create a dense result.

For native-like sparse behavior, pre-create:

```js
const result =
  new Array(
    length
  );
```

Then assign only indexes that exist.

---

## 6. Better Implementation

```js
Array.prototype.customMap =
  function (
    callback,
    thisArg
  ) {
    if (
      this == null
    ) {
      throw new TypeError(
        "Cannot map null or undefined"
      );
    }

    if (
      typeof callback !==
      "function"
    ) {
      throw new TypeError(
        "callback must be a function"
      );
    }

    const source =
      Object(this);

    const length =
      source.length >>> 0;

    const result =
      new Array(
        length
      );

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in source
      ) {
        result[i] =
          callback.call(
            thisArg,
            source[i],
            i,
            source
          );
      }
    }

    return result;
  };
```

---

## 7. Why New Array(length)?

This preserves the source array's length and holes.

If the source has a missing index, the result also has a missing index at that position.

---

## 8. Callback Arguments

Native-style callback:

```js
callback(
  value,
  index,
  array
);
```

Example:

```js
[10, 20]
  .customMap(
    (
      value,
      index
    ) =>
      value + index
  );
```

Result:

```js
[10, 21]
```

---

## 9. thisArg

```js
const context = {
  multiplier: 10,
};

const result =
  [1, 2, 3]
    .customMap(
      function (
        value
      ) {
        return (
          value *
          this.multiplier
        );
      },
      context
    );
```

Result:

```js
[10, 20, 30]
```

---

## 10. Original Array Is Not Mutated by map Itself

```js
const original = [
  1,
  2,
  3,
];

const result =
  original
    .customMap(
      (value) =>
        value * 2
    );
```

Now:

```text
original
[1, 2, 3]

result
[2, 4, 6]
```

---

## 11. Callback Can Still Mutate Original

Important nuance:

`map` itself creates a new output array.

But the callback can still mutate the source if you write it that way.

```js
array.map(
  (
    value,
    index,
    source
  ) => {
    source[index] =
      value * 2;

    return value;
  }
);
```

So:

> map is transformation-oriented, but callbacks are still arbitrary JavaScript.

---

## 12. Object Mapping

```js
const users = [
  {
    name:
      "Vikash",
  },
  {
    name:
      "Rahul",
  },
];

const names =
  users.customMap(
    (user) =>
      user.name
  );
```

Result:

```js
[
  "Vikash",
  "Rahul"
]
```

---

## 13. Returning Objects

Remember parentheses:

```js
const result =
  [1, 2]
    .customMap(
      (id) => ({
        id,
      })
    );
```

Without parentheses, arrow-function object syntax can be misread as a block.

---

## 14. Length Snapshot

Suppose:

```js
const arr = [
  1,
  2,
];

arr.customMap(
  (
    value,
    index
  ) => {
    if (
      index === 0
    ) {
      arr.push(3);
    }

    return value;
  }
);
```

The newly appended element should not become part of the original iteration.

Capturing initial length preserves this behavior.

---

## 15. Deleting During Iteration

If an index is removed before the loop reaches it, then:

```js
i in source
```

may become false and that index is skipped.

This is closer to native behavior.

---

## 16. Interview-Friendly Version

```js
Array.prototype.customMap =
  function (
    callback,
    thisArg
  ) {
    if (
      typeof callback !==
      "function"
    ) {
      throw new TypeError(
        "callback must be a function"
      );
    }

    const length =
      this.length;

    const result =
      new Array(
        length
      );

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in this
      ) {
        result[i] =
          callback.call(
            thisArg,
            this[i],
            i,
            this
          );
      }
    }

    return result;
  };
```

This is the version worth remembering for interviews.

---

## 17. Common Wrong Implementation

```js
Array.prototype.customMap =
  (callback) => {
    const result = [];

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      result.push(
        callback(
          this[i]
        )
      );
    }

    return result;
  };
```

Problems:

- arrow function breaks `this`
- does not pass index/array
- does not preserve sparse holes
- no callback validation
- no `thisArg`
- uses current length dynamically

---

## 18. map vs forEach

Use `map` when you want:

```text
input array
↓
transform every present item
↓
new array
```

Use `forEach` when you mainly need side effects.

---

## 19. Complexity

```text
Time:
O(n)

Output space:
O(n)
```

because a new result array is created.

---

## Interview Explanation

A strong answer:

> My custom `map` uses a regular prototype function so `this` refers to the source array. I capture the initial length, create a new result array of the same length, skip holes, and assign the callback result to the matching index. The callback receives `value`, `index`, and the original array, and I support `thisArg`.

---

## Key Takeaways

- `map` returns a new transformed array.
- It does not return `undefined` like `forEach`.
- Result should preserve sparse holes.
- Preallocate result with the source length.
- Callback gets value, index, array.
- Use a regular function for prototype implementation.
- Capture the initial length.
