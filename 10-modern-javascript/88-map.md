# Lesson 88 — Map

`Map` is a built-in JavaScript collection for storing key-value pairs.

It is similar to a plain object, but it has important differences.

The key idea:

> A `Map` can use values of any type as keys.

---

## 1. Create a Map

```js
const map =
  new Map();
```

Add values:

```js
map.set(
  "name",
  "Vikash"
);

map.set(
  "role",
  "Developer"
);
```

Read:

```js
map.get("name");
```

---

## 2. Keys Can Be Any Type

Objects convert most ordinary property keys to strings or symbols.

A `Map` can use:

- strings
- numbers
- objects
- functions
- booleans

Example:

```js
const user = {
  id: 1,
};

const map =
  new Map();

map.set(
  user,
  "profile data"
);

console.log(
  map.get(user)
);
```

Output:

```text
profile data
```

---

## 3. Object Identity Matters

```js
const map =
  new Map();

map.set(
  {
    id: 1,
  },
  "data"
);

console.log(
  map.get({
    id: 1,
  })
);
```

Output:

```text
undefined
```

Why?

The two object literals are different references.

```text
object A
≠
object B
```

Even if they contain the same values.

---

## 4. set()

```js
map.set(
  key,
  value
);
```

`set()` returns the Map itself.

That allows chaining:

```js
map
  .set("a", 1)
  .set("b", 2)
  .set("c", 3);
```

---

## 5. get()

```js
map.get(key);
```

If key does not exist:

```text
undefined
```

---

## 6. has()

```js
map.has("name");
```

Returns:

```text
true / false
```

Use `has()` when `undefined` might itself be a stored value.

---

## 7. delete()

```js
map.delete("name");
```

Returns whether an entry was removed.

---

## 8. clear()

```js
map.clear();
```

Removes all entries.

---

## 9. size

```js
map.size;
```

Unlike arrays:

```js
array.length
```

Maps use:

```js
map.size
```

---

## 10. Initialize with Entries

```js
const map =
  new Map([
    ["name", "Vikash"],
    ["role", "Developer"],
  ]);
```

Each entry is:

```js
[key, value]
```

---

## 11. Iterating a Map

Maps are iterable.

```js
for (
  const [key, value]
  of map
) {
  console.log(
    key,
    value
  );
}
```

Iteration follows insertion order.

---

## 12. keys()

```js
for (
  const key
  of map.keys()
) {
  console.log(key);
}
```

---

## 13. values()

```js
for (
  const value
  of map.values()
) {
  console.log(value);
}
```

---

## 14. entries()

```js
for (
  const entry
  of map.entries()
) {
  console.log(entry);
}
```

Each entry is:

```js
[key, value]
```

Also:

```js
map[Symbol.iterator]
===
map.entries
```

conceptually means default iteration yields entries.

---

## 15. forEach()

```js
map.forEach(
  (
    value,
    key
  ) => {
    console.log(
      key,
      value
    );
  }
);
```

Important parameter order:

```text
value, key
```

not:

```text
key, value
```

---

## 16. Map vs Object

Plain object:

```js
const user = {
  name: "Vikash",
};
```

Map:

```js
const user =
  new Map([
    ["name", "Vikash"],
  ]);
```

Main differences:

```text
Object
→ string/symbol property keys
→ prototype involvement
→ general-purpose records

Map
→ any key type
→ built-in size
→ explicit collection API
→ predictable iteration
```

---

## 17. When Object Is Better

Use a plain object for structured records:

```js
const user = {
  id: 1,
  name: "Vikash",
  role: "Developer",
};
```

This naturally represents an entity.

---

## 18. When Map Is Better

Use `Map` when:

- keys are dynamic
- keys are not just strings
- frequent add/remove/lookups matter
- you need insertion-order iteration
- data is conceptually a key-value collection

---

## 19. Convert Object to Map

```js
const user = {
  name: "Vikash",
  role: "Developer",
};

const map =
  new Map(
    Object.entries(user)
  );
```

---

## 20. Convert Map to Object

If keys are suitable property keys:

```js
const object =
  Object.fromEntries(
    map
  );
```

---

## 21. SameValueZero Key Comparison

Map key comparison uses behavior based on SameValueZero.

Important examples:

```js
const map =
  new Map();

map.set(
  NaN,
  "value"
);

console.log(
  map.get(NaN)
);
```

works.

Also:

```text
+0
and
-0
```

are treated as the same key.

---

## 22. Caching Example

```js
const cache =
  new Map();

function getUser(id) {
  if (
    cache.has(id)
  ) {
    return cache.get(id);
  }

  const user =
    loadUser(id);

  cache.set(
    id,
    user
  );

  return user;
}
```

Maps are common for caches, though memory management must be considered for long-lived caches.

---

## 23. Counting Example

```js
const words = [
  "js",
  "react",
  "js",
];

const counts =
  new Map();

for (const word of words) {
  counts.set(
    word,
    (
      counts.get(word) ??
      0
    ) + 1
  );
}
```

Result conceptually:

```text
js → 2
react → 1
```

---

## 24. React/Application Connection

Maps can be useful for:

- lookup tables
- normalized data
- caches
- metadata keyed by object identity

But React state should still be updated immutably.

Example:

```js
setItems((prev) => {
  const next =
    new Map(prev);

  next.set(
    id,
    value
  );

  return next;
});
```

Do not mutate the same Map instance and expect React change detection to behave like a new reference.

---

## Interview Questions

### Map vs Object?

Map is a dedicated key-value collection supporting keys of any type, built-in size, insertion-order iteration, and explicit collection methods.

### Can an object be a Map key?

Yes, and object identity determines the key.

### Is Map iterable?

Yes.

---

## Key Takeaways

- `Map` stores key-value pairs.
- Keys can be any JavaScript value.
- Object keys use reference identity.
- `set`, `get`, `has`, `delete`, and `clear` are core methods.
- `size` gives entry count.
- Maps preserve insertion order during iteration.
- Use plain objects for records and Maps for dynamic key-value collections.
