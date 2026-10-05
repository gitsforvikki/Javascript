# Lesson 100 — Mutation vs Immutability

Mutation and immutability are foundational concepts in modern JavaScript.

They affect:

- state management
- React rendering
- debugging
- data flow
- equality checks
- memoization
- predictable application behavior

The key idea is:

> Mutation changes an existing value or object in place. Immutability means producing a new value instead of modifying the original.

---

## 1. Mutation with Objects

```js
const user = {
  name: "Vikash",
  role: "Frontend Developer",
};

user.role =
  "Full Stack Developer";
```

The same object was changed.

Conceptually:

```text
before
user ─────► object A

after mutation
user ─────► same object A
             role changed
```

The object identity did not change.

---

## 2. Immutable Object Update

Instead of changing the original:

```js
const updatedUser = {
  ...user,
  role:
    "Full Stack Developer",
};
```

Now:

```text
user
↓
object A

updatedUser
↓
object B
```

The original object remains unchanged.

---

## 3. Compare Identity

```js
console.log(
  user === updatedUser
);
```

Output:

```text
false
```

Even if most properties are equal, they are different objects.

This matters heavily in React.

---

## 4. Mutation with Arrays

```js
const skills = [
  "JavaScript",
  "React",
];

skills.push(
  "Next.js"
);
```

`push()` mutates the original array.

Other common mutating array methods include:

```text
push
pop
shift
unshift
splice
sort
reverse
fill
copyWithin
```

---

## 5. Immutable Array Update

Add item:

```js
const updatedSkills = [
  ...skills,
  "Node.js",
];
```

Remove:

```js
const filtered =
  skills.filter(
    (skill) =>
      skill !== "React"
  );
```

Update:

```js
const updated =
  skills.map(
    (skill) =>
      skill === "React"
        ? "React 19"
        : skill
  );
```

These create new arrays.

---

## 6. Non-Mutating vs Mutating Methods

Common non-mutating methods:

```text
map
filter
slice
concat
toSorted
toReversed
toSpliced
with
```

Modern methods such as:

```js
array.toSorted();
array.toReversed();
```

return new arrays rather than changing the original.

---

## 7. sort() Mutation Trap

```js
const numbers = [
  3,
  1,
  2,
];

const sorted =
  numbers.sort();
```

Both variables reference the same mutated array.

```js
console.log(
  numbers
);
```

Output:

```js
[1, 2, 3]
```

Safer immutable version:

```js
const sorted =
  numbers.toSorted();
```

or:

```js
const sorted =
  [...numbers].sort();
```

---

## 8. Immutability Does Not Mean const

This is important.

```js
const user = {
  name: "Vikash",
};

user.name =
  "Rahul";
```

This is valid.

`const` prevents reassignment of the variable binding:

```js
user = {};
```

but does not make the object immutable.

---

## 9. Object.freeze()

```js
const user =
  Object.freeze({
    name: "Vikash",
  });
```

In strict mode, attempts to modify frozen properties can throw.

But:

> `Object.freeze()` is shallow.

---

## 10. Shallow Freeze

```js
const user =
  Object.freeze({
    profile: {
      city:
        "Bengaluru",
    },
  });
```

This nested mutation may still work:

```js
user.profile.city =
  "Pune";
```

Because only the top-level object was frozen.

---

## 11. Shallow Copy Does Not Mean Deep Immutability

```js
const user = {
  profile: {
    city:
      "Bengaluru",
  },
};

const copy = {
  ...user,
};
```

Now:

```js
copy.profile.city =
  "Pune";
```

also affects:

```js
user.profile.city
```

because both objects share the same nested `profile` reference.

---

## 12. Correct Nested Immutable Update

```js
const updated = {
  ...user,

  profile: {
    ...user.profile,

    city:
      "Pune",
  },
};
```

Now both levels that change receive new references.

---

## 13. Structural Sharing

Immutable updates do not require cloning everything.

Example:

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

Update user:

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

The `settings` object is reused.

Conceptually:

```text
old state
├── user A
└── settings X

new state
├── user B
└── settings X
```

This is called **structural sharing**.

---

## 14. Why Immutability Helps Debugging

Mutation:

```text
same object changes over time
↓
harder to know when value changed
```

Immutability:

```text
old value
→ new value
→ another new value
```

This creates clearer state transitions.

It helps with:

- logs
- undo/redo
- time-travel debugging
- memoization
- predictable updates

