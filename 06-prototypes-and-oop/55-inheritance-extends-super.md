# Lesson 55 — Inheritance, `extends` and `super`

ES6 classes provide cleaner syntax for inheritance.

Instead of manually linking prototype objects, you can use:

```text
extends
super
```

But underneath, JavaScript still creates prototype relationships.

---

## 1. Basic extends Example

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(
      `Hello, I am ${this.name}`
    );
  }
}

class Developer
  extends Person {
  code() {
    console.log(
      `${this.name} is coding`
    );
  }
}
```

Create:

```js
const dev =
  new Developer(
    "Vikash"
  );
```

Now:

```js
dev.greet();
dev.code();
```

Both work.

---

# 2. What extends Does Conceptually

```js
class Developer
  extends Person {}
```

creates prototype relationships roughly like:

```text
dev
↓
Developer.prototype
↓
Person.prototype
↓
Object.prototype
↓
null
```

This is prototypal inheritance expressed with cleaner syntax.

---

# 3. Derived Constructor

Suppose the child class needs its own constructor:

```js
class Developer
  extends Person {
  constructor(
    name,
    skill
  ) {
    // ...
  }
}
```

A derived constructor must call `super()` before using `this`.

Correct:

```js
class Developer
  extends Person {
  constructor(
    name,
    skill
  ) {
    super(name);

    this.skill = skill;
  }
}
```

---

# 4. Why super() Is Required

In a derived class constructor, `this` is not initialized for use until the parent constructor has run through `super()`.

This fails:

```js
class Developer
  extends Person {
  constructor(
    name,
    skill
  ) {
    this.skill = skill;

    super(name);
  }
}
```

It throws an error.

Correct order:

```text
derived constructor starts
        ↓
super(...)
        ↓
parent constructor runs
        ↓
this becomes available
        ↓
child initialization
```

---

# 5. What super(name) Does

Given:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
}
```

Inside:

```js
class Developer
  extends Person {
  constructor(
    name,
    skill
  ) {
    super(name);

    this.skill = skill;
  }
}
```

`super(name)` invokes the parent class constructor.

So:

```text
Person constructor
→ this.name = name

Developer constructor
→ this.skill = skill
```

Both initialize the same derived instance.

---

# 6. Parent Methods Are Inherited

```js
class Person {
  greet() {
    console.log("hello");
  }
}

class Developer
  extends Person {}
```

Then:

```js
const dev =
  new Developer();

dev.greet();
```

works because the prototype chain includes:

```text
Developer.prototype
↓
Person.prototype
```

---

# 7. Child Method Overrides Parent Method

```js
class Person {
  greet() {
    console.log(
      "Hello from Person"
    );
  }
}

class Developer
  extends Person {
  greet() {
    console.log(
      "Hello from Developer"
    );
  }
}
```

Then:

```js
const dev =
  new Developer();

dev.greet();
```

Output:

```text
Hello from Developer
```

Why?

The lookup finds the closer method first:

```text
dev
↓
Developer.prototype
↓
greet found
↓
stop
```

This is method overriding.

---

# 8. Calling Parent Method with super

Sometimes the child wants to extend parent behavior instead of completely replacing it.

```js
class Person {
  greet() {
    console.log(
      "Hello"
    );
  }
}

class Developer
  extends Person {
  greet() {
    super.greet();

    console.log(
      "I am a developer"
    );
  }
}
```

Output:

```text
Hello
I am a developer
```

`super.greet()` accesses the parent prototype's method.

---

# 9. super Is Not Just Parent Object Variable

It is tempting to think:

```text
super = parent object
```

That is too simplistic.

`super` has special language semantics tied to class method definitions and inheritance relationships.

Use this practical mental model:

```text
super(...)
→ call parent constructor

super.method()
→ call parent implementation
```

---

# 10. this Inside Parent Method Called with super

Consider:

```js
class Person {
  greet() {
    console.log(
      this.name
    );
  }
}

class Developer
  extends Person {
  constructor(name) {
    super(name);
  }

  greet() {
    super.greet();
  }
}
```

When:

```js
dev.greet();
```

then inside `Person.prototype.greet`:

```text
this = dev
```

The parent method runs with the current instance as `this`.

This is very important.

`super.greet()` chooses the parent implementation, but it does not change `this` into `Person.prototype`.

---

# 11. Full Example

