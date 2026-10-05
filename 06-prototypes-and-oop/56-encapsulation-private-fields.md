# Lesson 56 — Encapsulation and Private Fields

Encapsulation means keeping an object's internal state controlled instead of exposing every implementation detail directly.

In JavaScript classes, modern private fields use the `#` syntax.

The central idea is:

> Keep internal data private and expose only the operations that should be public.

---

## 1. Why Encapsulation Matters

Without encapsulation:

```js
class BankAccount {
  constructor(balance) {
    this.balance = balance;
  }
}
```

Any code can do:

```js
const account =
  new BankAccount(1000);

account.balance = -50000;
```

The class has no control over invalid state.

Encapsulation lets the class protect that state.

---

# 2. Private Fields

Use `#` before a field name:

```js
class BankAccount {
  #balance;

  constructor(balance) {
    this.#balance = balance;
  }
}
```

Now:

```js
const account =
  new BankAccount(1000);
```

This works inside the class:

```js
console.log(
  account
);
```

But this does not:

```js
account.#balance;
```

It is a syntax error outside the class body.

---

# 3. Private Fields Are Truly Private

This is important.

A private field is not simply a naming convention.

Compare:

```js
class User {
  _password = "secret";
}
```

The underscore only communicates:

> please treat this as internal.

But code can still access it:

```js
user._password;
```

With:

```js
class User {
  #password = "secret";
}
```

outside code cannot directly access `#password`.

---

# 4. Declaring Private Fields

Private fields must be declared in the class body.

```js
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }
}
```

You cannot dynamically invent a private field later like a normal property.

This is invalid:

```js
class User {
  constructor() {
    this.#unknown = 10;
  }
}
```

if `#unknown` was never declared.

---

# 5. Public Method Accessing Private State

Private data is usually exposed indirectly through methods.

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    if (amount <= 0) {
      throw new Error(
        "Amount must be positive"
      );
    }

    this.#balance += amount;
  }

  getBalance() {
    return this.#balance;
  }
}
```

Usage:

```js
const account =
  new BankAccount();

account.deposit(1000);

console.log(
  account.getBalance()
);
```

Output:

```text
1000
```

The caller cannot directly modify `#balance`.

---

# 6. Encapsulation Lets You Enforce Rules

```js
class BankAccount {
  #balance = 0;

  deposit(amount) {
    if (amount <= 0) {
      throw new Error(
        "Invalid deposit"
      );
    }

    this.#balance += amount;
  }

  withdraw(amount) {
    if (
      amount <= 0 ||
      amount > this.#balance
    ) {
      throw new Error(
        "Invalid withdrawal"
      );
    }

    this.#balance -= amount;
  }
}
```

Now the object's state can only change through controlled operations.

Mental model:

```text
outside code
     ↓
public method
     ↓
validation
     ↓
private state
```

---

# 7. Private Methods

JavaScript also supports private methods.

```js
class UserService {
  #validateEmail(email) {
    return email.includes("@");
  }

  createUser(email) {
    if (
      !this.#validateEmail(
        email
      )
    ) {
      throw new Error(
        "Invalid email"
      );
    }

    return {
      email,
    };
  }
}
```

`#validateEmail` is available only inside the class.

---

# 8. Private Static Fields

Private members can also be static.

```js
class Config {
  static #apiKey =
    "secret-key";

  static getKey() {
    return this.#apiKey;
  }
}
```

Use:

```js
Config.getKey();
```

But:

```js
Config.#apiKey;
```

is invalid outside the class.

---

# 9. Private Fields Are Per Instance

```js
class Counter {
  #count = 0;

  increment() {
    this.#count++;
  }

  getCount() {
    return this.#count;
  }
}
```

Create:

```js
const a =
  new Counter();

const b =
  new Counter();

a.increment();
a.increment();

b.increment();
```

Results:

```text
a → 2
b → 1
```

Each instance owns its own private field state.

---

# 10. Private Fields Are Not Prototype Properties

