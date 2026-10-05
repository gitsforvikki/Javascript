# Lesson 54 — ES6 Classes

ES6 introduced the `class` syntax to make object-oriented JavaScript easier to read and write.

But this is crucial:

> JavaScript classes are built on top of the prototype system.

They do not replace prototypes.

---

## 1. Basic Class Syntax

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(
      `Hello ${this.name}`
    );
  }
}
```

Create an instance:

```js
const user =
  new User("Vikash");

user.greet();
```

Output:

```text
Hello Vikash
```

---

# 2. What constructor Does

The `constructor` method runs when you create an instance using `new`.

```js
class User {
  constructor(
    name,
    role
  ) {
    this.name = name;
    this.role = role;
  }
}
```

Then:

```js
const user =
  new User(
    "Vikash",
    "Developer"
  );
```

Inside `constructor`:

```text
this = newly created instance
```

This is the same new-binding concept from Lesson 49.

---

# 3. Class Methods Live on the Prototype

Consider:

```js
class User {
  greet() {
    console.log("hello");
  }
}
```

The method is not normally recreated on every instance.

Check:

```js
const a = new User();
const b = new User();

console.log(
  a.greet === b.greet
);
```

Output:

```text
true
```

Why?

Conceptually:

```text
a
↓
User.prototype
   └── greet()

b
↓
User.prototype
   └── same greet()
```

---

# 4. Verify Prototype Relationship

```js
console.log(
  Object.getPrototypeOf(a)
    === User.prototype
);
```

Output:

```text
true
```

This proves classes still use prototypes.

---

# 5. Class Syntax vs Constructor Function

Constructor-function version:

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
```

Class version:

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(
      `Hello ${this.name}`
    );
  }
}
```

The class version is cleaner and easier to maintain.

---

# 6. Classes Must Be Called with new

This is invalid:

```js
class User {}

User();
```

It throws a `TypeError`.

Classes must be constructed with:

```js
new User();
```

This is stricter than ordinary constructor functions, which can technically be called without `new` even though doing so may be incorrect.

---

# 7. Classes Are Not Hoisted Like Function Declarations

This does not work:

```js
const user =
  new User();

class User {}
```

A class declaration is in a temporal dead zone before its declaration is evaluated.

So use classes after declaration.

---

# 8. Classes Run in Strict Mode

Code inside class bodies is strict mode automatically.

Example:

```js
class User {
  method() {
    console.log(this);
  }
}
```

If the method is detached:

```js
const user =
  new User();

const fn =
  user.method;

fn();
```

then:

```text
this = undefined
```

This matches strict-mode regular-function behavior.

---

# 9. Instance Properties

```js
class User {
  constructor(name) {
    this.name = name;
  }
}
```

`name` becomes an own property of the instance.

```js
const user =
  new User("Vikash");

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

# 10. Prototype Methods

```js
class User {
  greet() {}
}

const user =
  new User();
```

Check:

```js
Object.hasOwn(
  user,
  "greet"
);
```

Output:

```text
false
```

But:

```js
"greet" in user;
```

Output:

```text
true
```

The method is inherited from `User.prototype`.

---

# 11. Class Fields

Modern JavaScript allows class fields:

```js
class User {
  role = "Developer";

  constructor(name) {
    this.name = name;
  }
}
```

Create:

```js
const user =
  new User("Vikash");
```

Now both `name` and `role` are instance properties.

---

# 12. Arrow Function Class Fields

You may see:

```js
class User {
  name = "Vikash";

  greet = () => {
    console.log(
      this.name
    );
  };
}
```

Here `greet` is created as an instance field, not a shared prototype method.

So:

```js
const a = new User();
const b = new User();

console.log(
  a.greet === b.greet
);
```

Output:

```text
false
```

Each instance gets its own arrow function.

This can help with lexical `this`, but it also means the function is not shared through the prototype.

---

# 13. Methods and this

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(
      this.name
    );
  }
}

const user =
  new User("Vikash");

user.greet();
```

Since the call is:

```js
user.greet();
```

the receiver is `user`.

Therefore:

```text
this = user
```

Again, class syntax does not change the fundamental regular-function `this` rules.

---

# 14. Detached Class Method

```js
const fn =
  user.greet;

fn();
```

The receiver is lost.

Because class methods run in strict mode:

```text
this = undefined
```

This is why older React class components often needed method binding.

---

# 15. Getters and Setters in Classes

```js
class User {
  constructor(
    firstName,
    lastName
  ) {
    this.firstName =
      firstName;

    this.lastName =
      lastName;
  }

  get fullName() {
    return (
      this.firstName +
      " " +
      this.lastName
    );
  }

  set fullName(value) {
    const [first, last] =
      value.split(" ");

    this.firstName = first;
    this.lastName = last;
  }
}
```

Usage:

```js
const user =
  new User(
    "Vikash",
    "Kumar"
  );

console.log(
  user.fullName
);
```

---

# 16. Static Members Preview

Static methods belong to the class itself, not its instances.

```js
class User {
  static createGuest() {
    return new User(
      "Guest"
    );
  }

  constructor(name) {
    this.name = name;
  }
}
```

Use:

```js
const guest =
  User.createGuest();
```

Not:

```js
guest.createGuest();
```

Static members will be covered more deeply later.

---

# 17. Class Expression

Like functions, classes can also be expressions:

```js
const User =
  class {
    constructor(name) {
      this.name = name;
    }
  };
```

This is valid JavaScript, though class declarations are more common for named models.

---

# 18. Classes Are Functions

This surprises many developers.

```js
class User {}
```

Then:

```js
console.log(
  typeof User
);
```

Output:

```text
function
```

A class declaration creates a special kind of function object with class semantics.

---

# 19. Prototype Diagram

```js
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {}
}

const user =
  new User("Vikash");
```

Mental model:

```text
User
│
├── function object
└── prototype
      ↓
 User.prototype
      ├── greet()
      ├── constructor → User
      │
      └── [[Prototype]]
             ↓
       Object.prototype
             ↓
            null

user
├── name: "Vikash"
└── [[Prototype]]
       ↓
   User.prototype
```

---

# 20. Why Classes Are Useful

Classes improve readability when modeling:

- users
- orders
- products
- services
- domain entities
- reusable object-oriented structures

They provide clean syntax for:

- constructors
- methods
- inheritance
- getters/setters
- static members
- private fields

---

# 21. React Connection

Older React code commonly used:

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

Modern React primarily uses function components and hooks, but class syntax remains important for JavaScript fundamentals and legacy code.

---

# 22. Interview Questions

### Are JavaScript classes true replacements for prototypes?

No. Class syntax is built on JavaScript's prototype system.

### Where are normal class methods stored?

On the class's `prototype` object.

### Are class methods strict mode?

Yes.

### Can a class be called without new?

No.

---

# Key Takeaways

- ES6 classes provide cleaner syntax for constructor/prototype patterns.
- Classes still use prototypes internally.
- `constructor` initializes new instances.
- Normal class methods live on the prototype.
- Instance fields live directly on each instance.
- Arrow class fields create one function per instance.
- Classes must be used with `new`.
- Class bodies run in strict mode.
- Detached methods lose their receiver.
- Classes can use getters, setters, static members, inheritance, and private fields.
