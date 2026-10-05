# Lesson 50 — Object Internals and Property Descriptors

JavaScript objects look simple:

```js
const user = {
  name: "Vikash",
  age: 25,
};
```

But internally, object properties contain more information than just a key and a value.

A property can also have metadata such as:

- whether it can be changed
- whether it can be deleted
- whether it appears during enumeration
- whether it stores a value directly
- whether it uses getter/setter functions

This lesson builds the internal object model that later prototype lessons depend on.

---

## 1. What Is a JavaScript Object?

An object is a collection of properties.

Each property is identified by a property key.

Property keys can be:

- strings
- symbols

Example:

```js
const user = {
  name: "Vikash",
};

const id = Symbol("id");

user[id] = 101;
```

Conceptually:

```text
user
├── "name" → "Vikash"
└── Symbol(id) → 101
```

---

# 2. Properties Are More Than Key + Value

When you write:

```js
const user = {
  name: "Vikash",
};
```

you usually think:

```text
name → "Vikash"
```

But JavaScript internally associates attributes with that property.

For a normal property created this way, the descriptor is roughly:

```js
{
  value: "Vikash",
  writable: true,
  enumerable: true,
  configurable: true
}
```

These attributes are called a **property descriptor**.

---

# 3. Reading a Property Descriptor

Use:

```js
Object.getOwnPropertyDescriptor(
  object,
  propertyName
);
```

Example:

```js
const user = {
  name: "Vikash",
};

const descriptor =
  Object.getOwnPropertyDescriptor(
    user,
    "name"
  );

console.log(descriptor);
```

Typical result:

```js
{
  value: "Vikash",
  writable: true,
  enumerable: true,
  configurable: true
}
```

---

# 4. Own Property

Notice the method name:

```js
Object.getOwnPropertyDescriptor()
```

The word **own** is important.

An own property exists directly on the object itself.

Example:

```js
const user = {
  name: "Vikash",
};

console.log(
  user.hasOwnProperty("name")
);
```

Output:

```text
true
```

Later, you will learn that objects can also access properties inherited through the prototype chain.

Those are not own properties.

---

# 5. Data Property Descriptor

A normal value-based property is called a **data property**.

Its descriptor can contain:

```text
value
writable
enumerable
configurable
```

Example:

```js
const product = {};

Object.defineProperty(
  product,
  "price",
  {
    value: 1000,
    writable: true,
    enumerable: true,
    configurable: true,
  }
);
```

Now:

```js
console.log(product.price);
```

Output:

```text
1000
```

---

# 6. writable

`writable` controls whether the property's value can be reassigned.

Example:

```js
const user = {};

Object.defineProperty(
  user,
  "id",
  {
    value: 101,
    writable: false,
    enumerable: true,
    configurable: true,
  }
);
```

Now:

```js
user.id = 999;

console.log(user.id);
```

In non-strict code, the assignment fails silently.

The value remains:

```text
101
```

In strict mode:

```js
"use strict";

user.id = 999;
```

throws a `TypeError`.

Mental model:

```text
writable: true
→ value may be reassigned

writable: false
→ value cannot be reassigned
```

---

# 7. enumerable

`enumerable` controls whether a property appears in many enumeration operations.

Example:

```js
const user = {
  name: "Vikash",
};

Object.defineProperty(
  user,
  "secret",
  {
    value: "hidden",
    enumerable: false,
  }
);
```

Now:

```js
console.log(user.secret);
```

Output:

```text
hidden
```

But:

```js
console.log(
  Object.keys(user)
);
```

Output:

```js
["name"]
```

The property exists and is accessible, but it is not enumerable.

---

# 8. configurable

`configurable` controls whether a property can later be deleted or have most descriptor attributes reconfigured.

Example:

```js
const user = {};

Object.defineProperty(
  user,
  "id",
  {
    value: 101,
    writable: true,
    enumerable: true,
    configurable: false,
  }
);
```

Now:

```js
delete user.id;
```

will not remove the property.

Also, you generally cannot later change `configurable` back to `true`.

This setting is intentionally restrictive.

---

# 9. Important Difference: Object Literal vs defineProperty Defaults

When you create a normal object literal property:

```js
const user = {
  name: "Vikash",
};
```

the descriptor defaults are:

```text
writable: true
enumerable: true
configurable: true
```

But when using:

```js
Object.defineProperty(...)
```

and you omit descriptor flags, they default to `false`.

Example:

```js
const user = {};

Object.defineProperty(
  user,
  "name",
  {
    value: "Vikash",
  }
);
```

This is roughly:

```js
{
  value: "Vikash",
  writable: false,
  enumerable: false,
  configurable: false
}
```

This difference is very important.

---

# 10. Accessor Properties

Not every property stores a value directly.

A property may instead use:

- getter
- setter

Example:

```js
const user = {
  firstName: "Vikash",
  lastName: "Kumar",

  get fullName() {
    return (
      this.firstName +
      " " +
      this.lastName
    );
  },

  set fullName(value) {
    const [first, last] =
      value.split(" ");

    this.firstName = first;
    this.lastName = last;
  },
};
```

Reading:

```js
console.log(user.fullName);
```

calls the getter.

Writing:

```js
user.fullName =
  "Rahul Sharma";
```

calls the setter.

---

# 11. Accessor Descriptor

Accessor properties use a different descriptor shape.

```js
const descriptor =
  Object.getOwnPropertyDescriptor(
    user,
    "fullName"
  );

console.log(descriptor);
```

