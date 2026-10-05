# Lesson 49 — Constructor Functions and the `new` Keyword

So far, you have learned these `this` binding situations:

```text
fn()
→ default binding

obj.fn()
→ implicit binding

fn.call(obj)
fn.apply(obj)
fn.bind(obj)
→ explicit binding
```

Now we add another extremely important case:

```js
new Fn();
```

When a constructible function is invoked with `new`, JavaScript performs **constructor invocation** and provides a newly created object as `this`.

This is commonly called **new binding**.

---

# 1. What Is a Constructor Function?

Before ES6 classes, constructor functions were a common way to create multiple objects with the same structure.

Example:

```js
function User(name, role) {
  this.name = name;
  this.role = role;
}

const user1 =
  new User(
    "Vikash",
    "Developer"
  );

const user2 =
  new User(
    "Rahul",
    "Designer"
  );
```

Results conceptually:

```text
user1
{
  name: "Vikash",
  role: "Developer"
}

user2
{
  name: "Rahul",
  role: "Designer"
}
```

The same constructor function creates multiple independent objects.

---

# 2. Why Is It Called a Constructor Function?

There is no separate `constructor function` declaration syntax like:

```js
constructor function User() {}
```

Instead, a normal constructible function becomes a constructor **when it is invoked with `new`**.

Conventionally, constructor function names begin with a capital letter:

```js
function User() {}
function Product() {}
function Order() {}
```

The capitalization is a convention that tells developers:

> This function is intended to be called with `new`.

---

# 3. The Most Important Question — What Does new Do?

Consider:

```js
function User(name) {
  this.name = name;
}

const user =
  new User("Vikash");
```

A useful conceptual model is that `new` performs roughly these steps:

```text
1. Create a new empty object

2. Link that object's prototype
   to User.prototype

3. Call User with
   this = new object

4. If the constructor does not
   explicitly return another object,
   return the new object
```

Simplified visualization:

```text
new User("Vikash")
        ↓
create {}
        ↓
link prototype
        ↓
User executes
this = new object
        ↓
this.name = "Vikash"
        ↓
return new object
        ↓
user
```

This is the core of constructor invocation.

---

# 4. Understanding this with new

```js
function User(name) {
  console.log(this);

  this.name = name;
}

const user =
  new User("Vikash");
```

Inside `User`, `this` refers to the newly created instance.

Conceptually:

```text
new User("Vikash")
        ↓
new object created
        ↓
this → new object
        ↓
this.name = "Vikash"
```

So after construction:

```js
console.log(user.name);
```

Output:

```text
Vikash
```

---

# 5. Compare Normal Call vs new Call

Same function:

```js
function User(name) {
  this.name = name;
}
```

### Plain call

```js
"use strict";

User("Vikash");
```

This is:

```text
plain regular-function call
↓
default binding
↓
this = undefined
```

Trying:

```js
this.name = name;
```

causes an error because `this` is `undefined`.

### Constructor call

```js
const user =
  new User("Vikash");
```

Now:

```text
new binding
↓
this = newly created object
```

Same function.

Different invocation.

This perfectly demonstrates why `this` is about **how a regular function is invoked**.

---

# 6. Does the Constructor Need to Write return?

Usually, no.

```js
function User(name) {
  this.name = name;
}

const user =
  new User("Vikash");
```

Even though there is no:

```js
return this;
```

JavaScript returns the newly created object automatically.

So:

```js
console.log(user);
```

contains:

```text
{name: "Vikash"}
```

---

# 7. What If the Constructor Returns a Primitive?

```js
function User(name) {
  this.name = name;

  return 100;
}

const user =
  new User("Vikash");

console.log(user);
```

The primitive return value is ignored.

The newly constructed object is returned.

Conceptually:

```text
constructor returns primitive
        ↓
primitive ignored
        ↓
new instance returned
```

---

# 8. What If the Constructor Returns an Object?

This case is different.

```js
function User(name) {
  this.name = name;

  return {
    name: "Rahul",
  };
}

const user =
  new User("Vikash");

console.log(user.name);
```

Output:

```text
Rahul
```

Why?

If a constructor explicitly returns an object, that object becomes the result of the `new` expression.

Mental model:

```text
Constructor returns:

primitive
→ ignore it
→ return created instance

object/function
→ use explicitly returned object
```

This is a common interview question.

---

