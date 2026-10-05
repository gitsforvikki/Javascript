# Lesson 152 — Implement Custom `reduce`

`reduce()` is usually the most conceptual array polyfill interview question.

It tests whether you understand:

- accumulator
- current value
- initial value
- empty arrays
- sparse arrays
- callback arguments
- index selection

The hardest part is:

> What should happen when `initialValue` is not provided?

---

## 1. Native reduce Syntax

```js
array.reduce(
  callback,
  initialValue
);
```

Callback receives:

```js
callback(
  accumulator,
  currentValue,
  currentIndex,
  array
);
```

---

## 2. Basic Example

```js
const total =
  [1, 2, 3, 4]
    .reduce(
      (
        accumulator,
        value
      ) =>
        accumulator +
        value,
      0
    );
```

Flow:

```text
acc = 0
value = 1
→ 1

acc = 1
value = 2
→ 3

acc = 3
value = 3
→ 6

acc = 6
value = 4
→ 10
```

---

## 3. With initialValue

```js
[
  1,
  2,
  3,
].reduce(
  (
    acc,
    value
  ) =>
    acc + value,
  10
);
```

Initial accumulator:

```text
10
```

First current value:

```text
1
```

First index:

```text
0
```

---

## 4. Without initialValue

```js
[
  1,
  2,
  3,
].reduce(
  (
    acc,
    value
  ) =>
    acc + value
);
```

Now:

```text
initial accumulator
= first present element
= 1

iteration starts from
next present element
= index 1
```

This is the key rule.

---

## 5. Empty Array with initialValue

```js
[].reduce(
  (
    acc,
    value
  ) =>
    acc + value,
  0
);
```

Result:

```text
0
```

No callback execution is needed.

---

## 6. Empty Array without initialValue

```js
[].reduce(
  (
    acc,
    value
  ) =>
    acc + value
);
```

Throws:

```text
TypeError
```

Your custom implementation should reproduce this behavior.

---

## 7. First Naive Implementation

```js
Array.prototype.customReduce =
  function (
    callback,
    initialValue
  ) {
    let accumulator =
      initialValue;

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      accumulator =
        callback(
          accumulator,
          this[i],
          i,
          this
        );
    }

    return accumulator;
  };
```

This fails when no initial value is passed.

---

## 8. How to Detect Whether initialValue Was Passed

This is incorrect:

```js
if (
  initialValue ===
  undefined
)
```

Why?

The caller may intentionally pass:

```js
undefined
```

as the initial value.

These are different calls:

```js
arr.reduce(
  callback
);

arr.reduce(
  callback,
  undefined
);
```

We need to distinguish them.

---

## 9. Use arguments.length

Inside a regular function:

```js
arguments.length
```

tells us how many arguments were actually supplied.

For:

```js
customReduce(
  callback
)
```

length is:

```text
1
```

For:

```js
customReduce(
  callback,
  undefined
)
```

length is:

```text
2
```

This is a classic interview detail.

---

## 10. Sparse Arrays Make It Harder

Consider:

```js
const arr = [
  ,
  ,
  10,
  20,
];
```

If no initial value is supplied, accumulator should be:

```text
10
```

not:

```text
undefined from index 0
```

So we must search for the first **present** element.

---

## 11. Correct Initialization Logic

Pseudo-code:

```text
if initialValue exists:
    accumulator = initialValue
    start from index 0

else:
    find first existing array index

    if none exists:
        throw TypeError

    accumulator = that value
    continue after that index
```

---

## 12. Interview-Accurate Implementation

```js
Array.prototype.customReduce =
  function (
    callback,
    initialValue
  ) {
    if (
      this == null
    ) {
      throw new TypeError(
        "Cannot reduce null or undefined"
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

    let index = 0;
    let accumulator;

    const hasInitialValue =
      arguments.length >
      1;

    if (
      hasInitialValue
    ) {
      accumulator =
        initialValue;
    } else {
      while (
        index < length &&
        !(
          index in
          source
        )
      ) {
        index++;
      }

      if (
        index >= length
      ) {
        throw new TypeError(
          "Reduce of empty array with no initial value"
        );
      }

      accumulator =
        source[index];

      index++;
    }

    for (
      ;
      index < length;
      index++
    ) {
      if (
        index in source
      ) {
        accumulator =
          callback(
            accumulator,
            source[index],
            index,
            source
          );
      }
    }

    return accumulator;
  };
```

This is the key implementation to understand.

---

## 13. Test with initialValue

```js
const total =
  [
    1,
    2,
    3,
  ].customReduce(
    (
      acc,
      value
    ) =>
      acc + value,
    0
  );
```

Result:

```text
6
```

---

## 14. Test without initialValue

