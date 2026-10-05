# Lesson 89 — Set

`Set` is a collection of unique values.

The core rule:

> A Set stores each value only once.

---

## 1. Create a Set

```js
const set =
  new Set();
```

Add values:

```js
set.add("React");
set.add("Next.js");
set.add("React");
```

The duplicate `"React"` is not added twice.

---

## 2. Initialize from Iterable

```js
const set =
  new Set([
    1,
    2,
    2,
    3,
  ]);
```

Contents:

```text
1
2
3
```

---

## 3. add()

```js
set.add(value);
```

Returns the Set itself.

So:

```js
set
  .add(1)
  .add(2)
  .add(3);
```

works.

---

## 4. has()

```js
set.has("React");
```

Returns:

```text
true / false
```

---

## 5. delete()

```js
set.delete("React");
```

Returns whether the value existed and was removed.

---

## 6. clear()

```js
set.clear();
```

Removes all values.

---

## 7. size

```js
set.size;
```

Returns number of unique values.

---

## 8. Removing Duplicates from Array

Classic use case:

```js
const numbers = [
  1,
  2,
  2,
  3,
  3,
];

const unique =
  [
    ...new Set(
      numbers
    ),
  ];
```

Result:

```js
[1, 2, 3]
```

---

## 9. Set Uses Value Equality Rules

For primitives:

```js
new Set([
  1,
  1,
  1,
]);
```

contains one `1`.

But objects use identity:

```js
const set =
  new Set([
    {
      id: 1,
    },
    {
      id: 1,
    },
  ]);
```

The Set contains two objects because they are different references.

---

## 10. Same Object Reference

```js
const user = {
  id: 1,
};

const set =
  new Set([
    user,
    user,
  ]);
```

Contains one entry.

Same reference.

---

## 11. Iteration

Sets are iterable:

```js
for (
  const value
  of set
) {
  console.log(value);
}
```

Iteration follows insertion order.

---

## 12. values()

```js
set.values();
```

returns an iterator.

---

## 13. keys()

For Set:

```js
set.keys();
```

behaves like `values()`.

Why?

Set has values, not separate key/value pairs.

This API exists partly for consistency with Map.

---

## 14. entries()

```js
set.entries();
```

produces:

```js
[value, value]
```

pairs.

Again, this is largely for compatibility with collection iteration patterns.

---

## 15. forEach()

```js
set.forEach(
  (value) => {
    console.log(value);
  }
);
```

Its callback receives the value twice in key/value-style positions for API consistency.

Usually you only need the first value.

---

## 16. Array vs Set

Array:

```text
ordered sequence
duplicates allowed
index access
```

Set:

```text
unique-value collection
no index access
membership-focused
```

---

## 17. Membership Checks

If your main operation is:

```text
Does this value exist?
```

Set expresses the intent clearly:

```js
selectedIds.has(id);
```

instead of:

```js
selectedIds.includes(id);
```

when a Set is the appropriate data structure.

---

## 18. Unique Tags Example

```js
const tags =
  new Set();

tags.add("react");
tags.add("javascript");
tags.add("react");

console.log(
  [...tags]
);
```

Result:

```js
[
  "react",
  "javascript"
]
```

---

## 19. Intersection

Modern environments may provide newer Set methods, but the conceptual/manual form is important:

```js
const a =
  new Set([
    1,
    2,
    3,
  ]);

const b =
  new Set([
    2,
    3,
    4,
  ]);

const intersection =
  new Set(
    [...a].filter(
      (value) =>
        b.has(value)
    )
  );
```

Result:

```text
2, 3
```

---

## 20. Union

```js
const union =
  new Set([
    ...a,
    ...b,
  ]);
```

Result:

```text
1, 2, 3, 4
```

---

## 21. Difference

```js
const difference =
  new Set(
    [...a].filter(
      (value) =>
        !b.has(value)
    )
  );
```

Result:

```text
1
```

---

## 22. Convert Set to Array

```js
const array =
  [...set];
```

or:

```js
const array =
  Array.from(set);
```

---

## 23. React Connection

Set can represent selections:

```js
const selected =
  new Set([
    1,
    2,
  ]);
```

In React state, create a new Set when updating:

```js
setSelected((prev) => {
  const next =
    new Set(prev);

  next.add(id);

  return next;
});
```

This preserves immutable update semantics at the state-reference level.

---

## Interview Questions

### What is Set used for?

A collection of unique values.

### Does Set remove duplicate objects by structure?

No. Object identity is used.

### Is Set iterable?

Yes.

### Set vs Array?

Use Set when uniqueness and membership are central; use Array for ordered indexed sequences.

---

## Key Takeaways

- `Set` stores unique values.
- Duplicate primitives are removed.
- Objects are unique by reference identity.
- Core methods are `add`, `has`, `delete`, and `clear`.
- `size` returns the number of entries.
- Sets are iterable.
- Sets are useful for deduplication and membership checks.