# 9. Constructor Functions and Methods

You could write:

```js
function User(name) {
  this.name = name;

  this.showName = function () {
    console.log(this.name);
  };
}

const user1 =
  new User("Vikash");

const user2 =
  new User("Rahul");
```

But each instance gets its own `showName` function object.

```js
console.log(
  user1.showName ===
  user2.showName
);
```

Output:

```text
false
```

Conceptually:

```text
user1
├── name
└── showName function #1

user2
├── name
└── showName function #2
```

This can be unnecessary when all instances should share the same behavior.

---

# 10. Shared Methods with prototype

A more traditional constructor pattern is:

```js
function User(name) {
  this.name = name;
}

User.prototype.showName =
  function () {
    console.log(this.name);
  };

const user1 =
  new User("Vikash");

const user2 =
  new User("Rahul");
```

Now:

```js
console.log(
  user1.showName ===
  user2.showName
);
```

Output:

```text
true
```

Both instances find the same method through the prototype chain.

Conceptually:

```text
user1 ──┐
        │
        ├──→ User.prototype
        │        │
user2 ──┘        └── showName()
```

We will study prototypes deeply in the next section.

For now, remember that `new` links the new object's prototype to the constructor's `prototype` object.

---

# 11. Why Does showName's this Still Work?

```js
user1.showName();
```

Although `showName` is found through the prototype chain, the invocation receiver is still:

```text
user1
```

Therefore:

```text
this = user1
```

Likewise:

```js
user2.showName();
```

gives:

```text
this = user2
```

The location where the method is found does not determine `this`.

The call site still matters.

---

# 12. new with Arguments

```js
function Product(
  name,
  price
) {
  this.name = name;
  this.price = price;
}

const laptop =
  new Product(
    "Laptop",
    50000
  );
```

During execution:

```text
new Product("Laptop", 50000)
          ↓
new object
          ↓
this = new object
          ↓
name = "Laptop"
price = 50000
          ↓
instance returned
```

---

# 13. Constructor Functions vs Factory Functions

These concepts are related but different.

### Constructor

```js
function User(name) {
  this.name = name;
}

const user =
  new User("Vikash");
```

Uses:

```text
new
+
this
+
prototype relationship
```

### Factory

```js
function createUser(name) {
  return {
    name,
  };
}

const user =
  createUser("Vikash");
```

The factory explicitly creates/returns an object and does not require `new`.

Mental model:

```text
Constructor
new User()
→ new controls object creation

Factory
createUser()
→ function explicitly returns object
```

Both are valid patterns.

---

# 14. Arrow Functions Cannot Be Constructors

This does not work:

```js
const User = (name) => {
  this.name = name;
};

const user =
  new User("Vikash");
```

Result:

```text
TypeError:
User is not a constructor
```

Arrow functions are not constructible.

They do not have the constructor behavior needed by `new`.

This connects directly to Lesson 44.

---

# 15. new and Explicit Binding Priority

Recall:

```js
function User(name) {
  this.name = name;
}

const obj = {
  name: "Original",
};

const BoundUser =
  User.bind(obj);
```

Normal bound call:

```js
BoundUser("Vikash");
```

uses the bound `obj` as `this`.

But:

```js
const user =
  new BoundUser("Vikash");
```

constructor invocation creates a new instance.

The bound `thisArg` is ignored for this constructor call.

```text
User.bind(obj)
      ↓
bound function

new BoundUser()
      ↓
new constructor invocation
      ↓
new object becomes this
```

This is the important special case introduced in Lesson 48.

---

# 16. Complete this Binding Priority

For the major regular-function cases we have studied, use this decision process:

```text
Is the function invoked with new?
        │
        ├── YES
        │    ↓
        │  new binding
        │
        └── NO
             ↓
Is it explicitly bound?
call / apply / bound function
        │
        ├── YES
        │    ↓
        │ explicit binding
        │
        └── NO
             ↓
Is it called as object.method()?
        │
        ├── YES
        │    ↓
        │ implicit binding
        │
        └── NO
             ↓
        default binding
```

Useful shorthand:

```text
new
 >
explicit
 >
implicit
 >
default
```

Important nuance:

A function produced by `bind()` keeps its bound `this` for normal calls, while constructible bound functions invoked with `new` use constructor semantics.

---

# 17. But Check Arrow Functions First

Before applying the previous priority list, ask:

