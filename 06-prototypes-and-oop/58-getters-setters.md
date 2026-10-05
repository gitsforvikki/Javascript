# Lesson 58 — Getters and Setters

Getters and setters let you control how a property is read and written while still using normal property syntax.

Instead of calling:

```js
user.getFullName();
```

you can expose:

```js
user.fullName;
```

And instead of:

```js
user.setFullName(value);
```

you can write:

```js
user.fullName = value;
```

---

## 1. Basic Getter

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
}
```

Use:

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

Output:

```text
Vikash Kumar
```

Notice:

```js
user.fullName
```

not:

```js
user.fullName()
```

---

# 2. Getter Looks Like a Property

A getter is a function internally, but property access triggers it.

Mental model:

```text
user.fullName
      ↓
getter runs
      ↓
computed value returned
```

This lets you expose derived values elegantly.

---

# 3. Basic Setter

```js
class User {
  constructor(name) {
    this._name = name;
  }

  get name() {
    return this._name;
  }

  set name(value) {
    this._name = value;
  }
}
```

Use:

```js
user.name =
  "Rahul";
```

The assignment invokes the setter.

---

# 4. Avoid Infinite Recursion

This is wrong:

```js
class User {
  set name(value) {
    this.name = value;
  }
}
```

Why?

```text
this.name = value
↓
setter runs
↓
this.name = value
↓
setter runs again
↓
infinite recursion
```

Use a different backing field:

```js
this._name
```

or preferably a private field:

```js
this.#name
```

---

# 5. Getter/Setter with Private Field

```js
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }

  get name() {
    return this.#name;
  }

  set name(value) {
    this.#name = value;
  }
}
```

This combines:

- encapsulation
- controlled access
- clean property syntax

---

# 6. Validation in a Setter

```js
class User {
  #age = 0;

  get age() {
    return this.#age;
  }

  set age(value) {
    if (
      value < 0 ||
      value > 150
    ) {
      throw new Error(
        "Invalid age"
      );
    }

    this.#age = value;
  }
}
```

Now:

```js
user.age = -10;
```

throws an error.

This is one of the most useful setter patterns.

---

# 7. Computed Getter

```js
class Cart {
  constructor(items) {
    this.items = items;
  }

  get total() {
    return this.items.reduce(
      (sum, item) =>
        sum + item.price,
      0
    );
  }
}
```

Usage:

```js
console.log(
  cart.total
);
```

The value is computed when read.

---

# 8. Read-Only Property

You can define only a getter:

```js
class Circle {
  constructor(radius) {
    this.radius = radius;
  }

  get area() {
    return (
      Math.PI *
      this.radius **
        2
    );
  }
}
```

There is no setter for `area`.

So it acts like a derived read-only property.

---

# 9. Write-Only Setter

JavaScript allows a setter without a getter:

```js
class Password {
  #value;

  set value(input) {
    this.#value =
      input.trim();
  }
}
```

Reading:

```js
password.value
```

returns `undefined` if no getter exists.

Write-only properties are less common but valid.

---

# 10. Getters and Setters in Object Literals

They are not limited to classes.

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

Usage:

```js
console.log(
  user.fullName
);

user.fullName =
  "Rahul Sharma";
```

---

# 11. Getter/Setter Property Descriptor

Accessor properties use descriptors containing:

```text
get
set
enumerable
configurable
```

Unlike a data property, they do not use normal:

```text
value
writable
```

This connects directly to Lesson 50.

---

# 12. Object.defineProperty() Accessor

```js
const user = {
  firstName: "Vikash",
  lastName: "Kumar",
};

Object.defineProperty(
  user,
  "fullName",
  {
    get() {
      return (
        this.firstName +
        " " +
        this.lastName
      );
    },

    set(value) {
      const [first, last] =
        value.split(" ");

      this.firstName =
        first;

      this.lastName =
        last;
    },
  }
);
```

This creates an accessor property manually.

---

# 13. Getter Methods Live on Prototype in Classes

```js
class User {
  get name() {
    return "Vikash";
  }
}
```

The getter descriptor exists on:

```js
User.prototype
```

not as an own method on each instance.

Check:

```js
Object.getOwnPropertyDescriptor(
  User.prototype,
  "name"
);
```

You will see a descriptor containing `get`.

---

# 14. this Inside Getter

```js
class User {
  constructor(name) {
    this._name = name;
  }

