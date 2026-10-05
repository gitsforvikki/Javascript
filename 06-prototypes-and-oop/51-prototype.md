# Lesson 51 — Prototype

JavaScript uses **prototypes** as one of its core object-sharing and inheritance mechanisms.

The key idea is:

> An object can have another object as its prototype.

When a property is not found directly on an object, JavaScript may look for it on that object's prototype.

This is the foundation of prototypal inheritance.

---

## 1. Basic Mental Model

Suppose:

```js
const user = {
  name: "Vikash",
};
```

The object has its own property:

```text
user
└── name: "Vikash"
```

But `user` also normally has an internal prototype reference.

Conceptually:

```text
user
│
├── name: "Vikash"
│
└── [[Prototype]]
       ↓
   Object.prototype
```

The specification-level internal slot is commonly written:

```text
[[Prototype]]
```

You cannot access that internal slot directly as a normal property, but JavaScript provides APIs to inspect or change it.

---

# 2. Object.getPrototypeOf()

Use:

```js
Object.getPrototypeOf(user);
```

Example:

```js
const user = {
  name: "Vikash",
};

console.log(
  Object.getPrototypeOf(user)
    === Object.prototype
);
```

Output:

```text
true
```

For a normal object literal, the prototype is usually `Object.prototype`.

---

# 3. Property Lookup Through the Prototype

Consider:

```js
const user = {
  name: "Vikash",
};

console.log(
  user.toString
);
```

You did not define `toString` on `user`.

So why is it available?

JavaScript performs property lookup:

```text
user.toString
   ↓
Does user have own "toString"?
   ↓
no
   ↓
look at Object.prototype
   ↓
found
```

That is prototype-based property lookup.

---

# 4. Own vs Inherited Property

```js
const user = {
  name: "Vikash",
};
```

Check:

```js
Object.hasOwn(
  user,
  "name"
);
```

Output:

```text
true
```

But:

```js
Object.hasOwn(
  user,
  "toString"
);
```

Output:

```text
false
```

Yet:

```js
"toString" in user;
```

Output:

```text
true
```

Why?

`toString` is inherited from the prototype.

---

# 5. Creating an Object with a Specific Prototype

Use:

```js
Object.create(prototypeObject);
```

Example:

```js
const person = {
  greet() {
    console.log(
      `Hello, ${this.name}`
    );
  },
};

const user =
  Object.create(person);

user.name = "Vikash";

user.greet();
```

Output:

```text
Hello, Vikash
```

Why?

```text
user
├── own name: "Vikash"
└── [[Prototype]]
       ↓
     person
       └── greet()
```

When `user.greet` is requested:

```text
user
↓
greet own property?
no
↓
prototype person
↓
greet found
```

---

# 6. Important this Connection

In:

```js
user.greet();
```

the method may be **found** on `person`, but the call receiver is still `user`.

Therefore inside `greet`:

```text
this = user
```

This connects directly to Section 5.

Very important:

> Where a method is found and what `this` becomes are separate questions.

---

# 7. Prototype Is Not a Copy

When `user` inherits from `person`, JavaScript does not copy all of `person`'s properties into `user`.

Instead:

```text
user
  ↓
prototype reference
  ↓
person
```

This means multiple objects can share behavior through the same prototype object.

---

# 8. Shared Behavior

```js
const personMethods = {
  greet() {
    console.log(
      `Hello ${this.name}`
    );
  },
};

const user1 =
  Object.create(
    personMethods
  );

user1.name = "Vikash";

const user2 =
  Object.create(
    personMethods
  );

user2.name = "Rahul";
```

Both objects share one `greet` function:

```js
console.log(
  user1.greet ===
  user2.greet
);
```

Output:

```text
true
```

This is memory-efficient compared with creating a new identical method on every object.

---

# 9. Property Shadowing

Suppose the prototype has:

```js
const person = {
  role: "Person",
};
```

Create:

```js
const user =
  Object.create(person);

user.role =
  "Developer";
```

Now:

```js
console.log(user.role);
```

Output:

```text
Developer
```

Why?

JavaScript checks the object first.

```text
user.role
↓
own property exists?
yes
↓
return "Developer"
```

The prototype's `role` still exists, but it is **shadowed** by the own property.

---

# 10. Deleting a Shadowing Property

Continuing:

```js
delete user.role;
```

Now:

```js
console.log(user.role);
```

Output:

```text
Person
```

Why?

After deleting the own property:

```text
user.role
↓
own role?
no
↓
prototype role?
yes
↓
"Person"
```

This demonstrates prototype lookup clearly.

---

# 11. Object.setPrototypeOf()

You can change an object's prototype:

```js
const animal = {
  move() {
    console.log("moving");
  },
};

const dog = {};

Object.setPrototypeOf(
  dog,
  animal
);

dog.move();
```