Consider:

```js
class User {
  #name;

  constructor(name) {
    this.#name = name;
  }

  showName() {
    console.log(
      this.#name
    );
  }
}
```

`showName` lives on the prototype.

But `#name` is private instance state.

Conceptually:

```text
user instance
├── private #name
└── [[Prototype]]
       ↓
   User.prototype
       └── showName()
```

---

# 11. Private Fields and Inheritance

This part is important.

Private fields belong to the class that declares them.

```js
class Person {
  #name;

  constructor(name) {
    this.#name = name;
  }
}
```

A subclass cannot directly access:

```js
this.#name
```

unless it declares its own private field with that same spelling.

Private fields are not inherited in the same accessible way as public/protected members from some other languages.

---

# 12. Exposing Private Data Safely

You may expose a read-only view:

```js
class User {
  #email;

  constructor(email) {
    this.#email = email;
  }

  getEmail() {
    return this.#email;
  }
}
```

Or expose controlled updates:

```js
updateEmail(email) {
  if (!email.includes("@")) {
    throw new Error(
      "Invalid email"
    );
  }

  this.#email = email;
}
```

This keeps validation inside the class.

---

# 13. Encapsulation Is More Than Private Fields

Encapsulation can also be achieved using:

- closures
- modules
- functions
- private fields
- controlled APIs

Example using closure:

```js
function createCounter() {
  let count = 0;

  return {
    increment() {
      count++;
    },

    getCount() {
      return count;
    },
  };
}
```

Here `count` is private because of closure scope.

So:

> Encapsulation is the design goal. Private fields are one language tool for achieving it.

---

# 14. WeakMap Historical Pattern

Before private fields, WeakMap was sometimes used for private instance data.

Example conceptually:

```js
const privateData =
  new WeakMap();

class User {
  constructor(name) {
    privateData.set(
      this,
      {
        name,
      }
    );
  }
}
```

This still works, but `#privateField` syntax is usually clearer when appropriate.

---

# 15. Common Mistake — Thinking # Is a String Property

This does not work:

```js
user["#name"];
```

Private fields are not ordinary string-keyed properties.

They use separate private-name semantics.

So:

```text
#name
≠
"#name"
```

---

# 16. Common Mistake — Accessing with Object.keys()

Private fields do not appear in:

```js
Object.keys(user);
```

They also do not appear through ordinary property enumeration.

They are not normal object properties.

---

# 17. Practical Example

```js
class Subscription {
  #status =
    "inactive";

  activate() {
    this.#status =
      "active";
  }

  deactivate() {
    this.#status =
      "inactive";
  }

  isActive() {
    return (
      this.#status ===
      "active"
    );
  }
}
```

Outside code interacts with behavior, not raw internal state.

---

# 18. React / Application Connection

React function components usually use closures and hooks rather than class-private fields.

But encapsulation still appears everywhere:

- custom hooks
- modules
- service classes
- SDK wrappers
- domain models
- backend services

Example idea:

```js
class ApiClient {
  #token;

  constructor(token) {
    this.#token = token;
  }

  request() {
    // use token internally
  }
}
```

The consumer does not need direct access to internal implementation details.

---

# 19. Interview Questions

### What is encapsulation?

Encapsulation is the practice of hiding internal implementation details and exposing only controlled public behavior.

### What does # mean in a JavaScript class?

It declares a private field or private method that is accessible only within the declaring class body.

### Are private fields ordinary object properties?

No. They use separate private-field semantics and are not accessible through normal string property access.

---

# Key Takeaways

- Encapsulation protects internal object state.
- `#field` creates real private class state.
- Private fields must be declared in the class body.
- Outside code cannot directly access private fields.
- Public methods can safely expose controlled behavior.
- Private fields are per-instance unless declared static.
- Private methods are also supported.
- Private fields are not ordinary string-keyed properties.
- Subclasses cannot directly access a parent's private fields.
- Closures and modules can also provide encapsulation.
