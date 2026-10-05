# Lesson 52 — Prototype Chain

Lesson 51 introduced the idea that one object can have another object as its prototype.

But that prototype can itself have another prototype.

This creates a **prototype chain**.

The prototype chain is the sequence of prototype links JavaScript follows when searching for properties.

---

## 1. Basic Chain

Consider:

```js
const user = {
  name: "Vikash",
};
```

For a normal object literal, the chain is roughly:

```text
user
  ↓ [[Prototype]]
Object.prototype
  ↓ [[Prototype]]
null
```

`null` marks the end of the chain.

---

# 2. Property Lookup Through the Chain

Suppose:

```js
console.log(
  user.toString()
);
```

JavaScript searches:

```text
1. user
   has own toString?
   no

2. Object.prototype
   has toString?
   yes

3. use it
```

If the property is not found anywhere before `null`, the result of normal property access is `undefined`.

---

# 3. Custom Prototype Chain

```js
const livingThing = {
  alive: true,
};

const person =
  Object.create(
    livingThing
  );

person.walk = function () {
  console.log("walking");
};

const developer =
  Object.create(person);

developer.name = "Vikash";
```

The chain:

```text
developer
├── name
│
└── [[Prototype]]
       ↓
     person
     ├── walk()
     │
     └── [[Prototype]]
            ↓
        livingThing
        ├── alive
        │
        └── [[Prototype]]
               ↓
         Object.prototype
               ↓
             null
```

---

# 4. Lookup Example

```js
console.log(
  developer.alive
);
```

Search:

```text
developer
↓
alive?
no

person
↓
alive?
no

livingThing
↓
alive?
yes

return true
```

That is prototype-chain lookup.

---

# 5. Closest Property Wins

Suppose:

```js
const animal = {
  type: "animal",
};

const dog =
  Object.create(animal);

dog.type = "dog";
```

Now:

```js
console.log(dog.type);
```

Output:

```text
dog
```

JavaScript stops at the first match.

```text
dog.type
↓
own type found
↓
stop lookup
```

This is property shadowing.

---

# 6. Delete Reveals Inherited Property

```js
delete dog.type;

console.log(dog.type);
```

Output:

```text
animal
```

Now:

```text
dog
↓
own type?
no
↓
animal
↓
type found
```

The inherited property was never deleted.

Only the shadowing own property was removed.

---

# 7. Prototype Chain with Constructor Functions

Recall:

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

The chain:

```text
user
│
├── name: "Vikash"
│
└── [[Prototype]]
       ↓
   User.prototype
       │
       ├── greet()
       ├── constructor
       │
       └── [[Prototype]]
              ↓
        Object.prototype
              ↓
             null
```

This is the standard constructor-function prototype relationship.

---

# 8. Where Does Object.prototype Come From?

`User.prototype` is itself normally an ordinary object.

Therefore:

```js
Object.getPrototypeOf(
  User.prototype
) === Object.prototype
```

is typically:

```text
true
```

So instance lookup can continue beyond `User.prototype`.

Example:

```js
user.toString();
```

Searches:

```text
user
↓
User.prototype
↓
Object.prototype
↓
toString found
```

---

# 9. null Ends the Chain

```js
Object.getPrototypeOf(
  Object.prototype
);
```

returns:

```text
null
```

So:

```text
Object.prototype
      ↓
    null
```

At `null`, lookup stops.

There is no prototype beyond that.

---

# 10. Array Prototype Chain

Arrays are objects too.

```js
const numbers = [
  10,
  20,
  30,
];
```

A simplified chain is:

```text
numbers
   ↓
Array.prototype
   ↓
Object.prototype
   ↓
null
```

That is why arrays can use:

```js
numbers.map(...)
numbers.filter(...)
numbers.push(...)
```

Those methods are typically found on `Array.prototype`.

---

# 11. Verify Array Chain

```js
console.log(
  Object.getPrototypeOf(
    numbers
  ) === Array.prototype
);
```

Output:

```text
true
```

And:

```js
console.log(
  Object.getPrototypeOf(
    Array.prototype
  ) === Object.prototype
);
```

Output:

```text
true
```

This is an excellent real-world prototype-chain example.

---

# 12. Function Prototype Chain

Functions are also objects.

```js
function test() {}
```

A simplified chain:

```text
test function object
      ↓
Function.prototype
      ↓
Object.prototype
      ↓
null
```

That is why functions can use methods such as:

```js
test.call(...)
test.apply(...)
test.bind(...)
```

Those methods are found through `Function.prototype`.

This connects directly to Lessons 46–48.

---

# 13. Important: Function.prototype vs test.prototype

These are different.

Given:

```js
function test() {}
```

### Internal prototype of function object

```js
Object.getPrototypeOf(test)
  === Function.prototype
```

This is about the function object itself inheriting function behavior.

### Normal `prototype` property

```js
test.prototype
```

This is the object used when `test` is invoked with `new`.

Do not confuse them.

Mental model:

```text
test
├── [[Prototype]]
│      ↓
│  Function.prototype
│
└── prototype
       ↓
   object for future instances
```

This distinction is one of the most important prototype interview topics.

---

# 14. String Primitive and Prototype Methods

Consider:

```js
const name = "Vikash";

console.log(
  name.toUpperCase()
);
```

A string primitive is not simply a normal object instance stored permanently with methods attached.

JavaScript provides access to methods defined on `String.prototype` through language-level boxing/wrapper behavior.

Conceptually:

```text
"Vikash"
   ↓
String-related method lookup behavior
   ↓
String.prototype
   ↓
toUpperCase
```

