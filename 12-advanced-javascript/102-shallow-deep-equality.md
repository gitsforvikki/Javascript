# Lesson 102 — Shallow Equality vs Deep Equality

Two objects can be different references while containing similar data.

That leads to an important distinction:

- referential equality
- shallow equality
- deep equality

These concepts matter in:

- React
- memoization
- state management
- testing
- caching
- change detection

---

## 1. Referential Equality First

```js
const a = {
  name:
    "Vikash",
};

const b = {
  name:
    "Vikash",
};

console.log(
  a === b
);
```

Output:

```text
false
```

They are different objects.

---

## 2. What Is Shallow Equality?

Shallow equality compares only the immediate/top-level properties.

Conceptually:

```text
same top-level keys?
↓
same top-level values?
↓
for object-valued properties:
compare references only
```

It does not recursively inspect every nested property.

---

## 3. Basic Shallow Equal Example

```js
const a = {
  name:
    "Vikash",
  age: 25,
};

const b = {
  name:
    "Vikash",
  age: 25,
};
```

A shallow comparison can consider these equal because all top-level primitive values match.

Even though:

```js
a === b
// false
```

---

## 4. Nested Object Example

```js
const a = {
  profile: {
    city:
      "Bengaluru",
  },
};

const b = {
  profile: {
    city:
      "Bengaluru",
  },
};
```

A shallow comparison says:

```text
a.profile !== b.profile
```

So the objects are shallowly unequal.

A deep comparison may consider them equal because nested contents match.

---

## 5. Shared Nested Reference

```js
const profile = {
  city:
    "Bengaluru",
};

const a = {
  profile,
};

const b = {
  profile,
};
```

Now:

```js
a.profile ===
b.profile
// true
```

A shallow comparison can consider `a` and `b` equal if all top-level properties match by value/reference.

---

## 6. Shallow Equality Function

Simple educational implementation:

```js
function shallowEqual(
  a,
  b
) {
  if (
    Object.is(
      a,
      b
    )
  ) {
    return true;
  }

  if (
    typeof a !==
      "object" ||
    a === null ||
    typeof b !==
      "object" ||
    b === null
  ) {
    return false;
  }

  const keysA =
    Object.keys(a);

  const keysB =
    Object.keys(b);

  if (
    keysA.length !==
    keysB.length
  ) {
    return false;
  }

  for (
    const key
    of keysA
  ) {
    if (
      !Object.hasOwn(
        b,
        key
      ) ||
      !Object.is(
        a[key],
        b[key]
      )
    ) {
      return false;
    }
  }

  return true;
}
```

This is a learning implementation, not a universal equality library.

---

## 7. Why Object.is?

Using `Object.is()` handles edge cases such as:

```text
NaN
+0
-0
```

Many shallow comparison utilities use similar value semantics.

---

## 8. What Is Deep Equality?

Deep equality recursively compares nested structure and values.

Conceptually:

```text
compare root
↓
compare properties
↓
if nested object
recurse
↓
continue until leaves
```

---

## 9. Deep Equality Example

```js
const a = {
  profile: {
    city:
      "Bengaluru",
  },
};

const b = {
  profile: {
    city:
      "Bengaluru",
  },
};
```

Referential equality:

```text
false
```

Shallow equality:

```text
false
```

Deep equality:

```text
true
```

assuming a deep comparator that supports this structure.

---

## 10. Deep Equality Is Harder Than It Looks

A naïve recursive comparator may fail on:

- Date
- RegExp
- Map
- Set
- typed arrays
- Symbol keys
- prototypes
- property descriptors
- circular references
- functions
- class instances

So avoid assuming a tiny recursive function is a universal solution.

---

## 11. Circular Reference Problem

```js
const a = {};
a.self = a;

const b = {};
b.self = b;
```

A naïve recursive deep comparison may recurse forever.

Robust implementations need cycle tracking.

---

## 12. JSON.stringify Equality Trick

You may see:

```js
JSON.stringify(a) ===
JSON.stringify(b)
```

This is not a robust general deep-equality solution.

Problems include:

- property ordering assumptions
- unsupported values
- `undefined`
- functions
- Symbols
- circular references
- special built-ins

Use a proper comparator when deep equality is truly required.

---

## 13. Deep Equality Can Be Expensive

Shallow comparison cost is roughly proportional to immediate properties.

Deep comparison may traverse an entire object graph.

Conceptually:

