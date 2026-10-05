# Lesson 149 — Implement Custom `forEach`

Implementing `forEach` is a common JavaScript interview exercise because it checks your understanding of:

- array iteration
- callbacks
- `this`
- prototype methods
- sparse arrays
- callback arguments
- return behavior

The goal is not just to make something that loops.

The goal is to understand how the native method behaves.

---

## 1. Native forEach Behavior

Example:

```js
const numbers = [
  10,
  20,
  30,
];

numbers.forEach(
  (
    value,
    index,
    array
  ) => {
    console.log(
      value,
      index,
      array
    );
  }
);
```

The callback receives:

```text
1. current value
2. current index
3. original array
```

---

## 2. forEach Return Value

`forEach()` returns:

```js
undefined
```

Example:

```js
const result =
  [1, 2, 3]
    .forEach(
      (value) =>
        value * 2
    );

console.log(
  result
);
```

Output:

```text
undefined
```

This is a major difference from `map()`.

---

## 3. First Simple Implementation

A beginner implementation:

```js
Array.prototype.customForEach =
  function (
    callback
  ) {
    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      callback(
        this[i],
        i,
        this
      );
    }
  };
```

Usage:

```js
const numbers = [
  10,
  20,
  30,
];

numbers.customForEach(
  (value) => {
    console.log(value);
  }
);
```

This works for ordinary dense arrays.

But it is not yet accurate enough for an interview-quality polyfill.

---

## 4. Why We Use a Regular Function

Correct:

```js
Array.prototype.customForEach =
  function (
    callback
  ) {
    // ...
  };
```

Avoid:

```js
Array.prototype.customForEach =
  (
    callback
  ) => {
    // this is wrong
  };
```

Why?

Arrow functions do not have their own dynamic `this`.

For a prototype method:

```js
numbers.customForEach(...)
```

we need:

```js
this
```

to refer to `numbers`.

---

## 5. Validate the Callback

Native-style methods expect a callable callback.

So add:

```js
if (
  typeof callback !==
  "function"
) {
  throw new TypeError(
    "callback must be a function"
  );
}
```

---

## 6. thisArg Support

Native `forEach` accepts an optional second argument.

Example:

```js
const context = {
  prefix: "Value:",
};

[1, 2].forEach(
  function (
    value
  ) {
    console.log(
      this.prefix,
      value
    );
  },
  context
);
```

So our implementation should support:

```js
callback.call(
  thisArg,
  value,
  index,
  array
);
```

---

## 7. Sparse Arrays

This is an important interview edge case.

Example:

```js
const array = [
  10,
  ,
  30,
];
```

Index 1 is a hole.

Native `forEach` skips missing indexes.

So this implementation is slightly wrong:

```js
for (
  let i = 0;
  i < this.length;
  i++
) {
  callback(
    this[i],
    i,
    this
  );
}
```

because it executes for the hole too.

---

## 8. Detect Existing Indexes

Use:

```js
if (
  i in this
) {
  // callback
}
```

This checks whether that property exists on the array/prototype chain.

For a learning polyfill, it closely represents the native iteration behavior over present indexes.

---

## 9. Better Implementation

```js
Array.prototype.customForEach =
  function (
    callback,
    thisArg
  ) {
    if (
      this == null
    ) {
      throw new TypeError(
        "Cannot iterate over null or undefined"
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

    const array =
      Object(this);

    const length =
      array.length >>> 0;

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in array
      ) {
        callback.call(
          thisArg,
          array[i],
          i,
          array
        );
      }
    }
  };
```

This is much closer to native behavior.

---

## 10. Why Object(this)?

Conceptually:

```js
const array =
  Object(this);
```

normalizes the receiver into an object.

Array methods can sometimes be used on array-like objects too.

Example:

```js
const arrayLike = {
  0: "A",
  1: "B",
  length: 2,
};

Array.prototype
  .forEach
  .call(
    arrayLike,
    console.log
  );
```

The native method is more generic than "arrays only."

---

## 11. Why Capture length First?

Native iterative methods typically determine the relevant length before iteration proceeds.

Example:

```js
const arr = [
  1,
  2,
];

arr.forEach(
  (
    value,
    index
  ) => {
    if (
      index === 0
    ) {
      arr.push(3);
    }

    console.log(
      value
    );
  }
);
```

The newly pushed element is not visited by that original iteration.

So capturing:

```js
const length =
  array.length;
```

is important.

---

## 12. Interview-Friendly Version

For most interviews, this is enough:

```js
Array.prototype.customForEach =
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

    for (
      let i = 0;
      i < length;
      i++
    ) {
      if (
        i in this
      ) {
        callback.call(
          thisArg,
          this[i],
          i,
          this
        );
      }
    }
  };
```

This version is easier to explain in an interview.

---

## 13. Test It

```js
const numbers = [
  10,
  ,
  30,
];

numbers.customForEach(
  (
    value,
    index
  ) => {
    console.log(
      index,
      value
    );
  }
);
```

Expected:

```text
0 10
2 30
```

Index 1 is skipped.

---

## 14. Test thisArg

```js
const context = {
  multiplier: 10,
};

[1, 2, 3]
  .customForEach(
    function (
      value
    ) {
      console.log(
        value *
          this.multiplier
      );
    },
    context
  );
```

Output:

```text
10
20
30
```

---

## 15. forEach Cannot Be Broken Normally

You cannot use:

```js
break
```

inside the callback to stop `forEach`.

If you need early termination, alternatives include:

- `for`
- `for...of`
- `some`
- `every`
- `find`

---

## 16. Returning from Callback Does Not Stop Iteration

```js
[1, 2, 3]
  .forEach(
    (value) => {
      if (
        value === 2
      ) {
        return;
      }

      console.log(
        value
      );
    }
  );
```

The `return` exits only the callback invocation.

The outer iteration continues.

---

## 17. Common Wrong Implementation

Wrong:

```js
Array.prototype.customForEach =
  (callback) => {
    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      callback(
        this[i]
      );
    }
  };
```

Problems:

1. arrow function has wrong `this`
2. callback gets only value
3. no sparse-array handling
4. no callback validation
5. no `thisArg`
6. no length snapshot

---

## 18. Complexity

For an array of length `n`:

```text
Time:
O(n)

Extra space:
O(1)
```

excluding work done inside the callback.

---

## Interview Explanation

A strong interview answer:

> I would implement custom `forEach` as a regular prototype function so `this` refers to the calling array. I capture the initial length, loop through indexes, skip holes, and invoke the callback with `value`, `index`, and the original array. I also support an optional `thisArg`. The method returns `undefined`.

---

## Key Takeaways

- `forEach` is for side effects, not transformation.
- It returns `undefined`.
- Callback receives value, index, and array.
- Native behavior skips sparse-array holes.
- Prototype implementation should use a regular function.
- `thisArg` can be supported with `call`.
- Capture initial length before iteration.
