# Lesson 91 — Symbols

`Symbol` is a primitive JavaScript data type used to create unique values.

A common use is creating property keys that do not collide with normal string keys.

---

## 1. Create a Symbol

```js
const id =
  Symbol();
```

With description:

```js
const id =
  Symbol("id");
```

The description is mainly for debugging.

---

## 2. Every Symbol Is Unique

```js
const a =
  Symbol("id");

const b =
  Symbol("id");

console.log(
  a === b
);
```

Output:

```text
false
```

Same description does not mean same Symbol.

---

## 3. Symbols Are Primitive Values

```js
typeof Symbol("id");
```

Output:

```text
symbol
```

Symbol is one of JavaScript's primitive types.

---

## 4. Symbol as Object Key

```js
const id =
  Symbol("id");

const user = {
  name: "Vikash",
  [id]: 101,
};
```

Read:

```js
user[id];
```

This property does not collide with:

```js
user.id
```

because Symbol keys are distinct from string keys.

---

## 5. Avoiding Name Collisions

Suppose a library wants to attach internal metadata:

```js
const internal =
  Symbol(
    "internal"
  );

user[internal] = {
  cached: true,
};
```

Another library using:

```js
Symbol("internal")
```

gets a different key.

No accidental collision.

---

## 6. Symbols Are Not Truly Private

Important:

> Symbol-keyed properties are hidden from many common string-key enumeration APIs, but they are not private.

You can retrieve them:

```js
Object.getOwnPropertySymbols(
  user
);
```

So Symbol is not a security/privacy mechanism.

---

## 7. Object.keys()

Given:

```js
const id =
  Symbol("id");

const user = {
  name: "Vikash",
  [id]: 101,
};
```

```js
Object.keys(user);
```

returns:

```js
["name"]
```

Symbol keys are not included.

---

## 8. Reflect.ownKeys()

```js
Reflect.ownKeys(
  user
);
```

returns both:

- string keys
- Symbol keys

This connects to Lesson 50.

---

## 9. JSON.stringify()

Symbol-keyed properties are generally ignored by `JSON.stringify()`.

Example:

```js
JSON.stringify(
  user
);
```

will not include the Symbol-keyed `id` property.

---

## 10. Symbol.for()

`Symbol.for()` uses a global Symbol registry.

```js
const a =
  Symbol.for(
    "app.id"
  );

const b =
  Symbol.for(
    "app.id"
  );

console.log(
  a === b
);
```

Output:

```text
true
```

This differs from `Symbol()`.

---

## 11. Symbol.keyFor()

```js
const symbol =
  Symbol.for(
    "app.id"
  );

Symbol.keyFor(
  symbol
);
```

Returns:

```text
app.id
```

For a normal non-registry Symbol:

```js
Symbol.keyFor(
  Symbol("id")
);
```

returns `undefined`.

---

## 12. Registered vs Unique Symbol

```text
Symbol("x")
→ always fresh unique Symbol

Symbol.for("x")
→ returns shared registry Symbol
```

Use the registry only when shared identity is intentional.

---

## 13. Well-Known Symbols

JavaScript defines built-in Symbols that customize language behavior.

Examples:

```text
Symbol.iterator
Symbol.toPrimitive
Symbol.toStringTag
Symbol.hasInstance
```

These are called **well-known Symbols**.

---

## 14. Symbol.iterator

Objects use `Symbol.iterator` to participate in iteration.

Example:

```js
const iteratorMethod =
  array[
    Symbol.iterator
  ];
```

Lesson 92 will explore this deeply.

---

## 15. Symbol.toPrimitive

You can customize primitive conversion.

```js
const user = {
  name: "Vikash",

  [Symbol.toPrimitive](
    hint
  ) {
    if (
      hint === "string"
    ) {
      return this.name;
    }

    return 1;
  },
};
```

This affects language conversion behavior.

---

## 16. Symbol.toStringTag

```js
const collection = {
  [Symbol.toStringTag]:
    "CustomCollection",
};
```

Then built-in string tagging behavior can display the custom tag.

This is mostly used in libraries and platform objects.

---

## 17. Symbols and Object Spread

Enumerable own Symbol properties are copied by modern object spread.

Example:

```js
const key =
  Symbol("key");

const a = {
  [key]: 1,
};

const b = {
  ...a,
};

console.log(
  b[key]
);
```

Output:

```text
1
```

Do not assume "Symbol" means excluded from every object operation.

---

## 18. Property Descriptors

Symbol keys work with descriptor APIs:

```js
Object.defineProperty(
  user,
  id,
  {
    value: 101,
    enumerable: false,
  }
);
```

The key type and descriptor behavior are separate concepts.

---

## 19. Use Cases

Symbols are useful for:

- collision-resistant property keys
- language protocols
- library metadata
- custom iteration
- custom coercion
- framework internals

---

## 20. React/Application Connection

Normal React applications rarely need custom Symbols directly.

But JavaScript and many libraries use Symbols internally.

Understanding them helps with:

- iterables
- inspection/debugging
- framework internals
- advanced API design

---

## Interview Questions

### Are two Symbols with the same description equal?

No.

### Are Symbol properties private?

No.

### Symbol() vs Symbol.for()?

`Symbol()` creates a fresh unique Symbol. `Symbol.for()` retrieves or creates a shared Symbol from the global registry.

### What is Symbol.iterator?

A well-known Symbol used to define an object's iteration behavior.

---

## Key Takeaways

- Symbols are primitive unique values.
- They can be used as object property keys.
- Same descriptions do not create equal Symbols.
- Symbol-keyed properties are not private.
- `Symbol.for()` provides shared registry identity.
- Well-known Symbols customize language protocols.
- `Symbol.iterator` connects directly to iterable behavior.