```text
small shallow check
→ cheap

large nested deep check
→ potentially expensive
```

This matters in render-heavy applications.

---

## 14. Why Immutability Makes Shallow Equality Powerful

Suppose:

```js
const oldState = {
  user: {
    name:
      "Vikash",
  },

  settings: {
    theme:
      "dark",
  },
};
```

Immutable update:

```js
const newState = {
  ...oldState,

  user: {
    ...oldState.user,

    name:
      "Rahul",
  },
};
```

Then:

```text
oldState !== newState

oldState.user !==
newState.user

oldState.settings ===
newState.settings
```

Reference changes reveal exactly which branches changed.

---

## 15. Mutation Breaks Change Detection

Bad:

```js
oldState.user.name =
  "Rahul";

const newState =
  oldState;
```

Now:

```js
oldState ===
newState
// true
```

A reference-based comparison cannot detect the internal mutation.

This is why immutable updates pair so well with shallow equality.

---

## 16. React.memo

React can skip some child renders with:

```js
React.memo(
  Component
);
```

By default, React compares props using shallow-style per-prop comparison based on `Object.is`.

If a prop is a newly created object every render:

```jsx
<Child
  options={{
    page: 1,
  }}
/>
```

then that prop reference changes every render.

---

## 17. Stable Reference Example

```js
const options =
  useMemo(
    () => ({
      page: 1,
    }),
    []
  );
```

Then:

```jsx
<Child
  options={options}
/>
```

can preserve reference identity when appropriate.

Memoization should be justified, not automatic.

---

## 18. useEffect Dependencies

```js
useEffect(() => {
  // ...
}, [options]);
```

React compares dependency values with `Object.is`.

It does not deep-compare nested objects.

So a new object reference triggers the effect again.

---

## 19. Zustand / Redux Connection

State libraries often rely on selectors plus reference/shallow equality.

Why?

Deep comparison on every update can be expensive.

Immutable state updates make shallow checks meaningful.

---

## 20. Shallow Compare Arrays

```js
const a = [
  1,
  2,
];

const b = [
  1,
  2,
];
```

Reference:

```text
false
```

A shallow element-by-element comparator may say:

```text
true
```

because immediate primitive elements match.

---

## 21. Nested Arrays

```js
const a = [
  [1, 2],
];

const b = [
  [1, 2],
];
```

Shallow compare:

```text
false
```

because:

```js
a[0] !== b[0]
```

Deep compare may say true.

---

## 22. Functions and Equality

```js
const a = {
  onSave:
    () => {},
};

const b = {
  onSave:
    () => {},
};
```

A shallow comparator says false because the functions are different references.

A deep comparator generally cannot meaningfully decide that two arbitrary functions are behaviorally equivalent.

---

## 23. When to Use Referential Equality

Use reference equality when:

- identity itself matters
- immutable data model exists
- cache key is object identity
- React dependencies/state use references

It is the cheapest comparison.

---

## 24. When to Use Shallow Equality

Useful when:

- top-level values represent change boundaries
- immutable updates are used
- props/selectors are flat enough
- performance matters

Very common in frontend state systems.

---

## 25. When to Use Deep Equality

Use carefully when:

- structural equality is the true requirement
- objects are small/bounded
- comparisons are infrequent
- domain semantics justify it

Do not use deep equality as a default fix for unstable references.

---

## 26. Normalize Data Instead

If deeply nested comparisons become a constant problem, consider improving data shape.

Example:

```text
deep nested tree
↓
hard comparisons

normalized entities by ID
↓
clearer change boundaries
```

Data modeling can reduce the need for deep equality.

---

## 27. Interview Questions

### Referential vs shallow equality?

Referential equality asks whether two object references identify the same object. Shallow equality compares immediate properties without recursively comparing nested structures.

### Shallow vs deep equality?

Shallow checks one level; deep recursively compares nested content.

### Why is shallow equality useful with immutable data?

Changed branches receive new references, so shallow/reference checks can detect changes cheaply.

### Why not always deep-compare?

It can be expensive and difficult to define correctly for all JavaScript values.

---

## Key Takeaways

- `===` checks object identity.
- Shallow equality compares one level.
- Deep equality recursively compares structure.
- Nested reference identity matters in shallow checks.
- Deep equality has many edge cases.
- Immutability makes shallow equality useful.
- React dependency comparisons are not deep.
- Do not use deep equality to hide poor state/reference design.
