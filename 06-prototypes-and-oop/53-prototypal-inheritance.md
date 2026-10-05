# Lesson 53 — Prototypal Inheritance

In JavaScript, inheritance is fundamentally **prototype-based**.

That means one object can inherit behavior from another object through the prototype chain.

This is different from languages where inheritance is built primarily around classes.

The central idea is:

> An object can delegate property lookup to another object through its prototype.

---

## 1. Start with a Simple Parent Object

```js
const person = {
  greet() {
    console.log(
      `Hello, I am ${this.name}`
    );
  },
};
```

Now create another object using `person` as its prototype:

```js
const developer =
  Object.create(person);

developer.name = "Vikash";
developer.role = "Developer";
```

Now:

```js
developer.greet();
```

Output:

```text
Hello, I am Vikash
```

Why?

```text
developer
├── name
├── role
│
└── [[Prototype]]
       ↓
     person
       └── greet()
```

The method is inherited through the prototype relationship.

---

# 2. Inheritance Does Not Copy Methods

A common misunderstanding is:

> JavaScript copies `greet` from `person` into `developer`.

It does not.

Instead:

```text
developer.greet
      ↓
not found on developer
      ↓
look at prototype
      ↓
person.greet found
```

This is **delegation**, not duplication.

---

# 3. this Still Depends on the Call Site

```js
developer.greet();
```

The method is found on `person`, but the receiver is:

```text
developer
```

Therefore:

```text
this = developer
```

This is a very important connection between prototype lookup and the `this` lessons.

Remember:

```text
Where method is found
≠
What this becomes
```

---

# 4. Multi-Level Prototypal Inheritance

```js
const livingThing = {
  breathe() {
    console.log("breathing");
  },
};

const person =
  Object.create(livingThing);

person.walk = function () {
  console.log("walking");
};

const developer =
  Object.create(person);

developer.code = function () {
  console.log("coding");
};
```

Chain:

```text
developer
│
├── code()
│
└── [[Prototype]]
       ↓
     person
     ├── walk()
     │
     └── [[Prototype]]
            ↓
        livingThing
        ├── breathe()
        │
        └── [[Prototype]]
               ↓
         Object.prototype
               ↓
              null
```

Now `developer` can access:

```js
developer.code();
developer.walk();
developer.breathe();
```

---

# 5. Property Shadowing

Suppose:

```js
const person = {
  role: "Person",
};

const developer =
  Object.create(person);

developer.role =
  "Developer";
```

Now:

```js
console.log(
  developer.role
);
```

Output:

```text
Developer
```

The own property on `developer` shadows the inherited property.

---

# 6. Updating the Parent Prototype Object

```js
const person = {
  role: "Person",
};

const developer =
  Object.create(person);

console.log(
  developer.role
);
```

Output:

```text
Person
```

Now:

```js
person.role =
  "Human";
```

Then:

```js
console.log(
  developer.role
);
```

Output:

```text
Human
```

Why?

Because `developer` did not receive a copied value.

It still delegates lookup to the same prototype object.

---

# 7. Constructor Functions and Prototypal Inheritance

Prototypal inheritance is also the mechanism behind constructor functions.

Base constructor:

```js
function Person(name) {
  this.name = name;
}

Person.prototype.greet =
  function () {
    console.log(
      `Hello ${this.name}`
    );
  };
```

Child constructor:

```js
function Developer(
  name,
  skill
) {
  Person.call(
    this,
    name
  );

  this.skill = skill;
}
```

Here:

```js
Person.call(
  this,
  name
);
```

reuses the parent constructor logic.

But this alone does **not** establish prototype inheritance.

---

# 8. Linking the Child Prototype

To inherit methods from `Person.prototype`:

```js
Developer.prototype =
  Object.create(
    Person.prototype
  );
```

Now:

```text
Developer.prototype
      ↓
Person.prototype
```

This creates the inheritance relationship.

---

# 9. Restore constructor

After:

```js
Developer.prototype =
  Object.create(
    Person.prototype
  );
```

the `constructor` property now comes through the prototype chain from `Person.prototype`.

