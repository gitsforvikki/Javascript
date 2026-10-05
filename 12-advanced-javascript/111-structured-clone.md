# Lesson 111 — `structuredClone()` and Deep Cloning

Copying JavaScript data is more subtle than it first appears.

You need to distinguish:

- reference copying
- shallow copying
- deep cloning

The modern built-in:

```js
structuredClone(
  value
);
```

can deeply clone many supported JavaScript values.

But:

> `structuredClone()` does not clone every possible JavaScript value, and deep cloning is not always the correct solution.

---

## 1. Reference Copy

```js
const original = {
  name:
    "Vikash",
};

const copy =
  original;
```

Now:

```text
original ─┐
          ├──► same object
copy ─────┘
```

There is no clone.

---

## 2. Shallow Copy

```js
const original = {
  name:
    "Vikash",
  profile: {
    city:
      "Bengaluru",
  },
};

const copy = {
  ...original,
};
```

Top-level object is new.

But:

```js
copy.profile ===
original.profile
```

is:

```text
true
```

Nested object is shared.

---

## 3. Shallow Copy Diagram

```text
original ──► object A
              │
              └── profile ──► object X

copy ─────► object B
              │
              └── profile ──► object X
```

Only the root was copied.

---

## 4. Deep Clone

A deep clone recursively creates independent copies of nested supported data.

Conceptually:

```text
original
↓
object A
└── nested X

clone
↓
object B
└── nested Y
```

Now:

```text
X !== Y
```

---

## 5. structuredClone()

```js
const clone =
  structuredClone(
    original
  );
```

Now:

```js
clone === original
// false
```

and:

```js
clone.profile ===
original.profile
// false
```

for ordinary nested objects.

---

## 6. Basic Example

```js
const user = {
  name:
    "Vikash",

  skills: [
    "JavaScript",
    "React",
  ],

  address: {
    city:
      "Bengaluru",
  },
};

const cloned =
  structuredClone(
    user
  );

cloned.address.city =
  "Pune";
```

The original address remains unchanged.

---

## 7. Circular References

One major advantage over JSON cloning:

```js
const object = {
  name:
    "Vikash",
};

object.self =
  object;
```

This fails with JSON serialization:

```js
JSON.stringify(
  object
);
```

But:

```js
const clone =
  structuredClone(
    object
  );
```

can preserve the circular structure.

Conceptually:

```js
clone.self === clone
// true
```

---

## 8. Date

```js
const original = {
  createdAt:
    new Date(),
};

const clone =
  structuredClone(
    original
  );
```

`clone.createdAt` remains a `Date` object.

This is much better than JSON cloning, which converts Date to a string.

---

## 9. Map

```js
const original =
  new Map([
    ["name", "Vikash"],
  ]);

const clone =
  structuredClone(
    original
  );
```

The clone is also a Map with cloned entries.

---

## 10. Set

```js
const original =
  new Set([
    1,
    2,
    3,
  ]);

const clone =
  structuredClone(
    original
  );
```

The clone remains a Set.

---

## 11. ArrayBuffer and Typed Data

Structured cloning supports many binary data structures such as:

- ArrayBuffer
- typed arrays
- DataView

This is one reason the structured clone algorithm is used in browser messaging APIs.

---

## 12. RegExp

Supported environments can clone:

```js
const regex =
  /react/gi;

const clone =
  structuredClone(
    regex
  );
```

The clone remains a RegExp.

Some internal state details such as `lastIndex` should not be assumed to be preserved identically in every cloning scenario; check the platform contract when that detail matters.

---

## 13. Functions Cannot Be Cloned

This fails:

```js
structuredClone({
  handler() {},
});
```

Functions are not supported by the structured clone algorithm.

A `DataCloneError`-style exception is typically thrown.

---

## 14. DOM Nodes Are Not General Clone Targets

Do not use:

```js
structuredClone(
  domElement
);
```

to clone DOM.

DOM nodes have their own API:

```js
element.cloneNode(
  true
);
```

Different data domains use different cloning mechanisms.

---

## 15. Symbols

Symbol values generally cannot be cloned by `structuredClone()`.

Symbol-keyed object properties are also not a good reason to expect a perfect structural copy of every object semantic.

Always verify cloneability of advanced values.

---

## 16. Class Instances

This is subtle.

```js
class User {
  constructor(
    name
  ) {
    this.name =
      name;
  }

  greet() {
    return `Hi ${this.name}`;
  }
}
```

Do not assume:

```js
structuredClone(
  new User(
    "Vikash"
  )
);
```

will preserve custom class prototype/method semantics as an instance of `User`.

Structured cloning is for cloneable data, not arbitrary application object behavior.

---

## 17. Property Descriptors Are Not Preserved Like an Exact Object Replica

Suppose an object has:

- getters
- setters
- non-enumerable properties
- custom descriptors

A structured clone should not be thought of as:

> exact object internals duplicated byte-for-byte.

It clones data according to the structured clone algorithm, not arbitrary descriptor/prototype semantics.

---

## 18. JSON Clone Technique

