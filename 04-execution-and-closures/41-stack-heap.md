# Lesson 41 — Stack vs Heap Memory

"Stack vs Heap" is a common JavaScript interview topic, but it is also frequently explained too simplistically.

A common statement is:

> Primitives are stored on the stack and objects are stored on the heap.

That can be a useful beginner approximation, but it is **not a JavaScript language guarantee**.

A better interview answer focuses on:

- call stack
- references
- object identity
- lifetime
- garbage collection
- engine implementation freedom

---

## 1. What Is the Call Stack?

The call stack tracks active function execution.

Example:

```js
function a() {
  b();
}

function b() {
  console.log(
    "Hello"
  );
}

a();
```

At the deepest point:

```text
┌──────────────┐
│ b()          │
├──────────────┤
│ a()          │
├──────────────┤
│ global       │
└──────────────┘
```

This is execution-stack behavior.

---

## 2. What Is Heap Memory?

Conceptually, heap memory is a larger memory region used by runtimes for dynamically allocated data whose lifetime is not tied directly to one stack frame.

Objects, arrays, functions, closures, and internal runtime structures are often associated with heap allocation in typical engines.

But exact placement is an implementation detail.

---

## 3. Safe Interview Mental Model

Use this:

```text
Call Stack
→ active execution frames

Heap-like managed memory
→ dynamically allocated objects/data
```

Avoid claiming exact physical storage for every JavaScript value.

---

## 4. Primitive Values

Primitive types include:

- string
- number
- bigint
- boolean
- undefined
- symbol
- null

Example:

```js
let a = 10;

let b = a;
```

Changing `b` does not affect `a`.

```js
b = 20;

console.log(a);
// 10
```

Why?

Because primitives behave as values.

---

## 5. Objects Use Reference-Like Values

```js
const user1 = {
  name:
    "Vikash",
};

const user2 =
  user1;
```

Now both variables refer to the same object.

```js
user2.name =
  "Rahul";

console.log(
  user1.name
);
// Rahul
```

---

## 6. JavaScript Is Still Pass-by-Value

This is extremely important.

JavaScript is pass-by-value.

For objects, the value being copied is a reference-like value pointing to the same object.

Conceptually:

```text
user1
↓
reference value
↓
object

user2
↓
copied same reference value
↓
same object
```

---

## 7. Function Argument Example

```js
function updateUser(
  user
) {
  user.name =
    "Kumar";
}

const user = {
  name:
    "Vikash",
};

updateUser(user);

console.log(
  user.name
);
// Kumar
```

The function receives a copy of the reference value.

Both references point to the same object.

---

## 8. Reassignment Does Not Replace Caller Variable

```js
function replaceUser(
  user
) {
  user = {
    name:
      "New"
  };
}

const user = {
  name:
    "Vikash",
};

replaceUser(user);

console.log(
  user.name
);
// Vikash
```

Why?

The parameter receives a copy of the reference value.

Reassigning the parameter changes only that local parameter binding.

---

## 9. Mutation vs Reassignment

### Mutation

```js
user.name =
  "Kumar";
```

Changes the referenced object.

### Reassignment

```js
user = {
  name:
    "Kumar",
};
```

Changes which value the variable binding refers to.

This distinction is critical.

---

## 10. Reference Equality

```js
const a = {
  x: 1,
};

const b = {
  x: 1,
};
```

Then:

```js
a === b
```

is:

```text
false
```

Different object identities.

But:

```js
const c = a;

c === a
```

is:

```text
true
```

because they refer to the same object.

---

## 11. Shallow Copy

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
```

The root object is new.

But:

```js
copy.profile ===
original.profile
```

is:

```text
true
```

Nested object reference is shared.

---

## 12. Memory Diagram

Conceptually:

```text
original ──► object A
              │
              └── profile ──► object X

copy ─────► object B
              │
              └── profile ──► object X
```

The nested object is shared.

---

## 13. Deep Clone

With:

```js
const clone =
  structuredClone(
    original
  );
```

conceptually:

```text
original → object A → nested X

clone    → object B → nested Y
```

Now nested references are independent for supported cloneable values.

---

## 14. Closures and Memory Lifetime

```js
function outer() {
  const data = {
    value: 10,
  };

  return function () {
    return data.value;
  };
}
```

After `outer()` returns, its call-stack frame is gone.

But `data` can remain reachable because the returned closure still references it.

This shows:

> stack lifetime and data lifetime are not the same thing.

---

## 15. Call Stack Frame Removed ≠ Data Immediately Destroyed

A common misconception:

```text
function returns
→ all local data disappears
```

Not always.

If local values remain reachable through:

- closures
- objects
- callbacks
- global references

they can remain alive.

---

## 16. Garbage Collection Uses Reachability

```js
let user = {
  name:
    "Vikash",
};

user = null;
```

If nothing else refers to the object, it may become unreachable and eligible for garbage collection.

The object is not freed because "its stack variable was deleted".

Reachability is the important model.

---

## 17. Circular References

```js
let a = {};
let b = {};