So we often restore it:

```js
Developer.prototype.constructor =
  Developer;
```

Now create an instance:

```js
const dev =
  new Developer(
    "Vikash",
    "JavaScript"
  );
```

---

# 10. Complete Prototype Chain

For `dev`:

```text
dev
│
├── name
├── skill
│
└── [[Prototype]]
       ↓
Developer.prototype
       │
       └── [[Prototype]]
              ↓
        Person.prototype
              │
              ├── greet()
              │
              └── [[Prototype]]
                     ↓
               Object.prototype
                     ↓
                    null
```

So:

```js
dev.greet();
```

works through the chain.

---

# 11. Add Child-Specific Methods

```js
Developer.prototype.code =
  function () {
    console.log(
      `${this.name} is coding in ${this.skill}`
    );
  };
```

Now:

```js
dev.code();
dev.greet();
```

The object gets behavior from both prototype levels.

---

# 12. Parent Constructor Reuse vs Prototype Inheritance

These are two separate responsibilities.

### Parent constructor reuse

```js
Person.call(
  this,
  name
);
```

This initializes instance data.

### Prototype inheritance

```js
Developer.prototype =
  Object.create(
    Person.prototype
  );
```

This makes parent methods available through the prototype chain.

Do not confuse them.

---

# 13. instanceof with Inheritance

```js
console.log(
  dev instanceof Developer
);
```

Output:

```text
true
```

Also:

```js
console.log(
  dev instanceof Person
);
```

Output:

```text
true
```

Why?

Because both:

```text
Developer.prototype
Person.prototype
```

appear in `dev`'s prototype chain.

---

# 14. Object.create() vs Constructor Inheritance

Direct object inheritance:

```js
const child =
  Object.create(parent);
```

Constructor inheritance:

```js
Child.prototype =
  Object.create(
    Parent.prototype
  );
```

Both use the same core idea:

> create an object whose internal prototype points to another object.

---

# 15. Why Prototypal Inheritance Matters

It enables:

- shared methods
- code reuse
- delegation
- inheritance chains
- memory-efficient shared behavior
- JavaScript classes underneath the syntax

Even modern `class` syntax uses prototypes internally.

---

# 16. Composition vs Inheritance

Inheritance is useful, but it is not always the best design.

Deep chains can become difficult to reason about:

```text
A
↓
B
↓
C
↓
D
↓
E
```

In many application designs, composition can be simpler.

Example:

```js
const canCode = {
  code() {},
};

const canDebug = {
  debug() {},
};

const developer = {
  ...canCode,
  ...canDebug,
};
```

This copies behavior rather than using prototype delegation, so it is a different mechanism—but often a simpler design choice.

---

# 17. ES6 Class Preview

This older pattern:

```js
function Developer(
  name,
  skill
) {
  Person.call(
    this,
    name
  );

  this.skill = skill;
}

Developer.prototype =
  Object.create(
    Person.prototype
  );

Developer.prototype.constructor =
  Developer;
```

can be expressed much more cleanly using:

```js
class Developer
  extends Person {
  // ...
}
```

That is why classes were introduced as more readable syntax.

But remember:

> Classes do not replace the prototype system.

They are built on top of it.

---

# Interview Questions

### What is prototypal inheritance?

Prototypal inheritance is JavaScript's mechanism where objects inherit properties and methods through prototype links.

### Is inherited behavior copied into the child object?

No. Property lookup delegates through the prototype chain.

### What is the difference between calling Parent.call(this) and linking Child.prototype?

`Parent.call(this)` reuses constructor initialization logic. Linking `Child.prototype` to `Parent.prototype` establishes method inheritance.

---

# Key Takeaways

- JavaScript inheritance is prototype-based.
- Inherited methods are usually shared, not copied.
- Property lookup delegates through prototype links.
- `Object.create()` is a direct way to create prototype relationships.
- Constructor inheritance traditionally combines parent-constructor reuse and prototype linking.
- `instanceof` reflects prototype-chain relationships.
- Deep inheritance chains can become difficult to maintain.
- ES6 classes provide cleaner syntax over the same prototype system.