Common old pattern:

```js
const clone =
  JSON.parse(
    JSON.stringify(
      value
    )
  );
```

This can work for very simple JSON-compatible data.

But it is not a general deep clone.

---

## 19. JSON Clone Loses Values

Examples of problems:

- `undefined`
- functions
- Symbols
- circular references
- Date type
- Map
- Set
- Infinity / NaN behavior
- BigInt serialization

So avoid calling it a universal deep clone.

---

## 20. structuredClone vs JSON Clone

```text
structuredClone
→ supports many built-ins
→ supports cycles
→ preserves richer data types
→ rejects unsupported values

JSON clone
→ only JSON-compatible data
→ no cycles
→ loses types/information
```

---

## 21. Transferable Objects

Structured cloning can sometimes **transfer** certain objects instead of copying their underlying resource.

Example:

```js
const buffer =
  new ArrayBuffer(
    1024
  );

const clone =
  structuredClone(
    buffer,
    {
      transfer: [
        buffer,
      ],
    }
  );
```

The original buffer becomes detached.

This avoids copying the underlying bytes.

---

## 22. Clone vs Transfer

```text
clone
→ duplicate supported data

transfer
→ move ownership/resource
→ original becomes unusable/detached
```

Transfer is useful for large binary data between workers/contexts.

---

## 23. postMessage Connection

Browsers use the structured clone algorithm for APIs such as:

- `postMessage()`
- Web Workers
- MessageChannel

That is why many of the same data types can be passed between execution contexts.

---

## 24. Deep Clone Is Not Immutability

This is very important.

```js
const next =
  structuredClone(
    state
  );
```

creates a deep copy.

But immutability is a **state-update strategy**, not merely the act of cloning deeply.

Usually, you should copy only changed branches.

---

## 25. Why Deep-Cloning React State Is Often Wrong

Suppose:

```js
const state = {
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

If only `user.name` changes, this:

```js
structuredClone(
  state
);
```

creates new references for everything.

Then:

```text
old settings !==
new settings
```

even though settings did not change.

This destroys useful structural sharing.

---

## 26. Better Immutable Update

```js
const nextState = {
  ...state,

  user: {
    ...state.user,

    name:
      "Rahul",
  },
};
```

Now:

```js
nextState
  .settings ===
state.settings
```

is true.

Unchanged data keeps stable references.

---

## 27. When structuredClone Is Useful

Good cases:

- independent editable copy
- snapshotting supported data
- cloning messages/data graphs
- test fixtures
- worker communication preparation
- copying complex built-in collections

---

## 28. When structuredClone Is Not Appropriate

Avoid using it blindly for:

- every React state update
- objects containing functions
- custom class instances requiring prototype behavior
- DOM nodes
- performance-critical huge graphs without measuring

---

## 29. Performance

Deep cloning requires traversing the data graph.

For large objects, this costs:

- CPU
- memory
- allocation
- garbage collection pressure

Always ask:

> Do I actually need a completely independent deep copy?

Often the answer is no.

---

## 30. Clone Depth Mental Model

Reference copy:

```text
new variable
same object
```

Shallow copy:

```text
new root
shared nested objects
```

Deep clone:

```text
new root
new nested supported objects
```

---

## 31. structuredClone Error Handling

Because unsupported values can cause cloning to fail:

```js
try {
  const clone =
    structuredClone(
      value
    );
} catch (error) {
  console.error(
    "Value cannot be cloned",
    error
  );
}
```

Do not assume arbitrary runtime objects are cloneable.

---

## 32. Interview Questions

### What does structuredClone do?

It creates a deep clone of values supported by the structured clone algorithm.

### Is object spread a deep clone?

No. It is shallow.

### Why is JSON.parse(JSON.stringify()) not a general deep clone?

It loses unsupported/non-JSON types and cannot handle circular references.

### Can structuredClone clone functions?

No.

### Does structuredClone preserve custom class behavior automatically?

Do not rely on it for custom prototype/method semantics.

### Why not deep-clone all React state?

It destroys structural sharing, creates unnecessary allocations, and changes references for unchanged branches.

---

## Section 12 Complete Mental Model

```text
mutation vs immutability
↓
reference identity
↓
shallow vs deep equality
↓
debounce / throttle
↓
memoization
↓
currying
↓
partial application
↓
function composition
↓
recursion
↓
Proxy + Reflect
↓
structuredClone
```

Together these concepts cover:

- state predictability
- identity and equality
- event-rate control
- computation reuse
- functional transformation patterns
- recursive problem solving
- metaprogramming
- cloning/data isolation

---

## Key Takeaways

- Reference assignment is not cloning.
- Spread/Object.assign are shallow.
- `structuredClone()` deeply clones many supported types.
- It supports circular references and richer built-ins such as Map, Set, and Date.
- Functions and many special runtime objects are not cloneable.
- Transferables can move ownership instead of copying.
- Deep cloning is not the same as immutable updating.
- Structural sharing is usually better for React/state updates.
- Clone only as deeply as the actual use case requires.
