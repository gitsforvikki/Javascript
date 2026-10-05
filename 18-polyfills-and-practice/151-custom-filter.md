# Lesson 151 — Implement Custom `filter`

`filter()` creates a new array containing only the elements whose callback returns a truthy value.

This interview question tests:

- predicates
- callbacks
- new-array creation
- sparse arrays
- truthy/falsy behavior
- `thisArg`

---

## 1. Native filter Behavior

```js
const numbers = [
  1,
  2,
  3,
  4,
];

const even =
  numbers.filter(
    (value) =>
      value % 2 === 0
  );
```

Result:

```js
[2, 4]
```

---

## 2. Predicate Function

The callback is often called a **predicate**.

A predicate answers:

```text
Should this value
be included?
```

Truthy:

```text
include
```

Falsy:

```text
exclude
```

---

## 3. Basic Implementation

```js
Array.prototype.customFilter =
  function (
    callback
  ) {
    const result = [];

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      if (
        callback(
          this[i],
          i,
          this
        )
      ) {
        result.push(
          this[i]
        );
      }
    }

    return result;
  };
```

Good start, but we still need sparse-array handling and validation.

---

## 4. Sparse Arrays

```js
const arr = [
  10,
  ,
  30,
];
```

Native `filter` skips the hole entirely.

Unlike `map`, the output does not preserve the hole because filter creates a compact array of accepted values.

Example:

```js
arr.filter(
  () => true
);
```

Result conceptually:

```js
[10, 30]
```

---

## 5. map vs filter Sparse Behavior

```text
map
→ same length
→ preserves holes

filter
→ compact result
→ skipped holes disappear
```

This is an excellent interview distinction.

---

## 6. Better Implementation

```js
Array.prototype.customFilter =
  function (
    callback,
    thisArg
  ) {
    if (
      this == null
    ) {
      throw new TypeError(
        "Cannot filter null or undefined"
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

    const result = [];

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in source
      ) {
        const value =
          source[i];

        const keep =
          callback.call(
            thisArg,
            value,
            i,
            source
          );

        if (keep) {
          result.push(
            value
          );
        }
      }
    }

    return result;
  };
```

---

## 7. Truthy/Falsy, Not Strict true

This is important.

`filter` does not require the callback to return exactly:

```js
true
```

Example:

```js
[1, 2, 3]
  .filter(
    (value) =>
      value
  );
```

All non-zero values are included because they are truthy.

---

## 8. Callback Arguments

```js
callback(
  value,
  index,
  array
);
```

Example:

```js
const result =
  [
    10,
    20,
    30,
  ].customFilter(
    (
      value,
      index
    ) =>
      index !== 1
  );
```

Result:

```js
[10, 30]
```

---

## 9. thisArg

```js
const context = {
  minimum: 20,
};

const result =
  [
    10,
    20,
    30,
  ].customFilter(
    function (
      value
    ) {
      return (
        value >=
        this.minimum
      );
    },
    context
  );
```

Result:

```js
[20, 30]
```

---

## 10. Object Filtering

```js
const users = [
  {
    name:
      "A",
    active:
      true,
  },
  {
    name:
      "B",
    active:
      false,
  },
];

const activeUsers =
  users.customFilter(
    (user) =>
      user.active
  );
```

---

## 11. filter Returns Original References

Important:

If the source contains objects, `filter` does not clone them.

```js
const user = {
  name:
    "Vikash",
};

const result =
  [user]
    .filter(
      () => true
    );
```

Then:

```js
result[0] === user
```

is:

```text
true
```

Filter creates a new array, not new objects.

---

## 12. Mutating Object After Filtering

```js
result[0].name =
  "Rahul";
```

also changes:

```js
user.name
```

because both arrays reference the same object.

This connects to value/reference lessons.

---

## 13. Length Snapshot

As with `map` and `forEach`, capture initial length.

If callback appends new items, they should not be included in the original pass.

---

## 14. Interview-Friendly Version

```js
Array.prototype.customFilter =
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

    const result = [];

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in this
      ) {
        const value =
          this[i];

        if (
          callback.call(
            thisArg,
            value,
            i,
            this
          )
        ) {
          result.push(
            value
          );
        }
      }
    }

    return result;
  };
```

---

## 15. Common Wrong Implementation

Wrong:

```js
Array.prototype.customFilter =
  (callback) => {
    const result = [];

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      if (
        callback(
          this[i]
        )
      ) {
        result.push(
          this[i]
        );
      }
    }

    return result;
  };
```

Problems:

- arrow function `this`
- sparse-array holes not handled
- no index/array arguments
- no `thisArg`
- no callback validation
- length changes can affect iteration

---

## 16. filter vs find

```text
filter
→ all matching elements
→ array

find
→ first matching element
→ value or undefined
```

---

## 17. filter vs map

```text
map
→ transforms values
→ output length generally matches source length

filter
→ selects values
→ output length may be smaller
```

---

## 18. Complexity

```text
Time:
O(n)

Output space:
O(k)
```

where `k` is the number of included elements.

Worst case:

```text
O(n)
```

space.

---

## Interview Explanation

A strong answer:

> My custom `filter` uses a regular prototype function, captures the original length, skips missing indexes, and calls the predicate with value, index, and source array. If the callback returns a truthy value, I push the original element into a new compact result array. I also support `thisArg`.

---

## Key Takeaways

- `filter` selects values; it does not transform them.
- Predicate uses truthy/falsy semantics.
- Result is a new compact array.
- Sparse holes are skipped and not preserved.
- Object values are not cloned.
- Use regular function for prototype method.
- Capture initial length.
