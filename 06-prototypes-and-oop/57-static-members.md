# Lesson 57 — Static Methods and Properties

Most class members belong to instances.

Static members are different.

> Static methods and properties belong to the class itself, not to individual instances.

---

## 1. Instance Method vs Static Method

Instance method:

```js
class User {
  greet() {
    console.log("hello");
  }
}

const user =
  new User();

user.greet();
```

Static method:

```js
class User {
  static createGuest() {
    return new User();
  }
}
```

Call it on the class:

```js
User.createGuest();
```

Not on an instance:

```js
user.createGuest();
```

That would fail.

---

# 2. Static Method Syntax

```js
class MathUtils {
  static add(a, b) {
    return a + b;
  }
}
```

Use:

```js
console.log(
  MathUtils.add(
    10,
    20
  )
);
```

Output:

```text
30
```

No instance is required.

---

# 3. Static Property Syntax

Modern JavaScript supports static fields:

```js
class AppConfig {
  static appName =
    "CareerLoop";
}
```

Use:

```js
console.log(
  AppConfig.appName
);
```

Output:

```text
CareerLoop
```

---

# 4. Instance Cannot Access Static Members

```js
class User {
  static role =
    "system-user";
}

const user =
  new User();

console.log(
  user.role
);
```

Output:

```text
undefined
```

Why?

`role` belongs to the class constructor object:

```text
User
└── role

user
└── [[Prototype]]
     ↓
User.prototype
```

The static property is not on `User.prototype`.

---

# 5. Where Static Members Live Conceptually

Given:

```js
class User {
  static category =
    "account";

  static create() {}

  greet() {}
}
```

Mental model:

```text
User
├── category
├── create()
└── prototype
      ↓
  User.prototype
      └── greet()
```

So:

```text
static → class constructor object

normal method → prototype

instance field → individual instance
```

---

# 6. this Inside a Static Method

```js
class User {
  static type =
    "User";

  static showType() {
    console.log(
      this.type
    );
  }
}
```

Call:

```js
User.showType();
```

Inside `showType`:

```text
this = User
```

Why?

The receiver is:

```js
User
```

This follows the same regular-function method-call rule from Section 5.

---

# 7. Static Factory Method

A common use case:

```js
class User {
  constructor(
    name,
    role
  ) {
    this.name = name;
    this.role = role;
  }

  static createAdmin(
    name
  ) {
    return new User(
      name,
      "admin"
    );
  }
}
```

Usage:

```js
const admin =
  User.createAdmin(
    "Vikash"
  );
```

This makes object creation more expressive.

---

# 8. Static Utility Method

```js
class Validator {
  static isEmail(value) {
    return value.includes("@");
  }
}
```

Use:

```js
Validator.isEmail(
  "test@example.com"
);
```

No object instance is needed because the operation does not depend on per-instance state.

---

# 9. Static Counter Example

```js
class User {
  static count = 0;

  constructor(name) {
    this.name = name;

    User.count++;
  }
}
```

Create:

```js
new User("A");
new User("B");
new User("C");
```

Then:

```js
console.log(
  User.count
);
```

Output:

```text
3
```

The static field is shared at class level.

---

# 10. Static Members and Inheritance

Static members can be inherited by subclasses.

```js
class Person {
  static category =
    "human";

  static describe() {
    return this.category;
  }
}

class Developer
  extends Person {}
```

Now:

```js
Developer.describe();
```

works.

Why?

There is also an inheritance relationship between the class constructor objects.

---

# 11. Static Property Shadowing

```js
class Person {
  static category =
    "human";
}

class Developer
  extends Person {
  static category =
    "developer";
}
```

Then:

```js
console.log(
  Person.category
);
```

Output:

```text
human
```

And:

```js
console.log(
  Developer.category
);
```

Output:

```text
developer
```

The child static property shadows the inherited one.

---

# 12. super in Static Methods

```js
class Person {
  static describe() {
    return "human";
  }
}

class Developer
  extends Person {
  static describe() {
    return (
      super.describe() +
      " developer"
    );
  }
}
```

Then:

```js
Developer.describe();
```

returns:

```text
human developer
```

---

# 13. Private Static Fields

Combine static and private:

```js
class Config {
  static #secret =
    "abc123";

  static getSecret() {
    return this.#secret;
  }
}
```

Use:

```js
Config.getSecret();
```

But outside code cannot access:

```js
Config.#secret;
```

---

# 14. When Should You Use Static Members?

Use static members when behavior/data belongs to the class concept itself rather than an individual instance.

Good examples:

- utility methods
- factory methods
- validation helpers
- counters
- configuration
- constants
- shared class-level metadata

---

# 15. When Not to Use Static Members

Do not use static members for data that should vary per instance.

Bad:

```js
class User {
  static name;
}
```

if every user needs a different name.

Better:

```js
class User {
  constructor(name) {
    this.name = name;
  }
}
```

---

# 16. Object.hasOwn() Check

```js
class User {
  static type =
    "account";

  greet() {}
}
```

Check:

```js
Object.hasOwn(
  User,
  "type"
);
```

Output:

```text
true
```

But:

```js
Object.hasOwn(
  User.prototype,
  "type"
);
```

Output:

```text
false
```

This confirms where the static member lives.

---

# 17. Built-In Static Method Examples

JavaScript itself uses static methods heavily.

Examples:

```js
Object.keys(obj);
Object.create(proto);
Array.isArray(value);
Promise.all(promises);
Math.max(1, 2);
```

You do not write:

```js
new Object().keys(...)
```

because these operations belong to the constructor/namespace-like object itself.

---

# 18. Static vs Instance Mental Model

```text
class User
│
├── static create()
├── static count
│
└── prototype
      ↓
  User.prototype
      └── greet()

instance
├── own fields
└── [[Prototype]]
      ↓
  User.prototype
```

---

# 19. React / Application Connection

Static methods are common in:

- utility classes
- service abstractions
- domain models
- factory patterns
- serializers
- parsers

Example:

```js
class ApiResponse {
  static success(data) {
    return {
      success: true,
      data,
    };
  }
}
```

No instance is needed.

---

# 20. Interview Questions

### What is a static method?

A method that belongs to the class constructor itself rather than its instances.

### Can an instance call a static method directly?

No, not through the normal instance prototype chain.

### What does this refer to inside a static method called as Class.method()?

It refers to the class constructor object used as the receiver.

---

# Key Takeaways

- Static members belong to the class itself.
- Instance methods normally live on the prototype.
- Static fields live on the constructor object.
- Instances do not automatically access static members.
- Static methods can use `this` to refer to the class receiver.
- Static members can be inherited by subclasses.
- Static members can be shadowed or overridden.
- Private static fields are supported.
- Static methods are useful for factories, utilities, validation, and shared class-level behavior.