```text
Is this an arrow function?
```

If yes:

```text
arrow
↓
no own this
↓
lexical this
```

Do not blindly apply regular-function dynamic binding rules to arrows.

So the full mental process is:

```text
Function uses this
      ↓
Regular or arrow?
      ↓
Arrow
→ lexical this

Regular
→ inspect invocation
      ↓
new?
explicit?
implicit?
default?
```

This is the mental model you should carry into interviews.

---

# 18. ES6 Classes and new

Modern JavaScript often uses:

```js
class User {
  constructor(name) {
    this.name = name;
  }

  showName() {
    console.log(this.name);
  }
}

const user =
  new User("Vikash");
```

The syntax is cleaner, but `new` still plays the constructor role.

Conceptually:

```text
new User()
    ↓
new instance
    ↓
constructor runs
    ↓
this = instance
```

Classes also use prototypes for instance methods.

We will study classes more deeply later.

---

# 19. instanceof

Given:

```js
function User(name) {
  this.name = name;
}

const user =
  new User("Vikash");
```

We can check:

```js
console.log(
  user instanceof User
);
```

Output:

```text
true
```

At a high level, `instanceof` checks whether `User.prototype` appears in `user`'s prototype chain.

This will make much more sense after the prototype lessons.

---

# 20. Common Mistakes

## Mistake 1 — Forgetting new

```js
"use strict";

function User(name) {
  this.name = name;
}

const user =
  User("Vikash");
```

This is a plain call, not constructor invocation.

---

## Mistake 2 — Using Arrow as Constructor

```js
const User = () => {};

new User();
```

Arrow functions cannot be constructors.

---

## Mistake 3 — Thinking this Is the Constructor Function

Inside:

```js
function User() {
  console.log(this);
}
```

when called with:

```js
new User();
```

`this` is the newly created instance, not the `User` function itself.

---

## Mistake 4 — Putting Every Method Inside Constructor

This:

```js
function User(name) {
  this.name = name;

  this.show = function () {};
}
```

creates a separate function for every instance.

Shared behavior can often live on the prototype.

---

# 21. Interview Output Question

```js
function Person(name) {
  this.name = name;

  return 100;
}

const person =
  new Person("Vikash");

console.log(person.name);
```

Output:

```text
Vikash
```

Why?

The explicitly returned primitive is ignored during construction.

---

# 22. Another Interview Question

```js
function Person(name) {
  this.name = name;

  return {
    name: "Rahul",
  };
}

const person =
  new Person("Vikash");

console.log(person.name);
```

Output:

```text
Rahul
```

The explicitly returned object replaces the normally returned instance.

---

# 23. Section 5 Complete Mental Model

You can now connect the whole section:

```text
REGULAR FUNCTION

fn()
↓
default binding

obj.fn()
↓
implicit binding

fn.call(obj)
fn.apply(obj)
↓
explicit binding + execute now

fn.bind(obj)
↓
new bound function
↓
execute later

new Fn()
↓
constructor invocation
↓
new object becomes this
```

And separately:

```text
ARROW FUNCTION

no own this
↓
lexical this
↓
call/apply/bind cannot
replace its this
↓
cannot be used with new
```

This is the foundation for understanding `this` correctly.

---

# Interview Answers

**What happens when we use `new` with a constructor function?**

JavaScript creates a new object, links it to the constructor's prototype, invokes the constructor with the new object as `this`, and normally returns that object.

**What is new binding?**

It is the `this` binding created during constructor invocation, where `this` refers to the newly created instance.

**What happens if a constructor returns a primitive?**

The primitive is ignored and the newly created instance is returned.

**What happens if a constructor explicitly returns an object?**

That returned object becomes the result of the `new` expression.

**Can an arrow function be used with `new`?**

No. Arrow functions are not constructible.

---

# Key Takeaways

- Constructor functions are ordinary constructible functions intended to be called with `new`.
- `new` creates a new object.
- The new object's prototype is linked to the constructor's `prototype`.
- The constructor executes with the new object as `this`.
- The instance is normally returned automatically.
- Primitive constructor return values are normally ignored.
- Explicitly returned objects can replace the created instance.
- Prototype methods can be shared across instances.
- Arrow functions cannot be constructors.
- For the major regular-function cases, remember: `new > explicit > implicit > default`.
- Always separate arrow-function lexical `this` from regular-function binding rules.