Output:

```text
moving
```

However, changing prototypes dynamically can hurt engine optimization and is generally not a preferred pattern in performance-sensitive code.

Usually define the desired prototype relationship when creating the object.

---

# 12. __proto__

You may see:

```js
user.__proto__
```

Historically, `__proto__` has been widely used to inspect or set an object's prototype.

But modern code should generally prefer:

```js
Object.getPrototypeOf(obj)
Object.setPrototypeOf(obj, proto)
```

Do not confuse `__proto__` with a constructor function's `prototype` property.

That distinction is crucial.

---

# 13. [[Prototype]] vs prototype Property

These are often confused.

### Every ordinary object can have an internal prototype

Conceptually:

```text
object.[[Prototype]]
```

You inspect it using:

```js
Object.getPrototypeOf(object)
```

### Functions may have a normal property named prototype

Example:

```js
function User() {}

console.log(
  User.prototype
);
```

This `prototype` property is used when `User` is invoked with `new`.

These are related but not the same thing.

---

# 14. Constructor Function Connection

Recall Lesson 49:

```js
function User(name) {
  this.name = name;
}

User.prototype.greet =
  function () {
    console.log(
      `Hello ${this.name}`
    );
  };

const user =
  new User("Vikash");
```

During construction, the new object's internal prototype is linked to:

```js
User.prototype
```

Conceptually:

```text
user
├── name: "Vikash"
└── [[Prototype]]
       ↓
   User.prototype
       └── greet()
```

Therefore:

```js
user.greet();
```

works even though `greet` is not an own property of `user`.

---

# 15. Verify Constructor Prototype Relationship

```js
console.log(
  Object.getPrototypeOf(user)
    === User.prototype
);
```

Output:

```text
true
```

This is one of the most important relationships in JavaScript OOP.

---

# 16. prototype.constructor

Normally:

```js
function User() {}

console.log(
  User.prototype.constructor
    === User
);
```

Output:

```text
true
```

By default, the object stored in `User.prototype` has a `constructor` property pointing back to `User`.

Do not treat this property as magical identity storage; it is just part of the normal prototype object setup and can be changed.

---

# 17. Replacing the Entire prototype Object

Consider:

```js
function User() {}

User.prototype = {
  greet() {
    console.log("hello");
  },
};
```

Now:

```js
console.log(
  User.prototype.constructor
    === User
);
```

is no longer guaranteed to be `true`, because you replaced the original prototype object.

If you care about the `constructor` property, you may restore it explicitly.

This is a common interview nuance.

---

# 18. Object.create(null)

You can create an object with no prototype:

```js
const dictionary =
  Object.create(null);
```

Then:

```js
console.log(
  Object.getPrototypeOf(
    dictionary
  )
);
```

Output:

```text
null
```

Such an object does not inherit methods like:

```text
toString
hasOwnProperty
```

This can be useful for pure dictionary-like objects.

---

# 19. Why Prototypes Exist

Without shared prototypes, every instance might need duplicate method functions.

Example:

```js
function createUser(name) {
  return {
    name,

    greet() {
      console.log(
        `Hello ${name}`
      );
    },
  };
}
```

Every call creates another `greet` function.

Prototype sharing allows:

```text
many objects
   ↓
share behavior
   ↓
one prototype object
```

This is a core efficiency and inheritance mechanism in JavaScript.

---

# 20. React Connection

Modern React code rarely uses manual prototype manipulation directly.

However:

- JavaScript classes are built on prototypes
- many libraries use prototype-based behavior
- class methods live on prototypes
- understanding prototypes helps explain `instanceof`
- prototype pollution is a security topic
- debugging objects often requires understanding inherited properties

So prototypes remain fundamental JavaScript knowledge even if React function components do not expose them directly.

---

# Interview Questions

### What is a prototype in JavaScript?

A prototype is an object that another object can reference internally for inherited property lookup.

### What happens when a property is not found on an object?

JavaScript checks the object's prototype, then continues through the prototype chain until the property is found or the chain ends at `null`.

### What is the difference between `User.prototype` and an instance's internal `[[Prototype]]`?

`User.prototype` is a normal property on the constructor function. When `new User()` is used, the new instance's internal `[[Prototype]]` is linked to that object.

---

# Key Takeaways

- Objects can have another object as their prototype.
- The internal relationship is represented conceptually as `[[Prototype]]`.
- `Object.getPrototypeOf()` inspects an object's prototype.
- Missing properties may be resolved through the prototype.
- Inherited properties are not own properties.
- `Object.create(proto)` creates an object with a chosen prototype.
- Prototype inheritance is reference-based, not copying.
- Own properties can shadow inherited properties.
- `User.prototype` and an object's internal prototype are related but not identical concepts.
- `new User()` links the instance's prototype to `User.prototype`.