---

## 15. Function Side Effects

Mutating function:

```js
function addRole(
  user
) {
  user.role =
    "Developer";
}
```

The caller's object changes.

Pure-style version:

```js
function addRole(
  user
) {
  return {
    ...user,
    role:
      "Developer",
  };
}
```

The original input remains unchanged.

---

## 16. Mutation Is Not Always Bad

Mutation itself is not forbidden JavaScript.

Local mutation can be perfectly reasonable.

Example:

```js
function sum(
  numbers
) {
  let total = 0;

  for (
    const number
    of numbers
  ) {
    total += number;
  }

  return total;
}
```

`total` mutates locally, but no shared external state is changed.

The real danger is often:

> uncontrolled shared mutation.

---

## 17. Shared Mutation Problem

```js
const config = {
  debug: false,
};

function enableDebug() {
  config.debug =
    true;
}
```

Any code holding `config` observes the mutation.

Shared mutable state increases coupling.

---

## 18. Copy-on-Write Mental Model

A useful immutable strategy:

```text
Need to change?
↓
copy affected path
↓
change copy
↓
return new root
```

Not:

```text
deep-clone everything
```

This is why structural sharing is important.

---

## 19. React State Mutation Bug

Bad:

```js
const [
  user,
  setUser,
] = useState({
  name:
    "Vikash",
});

function update() {
  user.name =
    "Rahul";

  setUser(user);
}
```

The same reference is passed back.

React may not detect the change as expected because state identity did not change.

---

## 20. Correct React State Update

```js
setUser((prev) => ({
  ...prev,
  name:
    "Rahul",
}));
```

New reference:

```text
prev object
≠
next object
```

This works naturally with React's reference-based state model.

---

## 21. Nested React State

Bad:

```js
state.user.profile.city =
  "Pune";
```

Better:

```js
setState((prev) => ({
  ...prev,

  user: {
    ...prev.user,

    profile: {
      ...prev.user.profile,

      city:
        "Pune",
    },
  },
}));
```

Only changed paths receive new references.

---

## 22. Redux Connection

Redux strongly encourages immutable updates.

Why?

Because Redux relies on predictable state transitions and reference comparisons.

Modern Redux Toolkit uses Immer internally, so code may look mutative:

```js
state.count++;
```

but Immer produces immutable state updates underneath.

---

## 23. Immer Mental Model

Conceptually:

```text
write mutation-like code
↓
Immer tracks changes
↓
produces new immutable result
↓
unchanged branches reused
```

Do not confuse Immer's syntax with actual direct mutation of application state.

---

## 24. Deep Clone Is Not the Default Solution

This is often wasteful:

```js
structuredClone(
  state
);
```

for every tiny update.

Better:

- clone only changed branches
- reuse unaffected references

Deep cloning everything can:

- cost memory
- cost CPU
- destroy useful referential equality

---

## 25. structuredClone()

When you truly need a deep clone of supported data:

```js
const copy =
  structuredClone(
    original
  );
```

It handles many built-in data types better than JSON-based cloning.

But it still has limitations and should not replace thoughtful state design.

---

## 26. JSON Clone Problem

You may see:

```js
const copy =
  JSON.parse(
    JSON.stringify(
      value
    )
  );
```

This is not a general deep-clone solution.

It loses or changes values such as:

- `undefined`
- functions
- Symbols
- some built-in object types
- special numeric values

Use `structuredClone()` when appropriate.

---

## 27. Mutation and Performance

Immutability creates new objects, but that does not automatically mean it is slow.

Benefits include:

- cheap reference comparisons
- memoization
- selective updates
- predictable caching

Performance depends on the specific workload.

Avoid premature assumptions.

---

## 28. Interview Questions

### Mutation vs immutability?

Mutation changes an existing value in place. Immutable updates create a new value while leaving the original unchanged.

### Does const make an object immutable?

No.

### Is spread a deep copy?

No. It is shallow.

### Why is immutability useful in React?

React commonly relies on reference identity to detect and optimize state/prop changes.

---

## Key Takeaways

- Mutation changes existing data in place.
- Immutability creates new data.
- `const` does not freeze objects.
- Spread and `Object.assign` are shallow.
- Nested immutable updates require copying changed paths.
- Structural sharing reuses unchanged branches.
- Shared mutation is the main source of unpredictability.
- React state should generally be updated immutably.
- Deep cloning everything is usually unnecessary.