The exact internal mechanics are more nuanced, but the important learning point is that built-in methods are shared through prototypes.

---

# 15. hasOwn vs in with Chain

```js
const parent = {
  role: "Parent",
};

const child =
  Object.create(parent);

child.name = "Vikash";
```

Check:

```js
Object.hasOwn(
  child,
  "role"
);
```

Output:

```text
false
```

But:

```js
"role" in child;
```

Output:

```text
true
```

Because `in` searches the chain.

---

# 16. for...in and Inherited Enumerable Properties

`for...in` can enumerate enumerable string properties from both:

- the object
- its prototype chain

Example:

```js
const parent = {
  role: "Person",
};

const child =
  Object.create(parent);

child.name = "Vikash";

for (const key in child) {
  console.log(key);
}
```

Possible output:

```text
name
role
```

This is why code often checks:

```js
if (
  Object.hasOwn(
    child,
    key
  )
) {
  // own property only
}
```

---

# 17. Prototype Chain and this

Consider:

```js
const person = {
  greet() {
    console.log(
      this.name
    );
  },
};

const user =
  Object.create(person);

user.name = "Vikash";

user.greet();
```

Method lookup:

```text
user
↓
greet not own
↓
person
↓
greet found
```

Invocation:

```text
user.greet()
↑
receiver = user
```

Therefore:

```text
this = user
```

Again:

> Prototype lookup determines where the method is found. The call site determines regular-function `this`.

---

# 18. Prototype Mutation Affects Inheriting Objects

```js
const parent = {
  role: "Person",
};

const child =
  Object.create(parent);

console.log(child.role);
```

Output:

```text
Person
```

Now change:

```js
parent.role =
  "Updated Person";
```

Then:

```js
console.log(child.role);
```

Output:

```text
Updated Person
```

Why?

The value was not copied into `child`.

Lookup still reaches the shared prototype object.

---

# 19. Shadowing Prevents Seeing Later Prototype Changes

```js
child.role =
  "Developer";
```

Now:

```js
parent.role =
  "Manager";
```

But:

```js
console.log(child.role);
```

Output:

```text
Developer
```

The own property shadows the prototype property.

---

# 20. Long Chains Are Possible but Not Always Good Design

JavaScript allows several prototype levels:

```text
object
↓
prototype A
↓
prototype B
↓
prototype C
↓
Object.prototype
↓
null
```

But very deep inheritance chains can make behavior harder to reason about.

In real applications, composition is often easier to maintain than excessively deep inheritance.

You will study inheritance patterns in the next lesson.

---

# 21. instanceof and the Prototype Chain

Example:

```js
function User() {}

const user =
  new User();
```

```js
console.log(
  user instanceof User
);
```

Output:

```text
true
```

At a high level, `instanceof` checks whether:

```js
User.prototype
```

appears somewhere in `user`'s prototype chain.

Conceptually:

```text
user
↓
User.prototype ?
yes
↓
true
```

This is more accurate than thinking `instanceof` merely checks the constructor name.

---

# 22. Object.create(null) Has No Chain

```js
const dictionary =
  Object.create(null);
```

Its chain:

```text
dictionary
↓
null
```

Therefore:

```js
dictionary.toString
```

is:

```text
undefined
```

because there is no `Object.prototype` in the chain.

---

# 23. Complete Built-In Mental Model

Common chains:

```text
Plain object
obj
↓
Object.prototype
↓
null
```

```text
Array
arr
↓
Array.prototype
↓
Object.prototype
↓
null
```

```text
Function
fn
↓
Function.prototype
↓
Object.prototype
↓
null
```

```text
Constructor instance
instance
↓
Constructor.prototype
↓
Object.prototype
↓
null
```

These diagrams explain a large amount of everyday JavaScript behavior.

---

# 24. React / Real-World Connection

Even though modern React favors function components, prototype chains still matter for:

- class instances
- built-in arrays/functions
- third-party libraries
- `instanceof`
- debugging inherited methods
- object security
- prototype pollution
- understanding JavaScript classes

Classes do not replace prototypes; they provide nicer syntax over prototype-based behavior.

---

# 25. Interview Tracing Method

When asked:

> Where does this property come from?

Trace:

```text
1. Check own property

2. If missing:
   go to prototype

3. Repeat

4. Stop when found

5. If chain reaches null:
   property access returns undefined
```

When asked:

> What is this inside the inherited method?

Use a separate process:

```text
method lookup
≠
this binding

Find method through chain
then inspect call site
```

---

# Interview Answers

### What is a prototype chain?

A prototype chain is the sequence of prototype links JavaScript follows when resolving inherited properties.

### Where does the chain end?

At `null`.

### Why can arrays use methods such as map()?

Array instances inherit those methods through `Array.prototype`.

### How does instanceof use prototypes?

It checks whether the constructor's `prototype` object appears in the tested object's prototype chain.

---

# Key Takeaways

- Prototype chains are sequences of linked prototype objects.
- Property lookup starts on the object itself.
- JavaScript moves upward only when a property is missing.
- The nearest matching property wins.
- Own properties can shadow inherited properties.
- `Object.prototype` is near the top of many common chains.
- `null` ends the chain.
- Arrays inherit from `Array.prototype`.
- Functions inherit from `Function.prototype`.
- Constructor instances inherit from `Constructor.prototype`.
- Function object's `[[Prototype]]` and function's normal `prototype` property are different.
- `instanceof` relies on prototype-chain relationships.
- Method lookup and regular-function `this` binding are separate mechanisms.
