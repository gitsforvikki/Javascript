# Lesson 101 — Value vs Reference and Referential Equality

To understand advanced JavaScript behavior, you need a precise mental model of:

- primitive values
- object references
- assignment
- function arguments
- equality
- referential identity

A useful rule is:

> JavaScript variables always hold values. For objects, that value behaves like a reference to the object.

Avoid the oversimplified phrase:

> primitives are passed by value, objects are passed by reference

JavaScript is better described as **pass-by-value**, where object-reference values are copied.

---

## 1. Primitive Copy

```js
let a = 10;
let b = a;

b = 20;
```

Now:

```js
console.log(a);
// 10

console.log(b);
// 20
```

The value `10` was copied.

---

## 2. Object Reference Copy

```js
const userA = {
  name:
    "Vikash",
};

const userB =
  userA;
```

Conceptually:

```text
userA ─┐
       ├──► object X
userB ─┘
```

Both variables contain reference values pointing to the same object.

---

## 3. Mutating Through Either Reference

```js
userB.name =
  "Rahul";
```

Then:

```js
console.log(
  userA.name
);
```

Output:

```text
Rahul
```

Because there is only one object.

---

## 4. Reassignment Is Different from Mutation

```js
let userA = {
  name:
    "Vikash",
};

let userB =
  userA;

userB = {
  name:
    "Rahul",
};
```

Now:

```text
userA ──► object X

userB ──► object Y
```

Reassigning `userB` did not modify object X.

---

## 5. Primitive Equality

```js
10 === 10
// true

"hello" === "hello"
// true

true === true
// true
```

Primitive equality generally compares values.

---

## 6. Object Equality

```js
{} === {}
```

Output:

```text
false
```

Why?

Those are two distinct objects.

---

## 7. Referential Equality

```js
const a = {
  id: 1,
};

const b = a;

console.log(
  a === b
);
```

Output:

```text
true
```

Both references identify the same object.

This is **referential equality**.

---

## 8. Same Shape Does Not Mean Same Reference

```js
const a = {
  id: 1,
};

const b = {
  id: 1,
};

console.log(
  a === b
);
```

Output:

```text
false
```

Even though their contents look identical.

---

## 9. Arrays Are Objects

```js
[] === []
```

Output:

```text
false
```

But:

```js
const a = [];
const b = a;

a === b;
// true
```

Arrays use object identity.

---

## 10. Functions Are Objects Too

```js
const a = () => {};
const b = () => {};

console.log(
  a === b
);
```

Output:

```text
false
```

Each function expression creates a new function object.

---

## 11. Function Argument Mental Model

Primitive:

```js
function update(
  value
) {
  value = 100;
}

let count = 10;

update(count);

console.log(count);
// 10
```

The primitive value was copied into the parameter.

---

## 12. Object Argument

```js
function update(
  user
) {
  user.name =
    "Rahul";
}

const person = {
  name:
    "Vikash",
};

update(person);
```

`person.name` changes.

Why?

The reference value was copied into the parameter.

Both references identify the same object.

---

## 13. Reassigning Object Parameter

```js
function replace(
  user
) {
  user = {
    name:
      "Rahul",
  };
}

const person = {
  name:
    "Vikash",
};

replace(person);
```

The original `person` object is unchanged.

Why?

Only the local parameter reference was reassigned.

---

## 14. Pass-by-Value with References

Correct mental model:

```text
caller variable
contains reference value R

function parameter
receives copied reference value R

both point to same object
```

So JavaScript is still pass-by-value.

---

## 15. NaN Equality

```js
NaN === NaN
```

Output:

```text
false
```

Use:

```js
Number.isNaN(
  value
);
```

or:

```js
Object.is(
  NaN,
  NaN
);
// true
```

---

## 16. Object.is()

`Object.is()` is similar to strict equality with a few differences.

```js
Object.is(
  NaN,
  NaN
);
// true
```

```js
Object.is(
  +0,
  -0
);
// false
```

while:

```js
+0 === -0
// true
```

---

## 17. SameValueZero

Some collections use SameValueZero equality.

Examples:

- `Set`
- `Map` key comparison
- `Array.prototype.includes()`

This means:

```text
NaN equals NaN

+0 equals -0
```

for those operations.

---

## 18. Reference Identity in Map

```js
const map =
  new Map();

const key = {
  id: 1,
};

map.set(
  key,
  "value"
);

map.get({
  id: 1,
});
// undefined
```

A structurally identical new object is still a different key.

---

## 19. Reference Identity in Set

```js
const a = {
  id: 1,
};

const b = {
  id: 1,
};

const set =
  new Set([
    a,
    b,
  ]);

console.log(
  set.size
);
// 2
```

Two different object references remain unique entries.

---

## 20. React Props Example

Parent:

```jsx
<Child
  options={{
    theme:
      "dark",
  }}
/>
```

On every render, a new object is created.

Even if content is identical:

```text
previous options
≠
new options reference
```

This matters for memoization.

---

## 21. React State Identity

```js
setUser(user);
```

If `user` is the exact same reference as current state, React may treat it as unchanged.

New immutable update:

```js
setUser({
  ...user,
  name:
    "Rahul",
});
```

creates a new identity.

---

## 22. useEffect Dependency Example

```js
const options = {
  page: 1,
};

useEffect(() => {
  // ...
}, [options]);
```

A new `options` object on every render means the dependency reference changes every render.

That can cause the effect to rerun.

---

## 23. useMemo Connection

```js
const options =
  useMemo(
    () => ({
      page: 1,
    }),
    []
  );
```

Now the object reference can remain stable across renders.

Memoization is about controlling referential identity when it matters.

---

## 24. useCallback Connection

```js
const handleSave =
  useCallback(
    () => {
      // ...
    },
    []
  );
```

Without `useCallback`, a new function object is created each render.

Again:

```text
same code
≠
same function reference
```

---

## 25. Referential Stability

A reference is stable when repeated executions reuse the same object/function identity when appropriate.

This matters for:

- memoized children
- effect dependencies
- caches
- Maps/Sets
- dependency tracking

Do not memoize everything blindly.

Stability matters only where identity participates in behavior or optimization.

---

## 26. String Primitives Are Not Reference Types

Strings may be implemented efficiently internally, but at the language level:

```js
"hello" === "hello"
```

compares primitive string values.

Do not reason about JavaScript equality from low-level memory implementation guesses.

---

## 27. Common Misconception — Stack vs Heap

You may hear:

```text
primitives on stack
objects on heap
```

This is an implementation-level oversimplification, not a JavaScript language guarantee.

For language reasoning, focus on:

- primitive values
- object identity
- reference values
- equality semantics

---

## 28. Interview Questions

### Are objects passed by reference in JavaScript?

A more precise answer: JavaScript is pass-by-value. For objects, the value copied is a reference to the object.

### Why is {} === {} false?

Because each object literal creates a different object identity.

### Why can mutating a function parameter affect the caller's object?

Because the copied reference still points to the same object.

### Reassignment vs mutation?

Reassignment changes which value a variable holds. Mutation changes the referenced object itself.

---

## Key Takeaways

- JavaScript variables hold values.
- Object variables hold reference values.
- Assigning an object reference copies the reference value.
- Multiple references can point to one object.
- `===` compares object identity, not structure.
- JavaScript is pass-by-value.
- Function and array equality is also identity-based.
- Referential identity matters heavily in React and memoization.