```js
const total =
  [
    1,
    2,
    3,
  ].customReduce(
    (
      acc,
      value
    ) =>
      acc + value
  );
```

Flow:

```text
acc = 1

start index = 1

1 + 2
→ 3

3 + 3
→ 6
```

---

## 15. Sparse Array Test

```js
const arr = [
  ,
  ,
  10,
  20,
];

const result =
  arr.customReduce(
    (
      acc,
      value
    ) =>
      acc + value
  );
```

Result:

```text
30
```

The holes are skipped.

---

## 16. Explicit undefined initialValue

```js
const result =
  [
    1,
    2,
  ].customReduce(
    (
      acc,
      value
    ) => {
      console.log(
        acc,
        value
      );

      return value;
    },
    undefined
  );
```

First accumulator really is:

```text
undefined
```

because an initial value was explicitly supplied.

This is why `arguments.length` matters.

---

## 17. Building an Object

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

const byId =
  users.reduce(
    (
      acc,
      user
    ) => {
      acc[user.id] =
        user;

      return acc;
    },
    {}
  );
```

Reduce can accumulate into any type.

---

## 18. Counting Values

```js
const words = [
  "js",
  "react",
  "js",
];

const counts =
  words.reduce(
    (
      acc,
      word
    ) => {
      acc[word] =
        (
          acc[word] ??
          0
        ) + 1;

      return acc;
    },
    {}
  );
```

Result:

```js
{
  js: 2,
  react: 1
}
```

---

## 19. Flatten Example

```js
const nested = [
  [1, 2],
  [3, 4],
];

const flat =
  nested.reduce(
    (
      acc,
      item
    ) =>
      acc.concat(
        item
      ),
    []
  );
```

---

## 20. reduce Is Powerful, But Do Not Overuse It

This:

```js
array.reduce(...)
```

can implement:

- map
- filter
- grouping
- counting

But that does not mean it is always the clearest choice.

Use:

- `map` for transformation
- `filter` for selection
- `reduce` for accumulation

when those names communicate intent better.

---

## 21. No thisArg in reduce

Important difference:

Native `reduce` does **not** take a `thisArg` parameter like:

- `forEach`
- `map`
- `filter`

So our custom reduce should not invent one.

---

## 22. Callback Return Is Critical

Wrong:

```js
[1, 2, 3]
  .reduce(
    (
      acc,
      value
    ) => {
      acc + value;
    },
    0
  );
```

The callback returns:

```text
undefined
```

So the accumulator becomes undefined.

Correct:

```js
return (
  acc + value
);
```

---

## 23. Mutation Inside reduce

This pattern mutates the accumulator object:

```js
array.reduce(
  (
    acc,
    item
  ) => {
    acc[item.id] =
      item;

    return acc;
  },
  {}
);
```

That can be perfectly reasonable because the accumulator is local to the reduction.

Do not confuse all local mutation with unsafe shared-state mutation.

---

## 24. Complexity

For array length `n`:

```text
Time:
O(n)

Extra space:
depends on accumulator
```

A numeric sum uses:

```text
O(1)
```

extra space.

Building an object/array may use:

```text
O(n)
```

---

## 25. Common Wrong Implementation

```js
Array.prototype.customReduce =
  function (
    callback,
    initialValue
  ) {
    let acc =
      initialValue;

    for (
      let i = 0;
      i < this.length;
      i++
    ) {
      acc =
        callback(
          acc,
          this[i]
        );
    }

    return acc;
  };
```

Problems:

1. cannot distinguish missing initial value from explicit `undefined`
2. does not use first present element when initial value is missing
3. fails empty-array rule
4. does not skip sparse holes
5. does not pass index and array
6. no callback validation

---

## Interview Explanation

A strong answer:

> The tricky part of custom `reduce` is initialization. If an initial value is supplied, I use it and start from index 0. If not, I search for the first present array element, use that as the accumulator, and continue from the next index. If no present element exists, I throw a TypeError. I skip sparse holes and call the reducer with accumulator, value, index, and source array.

---

## Section 18 Mental Model So Far

```text
forEach
→ perform side effects
→ returns undefined

map
→ transform
→ same positional structure

filter
→ select
→ compact result

reduce
→ accumulate
→ one final result
```

---

## Key Takeaways

- `reduce` accumulates values into one result.
- Callback receives accumulator, value, index, and array.
- Initial-value behavior is the hardest part.
- Missing initial value is different from explicit `undefined`.
- Use `arguments.length` to detect whether it was supplied.
- Without initial value, first present element becomes accumulator.
- Empty/no-present-element array without initial value throws.
- Sparse holes are skipped.
- Native reduce has no `thisArg`.
