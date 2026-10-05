# Lesson 31 — Optional Chaining and Nullish Coalescing

Optional chaining (`?.`) and nullish coalescing (`??`) help safely work with values that may be missing.

They are especially useful with API responses, configuration and nested application data.

---

# Optional Chaining — ?.

Consider:

```js
const user = {
  name: "Vikash",
};
```

This can fail:

```js
console.log(
  user.address.city
);
```

because `user.address` is `undefined`.

JavaScript then tries to read `city` from `undefined`, causing a `TypeError`.

## Safe Access

```js
console.log(
  user.address?.city
);
```

Result:

```text
undefined
```

Instead of throwing, the optional chain stops when the value immediately before `?.` is `null` or `undefined`.

## Deep Optional Chaining

```js
const city =
  user?.profile?.address?.city;
```

Conceptually:

```text
user exists?
   ↓
profile exists?
   ↓
address exists?
   ↓
city
```

If a required step in the optional chain is nullish, the result becomes `undefined`.

## Optional Method Calls

```js
user.onLogin?.();
```

If `onLogin` is `null` or `undefined`, the call is skipped.

This is useful for optional callbacks.

React example:

```jsx
function Button({ onClick }) {
  return (
    <button
      onClick={() => onClick?.()}
    >
      Click
    </button>
  );
}
```

## Optional Element Access

```js
const users = null;

console.log(
  users?.[0]
); // undefined
```

It can also be used with dynamic property access:

```js
const key = "name";

console.log(
  user?.[key]
);
```

## Important: ?. Checks null and undefined

Optional chaining is concerned with **nullish values**:

```text
null
undefined
```

It does not stop for all falsy values such as:

```text
0
false
""
```

---

# Nullish Coalescing — ??

The nullish coalescing operator provides a fallback only when the left-hand value is:

```text
null
or
undefined
```

Example:

```js
const username = null;

const displayName =
  username ?? "Guest";

console.log(displayName);
// Guest
```

## Why Not Always Use ||?

Consider:

```js
const count = 0;

const value = count || 10;

console.log(value); // 10
```

Why?

Because `0` is falsy.

But sometimes `0` is a valid value.

Using `??`:

```js
const count = 0;

const value = count ?? 10;

console.log(value); // 0
```

This is often the desired behavior.

## || vs ??

`||` falls back when the left value is falsy.

Falsy values include:

```text
false
0
""
null
undefined
NaN
```

`??` falls back only for:

```text
null
undefined
```

Example:

```js
const name = "";

console.log(
  name || "Guest"
); // Guest

console.log(
  name ?? "Guest"
); // ""
```

If an empty string is valid application data, `??` preserves it.

---

# Combining ?. and ??

These operators work well together.

```js
const user = {
  profile: null,
};

const city =
  user.profile?.address?.city
  ?? "Unknown";

console.log(city);
// Unknown
```

Mental model:

```text
Try to read safely
        ↓
user.profile?.address?.city
        ↓
undefined
        ↓
?? fallback
        ↓
"Unknown"
```

## API Example

Suppose an API may or may not provide nested profile information.

```js
const response = {
  data: {
    user: {
      name: "Vikash",
    },
  },
};

const city =
  response.data?.user?.address?.city
  ?? "Not provided";
```

This is much cleaner than repeatedly writing manual checks.

## Important Mistake: Overusing Optional Chaining

Optional chaining should be used when missing data is expected or acceptable.

Suppose your application guarantees that `user` must exist.

Writing:

```js
user?.name
```

everywhere can hide an unexpected application-state bug.

Use optional chaining intentionally rather than automatically adding it to every property access.

## Grouping Can Break the Chain

Be careful with parentheses.

This is safely chained:

```js
user?.address?.city
```

But:

```js
(user?.address).city
```

can still throw if `user?.address` evaluates to `undefined`, because `.city` is outside the optional chain.

## React Connection

API-driven UI often uses optional chaining:

```jsx
<p>
  {user?.profile?.name ?? "Guest"}
</p>
```

Optional callback:

```js
onSuccess?.(data);
```

Default API value:

```js
const total =
  response?.data?.total ?? 0;
```

These patterns are common in production React and Next.js applications.

## Interview Perspective

**What does optional chaining do?**

Optional chaining safely accesses properties, elements or calls when the value before the optional chain may be `null` or `undefined`. If it is nullish, the chain returns `undefined` instead of continuing that access.

**Difference between || and ??**

`||` uses the fallback for any falsy left-hand value. `??` uses the fallback only when the left-hand value is `null` or `undefined`.

## Section 3 Connection

The important concepts from this section connect together:

```text
Arrays and Objects
       ↓
reference values
       ↓
mutation
       ↓
shallow copies
       ↓
nested references
       ↓
immutable updates
       ↓
React state behavior
```

Array transformations:

```text
map    → transform
filter → select
reduce → accumulate
find   → first match
some   → any match?
every  → all match?
```

Safe object access:

```text
?. → safely access nullish paths
?? → provide nullish fallback
```

## Key Takeaways

- `?.` safely handles `null` and `undefined` during chained access.
- Optional chaining works with properties, dynamic element access and calls.
- `??` provides a fallback only for nullish values.
- `||` and `??` are not interchangeable.
- `0`, `false` and empty strings are preserved by `??`.
- Optional chaining should be used intentionally.
- These operators are especially useful with API and UI data.
