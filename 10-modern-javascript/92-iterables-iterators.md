# Lesson 92 — Iterables and Iterators

Iteration is one of JavaScript's core protocols.

When you write:

```js
for (const value of array) {
  console.log(value);
}
```

JavaScript is using the iterable and iterator protocols underneath.

This lesson explains that mechanism clearly.

---

## 1. What Is an Iterable?

An iterable is an object that knows how to produce an iterator.

Technically, it provides a method at:

```js
Symbol.iterator
```

Example:

```js
const numbers = [
  10,
  20,
  30,
];

console.log(
  typeof numbers[
    Symbol.iterator
  ]
);
```

Output:

```text
function
```

Arrays are iterable.

---

## 2. Built-In Iterables

Common built-in iterables include:

- arrays
- strings
- maps
- sets
- typed arrays
- many DOM collections

Plain objects are not iterable by default.

Example:

```js
const user = {
  name: "Vikash",
};

for (const value of user) {
  // TypeError
}
```

---

## 3. Iterable Protocol

An iterable must expose:

```js
object[
  Symbol.iterator
]
```

as a function.

Calling it should return an iterator.

Mental model:

```text
iterable
↓
Symbol.iterator()
↓
iterator
```

---

## 4. What Is an Iterator?

An iterator is an object with a `next()` method.

Example:

```js
const iterator =
  [10, 20][
    Symbol.iterator
  ]();

console.log(
  iterator.next()
);
```

Result:

```js
{
  value: 10,
  done: false,
}
```

---

## 5. Iterator Result Object

Each call to:

```js
iterator.next()
```

returns an object with:

```text
value
done
```

Example sequence:

```js
iterator.next();
// { value: 10, done: false }

iterator.next();
// { value: 20, done: false }

iterator.next();
// { value: undefined, done: true }
```

---

## 6. What done Means

```text
done: false
→ iteration still has values

done: true
→ iteration is complete
```

Once complete, further calls usually continue returning a completed result.

---

## 7. for...of Uses the Protocol

This:

```js
for (
  const value
  of numbers
) {
  console.log(value);
}
```

conceptually does something similar to:

```js
const iterator =
  numbers[
    Symbol.iterator
  ]();

while (true) {
  const result =
    iterator.next();

  if (result.done) {
    break;
  }

  console.log(
    result.value
  );
}
```

The real language semantics are more detailed, but this is the correct learning model.

---

## 8. Iterable vs Iterator

These terms are related but not identical.

```text
Iterable
→ can create an iterator

Iterator
→ produces values one at a time
```

Example:

```js
const array =
  [1, 2, 3];

const iterator =
  array[
    Symbol.iterator
  ]();
```

`array` is the iterable.

`iterator` is the object producing values.

---

## 9. String Iterator

```js
const iterator =
  "JS"[
    Symbol.iterator
  ]();

console.log(
  iterator.next()
);

console.log(
  iterator.next()
);
```

Strings are iterable.

This is why:

```js
[..."hello"]
```

works.

---

## 10. Spread Uses Iterables

Array spread:

```js
const values =
  [...new Set([
    1,
    2,
    3,
  ])];
```

The spread syntax consumes the Set's iterator.

That connects iteration protocol to modern syntax.

---

## 11. Array.from()

```js
const array =
  Array.from(
    new Set([
      1,
      2,
      3,
    ])
  );
```

`Array.from()` can consume iterable values.

---

## 12. Destructuring Uses Iteration

```js
const [
  first,
  second,
] = new Set([
  "React",
  "Next.js",
]);
```

Array-pattern destructuring consumes an iterable.

This explains why destructuring works with Sets, not just arrays.

---

## 13. Map Iteration

```js
const map =
  new Map([
    ["name", "Vikash"],
    ["role", "Developer"],
  ]);

for (
  const entry
  of map
) {
  console.log(entry);
}
```

Each value is:

```js
[key, value]
```

Map's default iterator yields entries.

---

## 14. Set Iteration

