# Lesson 153 — Implement Custom `find`

`find()` returns the **first element** whose callback returns a truthy value.

This polyfill checks whether you understand:

- early termination
- callback arguments
- sparse arrays
- `thisArg`
- return behavior

---

## 1. Native find Behavior

```js
const numbers = [10, 20, 30, 40];

const result = numbers.find(
  (value) => value > 25
);

console.log(result);
// 30
```

Only the first match is returned.

---

## 2. No Match

```js
[1, 2, 3].find(
  (value) => value > 10
);
```

returns:

```text
undefined
```

---

## 3. Callback Arguments

The callback receives:

```js
callback(
  value,
  index,
  array
);
```

---

## 4. Basic Implementation

```js
Array.prototype.customFind =
  function (
    callback
  ) {
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
        return this[i];
      }
    }

    return undefined;
  };
```

This demonstrates the main idea:

> stop as soon as the predicate succeeds.

---

## 5. Why Early Return Matters

`find` should not scan the rest of the array after a match.

Example:

```js
const result =
  [5, 10, 15]
    .find(
      (value) => {
        console.log(value);
        return value >= 10;
      }
    );
```

Output:

```text
5
10
```

`15` is never checked.

---

## 6. Sparse Arrays

This is a subtle point.

Modern native `find()` visits indexes from `0` to `length - 1`, including empty slots, and passes `undefined` for holes.

Example:

```js
const arr = [
  10,
  ,
  30,
];

arr.find(
  (
    value,
    index
  ) => {
    console.log(
      index,
      value
    );

    return false;
  }
);
```

Conceptually:

```text
0 10
1 undefined
2 30
```

This differs from:

- `forEach`
- `map`
- `filter`

which skip holes.

---

## 7. Interview-Accurate Implementation

```js
Array.prototype.customFind =
  function (
    callback,
    thisArg
  ) {
    if (
      this == null
    ) {
      throw new TypeError(
        "Cannot find on null or undefined"
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

    for (
      let i = 0;
      i < length;
      i++
    ) {
      const value =
        source[i];

      if (
        callback.call(
          thisArg,
          value,
          i,
          source
        )
      ) {
        return value;
      }
    }

    return undefined;
  };
```

---

## 8. thisArg Support

```js
const context = {
  minimum: 20,
};

const result =
  [10, 20, 30]
    .customFind(
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

```text
20
```

---

## 9. Why Regular Function for Prototype Method?

Correct:

```js
Array.prototype.customFind =
  function (
    callback
  ) {
    // this → calling array
  };
```

Avoid an arrow function because it does not have dynamic `this`.

---

## 10. Length Snapshot

Capture length before looping.

Why?

If the callback pushes new elements during iteration, those newly appended items should not extend the original search range.

---

## 11. find vs filter

```text
find
→ first match
→ value or undefined

filter
→ all matches
→ new array
```

---

## 12. find vs findIndex

```text
find
→ matching value

findIndex
→ matching index
```

---

## 13. Object Example

```js
const users = [
  {
    id: 1,
    name: "A",
  },
  {
    id: 2,
    name: "B",
  },
];

const user =
  users.find(
    (item) =>
      item.id === 2
  );
```

Result:

```js
{
  id: 2,
  name: "B"
}
```

The object itself is returned, not a clone.

---

## 14. Common Wrong Implementation

```js
Array.prototype.customFind =
  (callback) => {
    let result;

    this.forEach(
      (value) => {
        if (
          callback(value)
        ) {
          result = value;
        }
      }
    );

    return result;
  };
```

Problems:

- arrow function breaks `this`
- does not stop at first match
- later matches can overwrite earlier one
- no index/array args
- no `thisArg`
- no callback validation

---

## 15. Complexity

Worst case:

```text
Time:
O(n)

Extra space:
O(1)
```

Best case:

```text
O(1)
```

if the first element matches.

---

## Interview Explanation

A strong answer:

> My custom `find` uses a regular prototype function, validates the callback, captures the initial length, checks indexes from left to right, and immediately returns the first value whose predicate is truthy. If nothing matches, it returns `undefined`. I also support the optional `thisArg`.

---

## Key Takeaways

- `find` returns the first matching value.
- It stops immediately after a match.
- No match returns `undefined`.
- Callback receives value, index, and source.
- `thisArg` is supported.
- Modern `find` visits empty slots as `undefined`.
