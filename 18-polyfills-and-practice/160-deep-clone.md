# Lesson 160 — Implement Deep Clone

Deep cloning means creating a new object graph where nested mutable objects are also copied.

This interview question tests:

- recursion
- object identity
- arrays
- built-in objects
- circular references
- property descriptors
- prototypes
- limitations of handwritten cloning

The core distinction is:

```text
shallow copy
→ new root only

deep clone
→ new nested object graph
```

---

## 1. Shallow Copy Problem

```js
const original = {
  profile: {
    city:
      "Bengaluru",
  },
};

const copy = {
  ...original,
};

copy.profile.city =
  "Pune";
```

Now:

```js
original.profile.city
```

is also:

```text
Pune
```

because nested `profile` is shared.

---

## 2. Basic Recursive Clone

```js
function deepClone(
  value
) {
  if (
    value === null ||
    typeof value !==
      "object"
  ) {
    return value;
  }

  if (
    Array.isArray(
      value
    )
  ) {
    return value.map(
      deepClone
    );
  }

  const result = {};

  for (
    const key
    of Object.keys(
      value
    )
  ) {
    result[key] =
      deepClone(
        value[key]
      );
  }

  return result;
}
```

This handles plain objects and arrays.

But it is not complete.

---

## 3. Circular Reference Problem

```js
const user = {
  name: "Vikash",
};

user.self =
  user;
```

Naive recursion loops forever.

We need to remember already-cloned objects.

---

## 4. WeakMap for Cycle Tracking

```js
function deepClone(
  value,
  seen =
    new WeakMap()
) {
  if (
    value === null ||
    typeof value !==
      "object"
  ) {
    return value;
  }

  if (
    seen.has(value)
  ) {
    return seen.get(
      value
    );
  }

  const clone =
    Array.isArray(
      value
    )
      ? []
      : {};

  seen.set(
    value,
    clone
  );

  for (
    const key
    of Reflect.ownKeys(
      value
    )
  ) {
    clone[key] =
      deepClone(
        value[key],
        seen
      );
  }

  return clone;
}
```

Now cycles can be preserved.

---

## 5. Why WeakMap?

We need:

```text
original object
→ cloned object
```

mapping.

WeakMap is ideal because keys are objects and it does not keep them alive unnecessarily after cloning finishes.

---

## 6. Date

```js
if (
  value instanceof
  Date
) {
  return new Date(
    value.getTime()
  );
}
```

Without this, Date becomes a plain object in many naive implementations.

---

## 7. RegExp

```js
if (
  value instanceof
  RegExp
) {
  return new RegExp(
    value.source,
    value.flags
  );
}
```

Optionally preserve:

```js
clone.lastIndex =
  value.lastIndex;
```

---

## 8. Map

```js
if (
  value instanceof
  Map
) {
  const clone =
    new Map();

  seen.set(
    value,
    clone
  );

  for (
    const [
      key,
      item,
    ]
    of value
  ) {
    clone.set(
      deepClone(
        key,
        seen
      ),
      deepClone(
        item,
        seen
      )
    );
  }

  return clone;
}
```

---

## 9. Set

```js
if (
  value instanceof
  Set
) {
  const clone =
    new Set();

  seen.set(
    value,
    clone
  );

  for (
    const item
    of value
  ) {
    clone.add(
      deepClone(
        item,
        seen
      )
    );
  }

  return clone;
}
```

---

## 10. Preserve Prototype

Instead of:

```js
const clone = {};
```

use:

```js
const clone =
  Object.create(
    Object.getPrototypeOf(
      value
    )
  );
```

This preserves the original prototype chain.

---

## 11. Property Descriptors

Simple assignment:

```js
clone[key] =
  value[key];
```

does not preserve:

- writable
- enumerable
- configurable
- getters
- setters

For a more advanced clone, inspect descriptors.

---

## 12. Descriptor-Based Copy

```js
const descriptors =
  Object.getOwnPropertyDescriptors(
    value
  );
```

For each data descriptor, deep-clone:

```js
descriptor.value
```

Then define properties with:

```js
Object.defineProperties(
  clone,
  descriptors
);
```

---

## 13. More Advanced Implementation