```js
const set =
  new Set([
    "React",
    "Next.js",
  ]);

for (
  const value
  of set
) {
  console.log(value);
}
```

Set's iterator yields values.

---

## 15. Plain Object Is Not Iterable

```js
const user = {
  name: "Vikash",
  role: "Developer",
};
```

This fails:

```js
for (
  const item
  of user
) {
}
```

Because the object has no default `Symbol.iterator`.

---

## 16. Iterate Object Data with Object.entries()

```js
for (
  const [key, value]
  of Object.entries(
    user
  )
) {
  console.log(
    key,
    value
  );
}
```

`Object.entries()` creates an array, which is iterable.

---

## 17. Build a Custom Iterator

```js
const range = {
  start: 1,
  end: 3,

  [Symbol.iterator]() {
    let current =
      this.start;

    const end =
      this.end;

    return {
      next() {
        if (
          current <= end
        ) {
          return {
            value:
              current++,
            done: false,
          };
        }

        return {
          value:
            undefined,
          done: true,
        };
      },
    };
  },
};
```

Now:

```js
for (
  const number
  of range
) {
  console.log(number);
}
```

Output:

```text
1
2
3
```

---

## 18. Why Custom Iterables Matter

Custom iteration lets an object define:

> What does it mean to iterate over me?

Possible use cases:

- ranges
- tree traversal
- paginated sequences
- custom collections
- domain-specific structures

---

## 19. Iterable Iterator

Sometimes the iterator itself is also iterable.

It can return itself from:

```js
[Symbol.iterator]()
```

Example shape:

```js
const iterator = {
  next() {
    // ...
  },

  [Symbol.iterator]() {
    return this;
  },
};
```

This allows the iterator itself to be used with `for...of`.

---

## 20. Iterator Is Stateful

```js
const iterator =
  [10, 20, 30][
    Symbol.iterator
  ]();

iterator.next();
// 10

iterator.next();
// 20
```

The iterator remembers its current position.

This internal state is why repeated `next()` calls advance.

---

## 21. New Iterator Starts Fresh

```js
const array =
  [10, 20];

const a =
  array[
    Symbol.iterator
  ]();

const b =
  array[
    Symbol.iterator
  ]();
```

`a` and `b` have independent iteration state.

---

## 22. for...in vs for...of

`for...in`:

```js
for (
  const key
  in object
) {
}
```

iterates enumerable property keys, including inherited enumerable string keys.

`for...of`:

```js
for (
  const value
  of iterable
) {
}
```

uses the iterable protocol to produce values.

Do not confuse them.

---

## 23. Early Termination

A `for...of` loop can end early with:

```js
break;
```

Iterators may optionally provide a `return()` method for cleanup when iteration stops early.

This is an advanced part of the iterator protocol.

Generators support this behavior naturally.

---

## 24. Async Iteration Preview

JavaScript also has an asynchronous iteration protocol using:

```js
Symbol.asyncIterator
```

and:

```js
for await (
  const value
  of asyncIterable
) {
}
```

That is a separate advanced topic.

The synchronous iterator protocol is the foundation.

---

## 25. React/Application Connection

You constantly consume iterables without thinking about it:

```js
const uniqueIds =
  [...new Set(ids)];
```

```js
for (
  const [key, value]
  of map
) {
}
```

Understanding the protocol explains why these language features work across different collection types.

---

## Interview Questions

### What is an iterable?

An object with a `Symbol.iterator` method that returns an iterator.

### What is an iterator?

An object with a `next()` method that returns `{ value, done }`.

### Is a plain object iterable by default?

No.

### Difference between for...in and for...of?

`for...in` iterates enumerable property keys; `for...of` consumes iterable values.

---

## Key Takeaways

- Iterables expose `Symbol.iterator`.
- Calling it returns an iterator.
- Iterators expose `next()`.
- `next()` returns `{ value, done }`.
- Arrays, strings, Maps, and Sets are iterable.
- Plain objects are not iterable by default.
- `for...of`, spread, and destructuring consume iterables.
- Custom objects can implement their own iteration behavior.