```js
class Person {
  constructor(
    name,
    age
  ) {
    this.name = name;
    this.age = age;
  }

  introduce() {
    console.log(
      `I am ${this.name}`
    );
  }
}

class Developer
  extends Person {
  constructor(
    name,
    age,
    skill
  ) {
    super(
      name,
      age
    );

    this.skill = skill;
  }

  introduce() {
    super.introduce();

    console.log(
      `My skill is ${this.skill}`
    );
  }
}

const dev =
  new Developer(
    "Vikash",
    25,
    "JavaScript"
  );

dev.introduce();
```

Output:

```text
I am Vikash
My skill is JavaScript
```

---

# 12. Prototype Chain Behind extends

For:

```js
class Developer
  extends Person {}
```

instance side:

```text
dev
↓
Developer.prototype
↓
Person.prototype
↓
Object.prototype
↓
null
```

Constructor side also has an inheritance relationship:

```text
Developer
↓
Person
↓
Function.prototype
↓
Object.prototype
↓
null
```

That constructor-side inheritance is why static members can be inherited too.

---

# 13. Static Inheritance

```js
class Person {
  static category() {
    return "human";
  }
}

class Developer
  extends Person {}
```

Then:

```js
console.log(
  Developer.category()
);
```

Output:

```text
human
```

The child class inherits static behavior from the parent class constructor.

---

# 14. super in Static Methods

```js
class Person {
  static category() {
    return "human";
  }
}

class Developer
  extends Person {
  static category() {
    return (
      super.category() +
      " developer"
    );
  }
}
```

Then:

```js
Developer.category();
```

returns:

```text
human developer
```

---

# 15. Default Child Constructor

If a derived class has no explicit constructor:

```js
class Developer
  extends Person {}
```

JavaScript conceptually provides behavior similar to:

```js
constructor(...args) {
  super(...args);
}
```

So parent initialization still works.

---

# 16. instanceof with extends

```js
const dev =
  new Developer(
    "Vikash"
  );
```

Then:

```js
dev instanceof Developer;
```

is:

```text
true
```

and:

```js
dev instanceof Person;
```

is also:

```text
true
```

because both prototype objects appear in the chain.

---

# 17. Method Lookup with Overriding

Suppose:

```js
class A {
  test() {
    console.log("A");
  }
}

class B extends A {
  test() {
    console.log("B");
  }
}

class C extends B {}
```

Call:

```js
const c = new C();

c.test();
```

Search:

```text
c
↓
C.prototype
↓
test?
no
↓
B.prototype
↓
test?
yes
↓
"B"
```

The nearest implementation wins.

---

# 18. Deep Inheritance Warning

Technically you can build:

```text
Base
↓
Level1
↓
Level2
↓
Level3
↓
Level4
```

But deep inheritance trees often create:

- tight coupling
- hidden behavior
- difficult debugging
- fragile overrides

In production design, composition is often preferred when inheritance becomes too deep.

---

# 19. extends Works with Constructible Values

Classes commonly extend classes:

```js
class Developer
  extends Person {}
```

But conceptually `extends` works with a constructible parent value that can serve as the superclass.

For normal application code, extending classes is the standard and clearest pattern.

---

# 20. React Class Connection

Older React components used:

```jsx
class Counter
  extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      count: 0,
    };
  }

  render() {
    return (
      <div>
        {this.state.count}
      </div>
    );
  }
}
```

Why `super(props)`?

Because `Counter` is a derived class and the parent constructor must run before using `this` in the derived constructor.

Modern function components avoid this pattern, but it remains an important JavaScript inheritance example.

---

# 21. extends vs Manual Prototype Setup

Old style:

```js
function Developer() {}

Developer.prototype =
  Object.create(
    Person.prototype
  );

Developer.prototype.constructor =
  Developer;
```

Modern class syntax:

```js
class Developer
  extends Person {}
```

The modern syntax is shorter and easier to read, but the prototype relationship still exists underneath.

---

# 22. Interview Questions

### What does extends do?

It establishes inheritance between a child class and a parent class, creating prototype relationships for both instance methods and class constructors.

### Why must super() be called before this in a derived constructor?

Because the derived instance's `this` is not available for use until the parent constructor has been invoked.

### What does super.method() do?

It invokes the parent implementation while preserving the current instance as `this`.

### What is method overriding?

It occurs when a child class defines a method with the same name as an inherited parent method.

---

# Key Takeaways

- `extends` creates inheritance between classes.
- The underlying mechanism is still prototype-based.
- Derived constructors must call `super()` before using `this`.
- `super()` invokes the parent constructor.
- `super.method()` calls the parent implementation.
- Parent methods run with the current child instance as `this`.
- Child methods can override parent methods.
- Static methods can also be inherited.
- `instanceof` reflects the resulting prototype chain.
- Deep inheritance should be used carefully; composition is often easier to maintain.