  get name() {
    return this._name;
  }
}
```

For:

```js
user.name
```

the getter runs with:

```text
this = user
```

because the property access receiver is `user`.

---

# 15. Setter and this

Likewise:

```js
user.name =
  "Rahul";
```

invokes the setter with:

```text
this = user
```

So getters/setters can operate on the receiver's state.

---

# 16. Inherited Getter

```js
const person = {
  get label() {
    return this.name;
  },
};

const user =
  Object.create(person);

user.name = "Vikash";

console.log(
  user.label
);
```

Output:

```text
Vikash
```

The getter is found on the prototype, but:

```text
this = user
```

This is the same pattern you learned with inherited regular methods.

---

# 17. When Getters Are Useful

Good use cases:

- derived values
- formatted values
- read-only computed properties
- hiding internal storage
- lazy calculations
- normalized access APIs

Example:

```js
class Product {
  constructor(
    price,
    taxRate
  ) {
    this.price = price;
    this.taxRate = taxRate;
  }

  get finalPrice() {
    return (
      this.price *
      (1 + this.taxRate)
    );
  }
}
```

---

# 18. When Setters Are Useful

Good use cases:

- validation
- normalization
- controlled mutation
- hiding implementation details

Example:

```js
class User {
  #email;

  set email(value) {
    const normalized =
      value.trim().toLowerCase();

    if (
      !normalized.includes(
        "@"
      )
    ) {
      throw new Error(
        "Invalid email"
      );
    }

    this.#email =
      normalized;
  }

  get email() {
    return this.#email;
  }
}
```

---

# 19. Do Not Hide Expensive Work Unexpectedly

This is a design concern.

Because:

```js
user.report
```

looks like simple property access, avoid placing surprising heavy side effects inside getters.

Bad idea:

```js
get report() {
  // huge network request
}
```

Getters are best for predictable property-like behavior.

---

# 20. Getter vs Method

Getter:

```js
user.fullName
```

Method:

```js
user.getFullName()
```

Use a getter when the result conceptually behaves like a property.

Use a method when the operation feels like an action or requires arguments.

---

# 21. Setter vs Method

Setter:

```js
user.email =
  "a@b.com";
```

Method:

```js
user.updateEmail(
  "a@b.com"
);
```

Both are valid designs.

A method may be clearer when the operation has meaningful side effects or business semantics.

---

# 22. Static Getter

Getters can also be static.

```js
class Config {
  static get environment() {
    return "production";
  }
}
```

Use:

```js
Config.environment;
```

This property belongs to the class, not instances.

---

# 23. Static Setter

```js
class Config {
  static #mode =
    "development";

  static get mode() {
    return this.#mode;
  }

  static set mode(value) {
    this.#mode = value;
  }
}
```

Use:

```js
Config.mode =
  "production";
```

---

# 24. Practical Full Example

```js
class Employee {
  #salary;

  constructor(
    name,
    salary
  ) {
    this.name = name;
    this.salary = salary;
  }

  get salary() {
    return this.#salary;
  }

  set salary(value) {
    if (value < 0) {
      throw new Error(
        "Salary cannot be negative"
      );
    }

    this.#salary = value;
  }

  get summary() {
    return (
      `${this.name}: ₹${this.#salary}`
    );
  }
}
```

This combines:

- private fields
- setter validation
- getter access
- derived getter values

---

# 25. React / Application Connection

Getters/setters are not a common pattern for React state itself.

But they appear in:

- domain models
- class-based services
- ORMs
- SDKs
- libraries
- browser APIs

In React, explicit state updates are usually preferred over hiding mutations behind setters.

---

# 26. Interview Questions

### What is a getter?

A getter is an accessor function that runs when a property is read.

### What is a setter?

A setter is an accessor function that runs when a property is assigned.

### Why should a setter not assign to the same property name directly?

Because it would trigger itself recursively and cause infinite recursion.

### Where do class getters/setters live?

Normal class accessors are defined on the class prototype.

---

# Key Takeaways

- Getters run on property reads.
- Setters run on property assignments.
- Getters/setters use property syntax, not normal function-call syntax.
- Private fields are excellent backing storage for accessors.
- Setters can validate or normalize values.
- Getters are useful for computed and read-only properties.
- Avoid infinite recursion by using a different backing field.
- Accessors can be declared in classes or object literals.
- Accessor descriptors use `get` and `set`, not `value` and `writable`.
- Inherited getters/setters still operate with the actual receiver as `this`.