a.b = b;
b.a = a;

a = null;
b = null;
```

Even though the objects point to each other, they can still be garbage collected if no reachable root points to them.

Modern GC is based on reachability, not simple reference counting.

---

## 18. Why the Stack/Heap Shortcut Is Dangerous

Suppose an interviewer asks:

> Are strings always stored on the stack?

A strong answer is:

> JavaScript does not specify exact physical memory placement. Engines are free to optimize values. The useful conceptual distinction is that the call stack tracks active execution, while objects and other dynamic data are managed by the engine according to lifetime and reachability.

---

## 19. Engine Optimizations

Modern engines may:

- keep values in registers
- inline objects
- optimize short-lived allocations
- move objects during garbage collection
- represent values differently internally

So exact physical memory placement is engine-specific.

---

## 20. Escape Analysis

Engines may detect that an object never escapes a local context and optimize its allocation.

Conceptually:

```js
function calculate() {
  const point = {
    x: 1,
    y: 2,
  };

  return (
    point.x +
    point.y
  );
}
```

An engine is free to optimize this aggressively.

This is another reason not to make rigid stack/heap claims.

---

## 21. Stack Overflow vs Heap Pressure

These are different problems.

### Stack Overflow

Too many active nested calls.

Example:

```js
function recurse() {
  recurse();
}
```

### Heap / Managed-Memory Pressure

Too many retained objects or large allocations.

Example:

```js
const data = [];

while (true) {
  data.push(
    new Array(
      1_000_000
    )
  );
}
```

Different causes, different symptoms.

---

## 22. Recursion and Stack Memory

Each recursive call adds another active frame.

```js
function factorial(
  n
) {
  if (
    n <= 1
  ) {
    return 1;
  }

  return (
    n *
    factorial(
      n - 1
    )
  );
}
```

Very deep recursion may overflow the call stack.

---

## 23. Arrays and Objects

Arrays are objects in JavaScript.

```js
typeof [];
// "object"
```

Their dynamic data is typically managed as object-like runtime allocations.

Again, exact representation is engine-specific.

---

## 24. Functions Are Objects Too

Functions can:

- have properties
- be stored
- be passed
- close over variables

Their runtime representation is more complex than a simple "function on stack" statement.

---

## 25. React Connection — Referential Equality

```jsx
const user = {
  name:
    "Vikash",
};
```

Creating this during every render creates a new object identity.

That matters for:

- dependency arrays
- memoization
- `React.memo`
- equality checks

This connects directly to object/reference semantics.

---

## 26. React State Update Example

Bad mutation:

```js
user.name =
  "Rahul";

setUser(user);
```

The object reference may remain the same.

Better:

```js
setUser({
  ...user,
  name:
    "Rahul",
});
```

Now the root object gets a new reference.

---

## 27. Interview Trap — Pass by Reference

Avoid saying:

> JavaScript passes objects by reference.

More precise:

> JavaScript passes everything by value. For objects, the copied value refers to the same object.

This explains both mutation and reassignment behavior correctly.

---

## 28. Interview Problem

```js
function change(
  obj
) {
  obj.value = 2;

  obj = {
    value: 3,
  };
}

const item = {
  value: 1,
};

change(item);

console.log(
  item.value
);
```

Output:

```text
2
```

Why?

First:

```js
obj.value = 2;
```

mutates the shared object.

Then:

```js
obj = {
  value: 3,
};
```

only reassigns the local parameter.

---

## 29. Interview Mental Model

Use:

```text
Call Stack
→ active execution contexts

Bindings
→ hold values

Primitive value
→ copied as value

Object variable
→ holds a reference-like value

Copy object variable
→ copy reference-like value

Garbage Collection
→ based on reachability
```

This is accurate enough for interviews without pretending to know engine-specific memory layout.

---

## Interview Answer — 30 Seconds

> The call stack tracks active execution contexts and function calls. Objects and other dynamic runtime data are commonly managed in heap-like memory, but JavaScript does not specify exact physical placement. The more important concept is that JavaScript is pass-by-value: primitives copy their values, while object variables copy a reference-like value pointing to the same object. Garbage collection is based on reachability, not simply on whether a function returned.

---

## Section 4 Complete Mental Model

```text
JavaScript Engine + Runtime
↓
Execution Context
↓
Creation Phase
↓
Hoisting / TDZ
↓
Call Stack
↓
Lexical Environment
↓
Closures
↓
Practical Closure Patterns
↓
Memory / References / Reachability
```

---

## Key Takeaways

- Call stack tracks active execution.
- Heap-like memory is an implementation concept for dynamic data.
- Do not claim primitives always physically live on the stack.
- JavaScript is pass-by-value.
- Object variables copy reference-like values.
- Mutation and reassignment are different.
- Closures can keep data alive after a function returns.
- Garbage collection is based on reachability.
- Stack overflow and heap pressure are different problems.