Conceptually:

```js
{
  get: function,
  set: function,
  enumerable: true,
  configurable: true
}
```

Notice there is no normal `value` or `writable`.

---

# 12. Data vs Accessor Property

Data property:

```js
{
  value,
  writable,
  enumerable,
  configurable
}
```

Accessor property:

```js
{
  get,
  set,
  enumerable,
  configurable
}
```

A property descriptor cannot meaningfully be both a data descriptor and an accessor descriptor at the same time.

---

# 13. defineProperties()

You can define multiple properties at once:

```js
const user = {};

Object.defineProperties(
  user,
  {
    id: {
      value: 101,
      enumerable: true,
    },

    name: {
      value: "Vikash",
      writable: true,
      enumerable: true,
    },
  }
);
```

This is useful when you need precise control over several properties.

---

# 14. Object.keys vs Object.getOwnPropertyNames

Consider:

```js
const user = {
  name: "Vikash",
};

Object.defineProperty(
  user,
  "secret",
  {
    value: 123,
    enumerable: false,
  }
);
```

```js
console.log(
  Object.keys(user)
);
```

returns only enumerable own string keys:

```js
["name"]
```

But:

```js
console.log(
  Object.getOwnPropertyNames(user)
);
```

includes non-enumerable own string properties:

```js
["name", "secret"]
```

---

# 15. Reflect.ownKeys()

If an object also contains symbol keys:

```js
const id = Symbol("id");

const user = {
  name: "Vikash",
  [id]: 101,
};
```

Then:

```js
Reflect.ownKeys(user);
```

returns all own property keys:

- strings
- symbols
- enumerable
- non-enumerable

This makes it one of the most complete own-key inspection tools.

---

# 16. Property Lookup Preview

When you access:

```js
user.name
```

JavaScript first checks whether `name` exists directly on `user`.

Conceptually:

```text
user.name
   ↓
Does user have own "name"?
   ↓
yes → return it

no
↓
look at prototype
```

That second step is the beginning of the prototype system, which starts in Lesson 51.

---

# 17. hasOwn()

Modern JavaScript provides:

```js
Object.hasOwn(
  user,
  "name"
);
```

This is preferred over calling:

```js
user.hasOwnProperty("name")
```

in many cases because an object might not inherit `hasOwnProperty`, or might shadow that name.

Example:

```js
const user = {
  hasOwnProperty: "not a function",
  name: "Vikash",
};

console.log(
  Object.hasOwn(
    user,
    "name"
  )
);
```

Output:

```text
true
```

---

# 18. in Operator

`in` asks whether a property exists either:

- directly on the object
- or somewhere in its prototype chain

Example:

```js
"name" in user;
```

This is different from:

```js
Object.hasOwn(user, "name");
```

Later, this distinction becomes important with inherited properties.

---

# 19. Object.freeze()

```js
const user = {
  name: "Vikash",
};

Object.freeze(user);
```

At a high level, freezing an object prevents adding/removing properties and makes existing data properties non-writable/non-configurable.

But remember:

> `Object.freeze()` is shallow.

Example:

```js
const user = {
  profile: {
    city: "Bengaluru",
  },
};

Object.freeze(user);

user.profile.city = "Pune";
```

The nested object can still be mutated unless it is frozen separately.

---

# 20. Object.seal()

```js
Object.seal(user);
```

A sealed object cannot have properties added or deleted.

Existing writable properties may still be changed.

Simplified comparison:

```text
seal
→ structure locked
→ existing writable values may change

freeze
→ structure locked
→ existing data values also made non-writable
```

Both are shallow.

---

# 21. Object.preventExtensions()

```js
Object.preventExtensions(user);
```

This prevents adding new properties.

Existing properties may still be modified or deleted according to their descriptors.

Think of these three as increasing restrictions:

```text
preventExtensions
      ↓
cannot add new properties

seal
      ↓
cannot add/delete properties

freeze
      ↓
cannot add/delete
+
data values become non-writable
```

---

# 22. React Connection

Property descriptors are not something you use every day in React components, but understanding them helps with:

- library internals
- framework behavior
- immutability concepts
- debugging objects
- proxies and meta-programming
- understanding why some properties behave differently

React code more commonly uses ordinary objects:

```js
const nextState = {
  ...state,
  name: "Vikash",
};
```

But underneath, JavaScript's object/property system still follows these core rules.

---

# 23. Interview Questions

### What is a property descriptor?

A property descriptor is metadata describing how a property behaves, including attributes such as `value`, `writable`, `enumerable`, and `configurable`, or accessor functions `get` and `set`.

### What is the difference between enumerable and configurable?

`enumerable` affects whether a property appears in enumeration operations. `configurable` controls whether the property can be deleted or substantially redefined.

### What is the difference between Object.hasOwn() and the in operator?

`Object.hasOwn()` checks only own properties. `in` also checks inherited properties through the prototype chain.

---

# Key Takeaways

- Object properties contain metadata called property descriptors.
- Data descriptors use `value` and `writable`.
- Accessor descriptors use `get` and `set`.
- `enumerable` controls many enumeration behaviors.
- `configurable` controls deletion and most reconfiguration.
- Object literal properties normally have all three flags set to `true`.
- `Object.defineProperty()` omitted flags default to `false`.
- Own properties and inherited properties are different concepts.
- `Object.hasOwn()` checks only own properties.
- `in` also checks the prototype chain.
- Property lookup naturally leads into the prototype system.