```js
function deepClone(
  value,
  seen =
    new WeakMap()
) {
  if (
    value === null ||
    typeof value !==
      "object"
  ) {
    return value;
  }

  if (
    seen.has(value)
  ) {
    return seen.get(
      value
    );
  }

  if (
    value instanceof
    Date
  ) {
    return new Date(
      value.getTime()
    );
  }

  if (
    value instanceof
    RegExp
  ) {
    const clone =
      new RegExp(
        value.source,
        value.flags
      );

    clone.lastIndex =
      value.lastIndex;

    return clone;
  }

  if (
    value instanceof
    Map
  ) {
    const clone =
      new Map();

    seen.set(
      value,
      clone
    );

    for (
      const [
        key,
        item,
      ]
      of value
    ) {
      clone.set(
        deepClone(
          key,
          seen
        ),
        deepClone(
          item,
          seen
        )
      );
    }

    return clone;
  }

  if (
    value instanceof
    Set
  ) {
    const clone =
      new Set();

    seen.set(
      value,
      clone
    );

    for (
      const item
      of value
    ) {
      clone.add(
        deepClone(
          item,
          seen
        )
      );
    }

    return clone;
  }

  const clone =
    Array.isArray(
      value
    )
      ? []
      : Object.create(
          Object.getPrototypeOf(
            value
          )
        );

  seen.set(
    value,
    clone
  );

  const descriptors =
    Object.getOwnPropertyDescriptors(
      value
    );

  for (
    const key
    of Reflect.ownKeys(
      descriptors
    )
  ) {
    const descriptor =
      descriptors[key];

    if (
      "value" in
      descriptor
    ) {
      descriptor.value =
        deepClone(
          descriptor.value,
          seen
        );
    }
  }

  Object.defineProperties(
    clone,
    descriptors
  );

  return clone;
}
```

This is a much stronger interview implementation.

---

## 14. Functions

Functions are tricky.

Usually:

```text
function object
→ preserve same reference
```

Trying to "clone" function code using `toString` + `eval` is unsafe and incorrect.

So:

```js
typeof value ===
  "function"
```

is often treated as a non-cloneable/same-reference value.

---

## 15. Symbol-Keyed Properties

Using:

```js
Reflect.ownKeys(
  value
)
```

or property descriptors includes Symbol keys.

`Object.keys()` would miss them.

---

## 16. Non-Enumerable Properties

```js
Object.keys()
```

returns only enumerable string keys.

Descriptor-based cloning preserves non-enumerable own properties too.

---

## 17. Accessor Properties

Getter/setter descriptors contain:

```js
get
set
```

not `value`.

A descriptor-based clone can preserve these function references without invoking the getter.

This is safer than reading every property value directly.

---

## 18. Typed Arrays and ArrayBuffer

A more complete deep clone may need special handling for:

- ArrayBuffer
- Uint8Array
- Float32Array
- DataView

These require binary-aware copying.

This is one reason handwritten deep clone utilities become complex quickly.

---

## 19. WeakMap and WeakSet

Weak collections are not enumerable.

You cannot generally deep-clone their contents because JavaScript does not expose iteration over them.

A handwritten generic deep clone cannot fully reproduce them.

---

## 20. DOM Nodes

DOM elements need DOM-specific cloning:

```js
element.cloneNode(
  true
);
```

A generic object deep-clone function should not pretend to clone DOM semantics.

---

## 21. Error Objects

Errors contain special internal/runtime behavior.

A robust clone might recreate:

- name
- message
- cause
- custom properties

but stack behavior is runtime-specific.

---

## 22. JSON Clone Comparison

Bad general solution:

```js
JSON.parse(
  JSON.stringify(
    value
  )
);
```

Problems:

- circular refs fail
- Date becomes string
- Map/Set lost
- undefined lost
- functions lost
- Symbols lost
- BigInt fails
- prototype/descriptors lost

---

## 23. structuredClone()

Modern JavaScript provides:

```js
structuredClone(
  value
);
```

It supports many built-in types and circular references.

For real application code, prefer it when its semantics match your needs.

---

## 24. But structuredClone Is Not Exact Object Duplication

It does not preserve every custom prototype/property-descriptor behavior exactly as a handwritten descriptor-aware clone might attempt.

It follows the structured clone algorithm.

So:

```text
structuredClone
≠
"clone absolutely everything"
```

---

## 25. Deep Clone vs Immutability

Do not deep-clone entire state just to update one nested field.

Better:

```js
const next = {
  ...state,
  user: {
    ...state.user,
    name:
      "Rahul",
  },
};
```

This preserves structural sharing.

---

## 26. Complexity

For `n` reachable properties/elements:

```text
Time:
O(n)

Space:
O(n)
```

plus cycle-tracking storage.

---

## 27. Common Wrong Implementation

```js
function deepClone(
  object
) {
  return JSON.parse(
    JSON.stringify(
      object
    )
  );
}
```

This is not a general deep clone.

---

## Interview Explanation

A strong answer:

> I recursively clone object values, use a WeakMap to preserve cycles and shared references, special-case built-ins such as Date, RegExp, Map, and Set, and preserve prototypes/descriptors for ordinary objects. I would also explain that a handwritten generic deep clone has unavoidable limitations, and in real code I would prefer `structuredClone()` when its semantics fit.

---

## Section 18 Mental Model So Far

```text
debounce
→ wait for silence

throttle
→ limit frequency

memoize
→ cache repeated results

deep clone
→ copy object graph
→ preserve cycles carefully
```

---

## Key Takeaways

- Deep clone is recursive graph copying.
- WeakMap prevents infinite recursion on cycles.
- Built-ins need special handling.
- Preserve prototypes/descriptors if interview depth requires it.
- Symbol and non-enumerable keys need special APIs.
- Functions and weak collections are not meaningfully generic-cloneable.
- JSON cloning is not a general solution.
- Prefer `structuredClone()` in real code when appropriate.
