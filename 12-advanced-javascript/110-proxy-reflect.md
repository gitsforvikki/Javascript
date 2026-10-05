# Lesson 110 — Proxy and Reflect

`Proxy` and `Reflect` are advanced JavaScript features used for **metaprogramming**.

Metaprogramming means:

> Writing code that intercepts or customizes how other code interacts with objects.

With `Proxy`, you can intercept operations such as:

- reading properties
- writing properties
- checking property existence
- deleting properties
- calling functions
- constructing objects
- listing keys

`Reflect` provides standard methods for performing these underlying operations safely and consistently.

---

## 1. Basic Proxy

Syntax:

```js
const proxy =
  new Proxy(
    target,
    handler
  );
```

Example:

```js
const user = {
  name:
    "Vikash",
};

const proxy =
  new Proxy(
    user,
    {}
  );
```

With an empty handler, the proxy mostly behaves like the target.

---

## 2. target and handler

```text
target
→ original object

handler
→ object containing traps
```

A trap intercepts a specific operation.

Example:

```text
property read
→ get trap

property write
→ set trap
```

---

## 3. get Trap

```js
const user = {
  name:
    "Vikash",
};

const proxy =
  new Proxy(
    user,
    {
      get(
        target,
        property,
        receiver
      ) {
        console.log(
          "Reading:",
          property
        );

        return Reflect.get(
          target,
          property,
          receiver
        );
      },
    }
  );
```

Now:

```js
proxy.name;
```

logs the property access.

---

## 4. Why Use Reflect.get?

You could write:

```js
return target[
  property
];
```

But:

```js
Reflect.get(
  target,
  property,
  receiver
);
```

more accurately performs the normal language-level property access operation and correctly preserves receiver semantics for getters and prototype behavior.

---

## 5. set Trap

```js
const proxy =
  new Proxy(
    user,
    {
      set(
        target,
        property,
        value,
        receiver
      ) {
        console.log(
          "Setting:",
          property,
          value
        );

        return Reflect.set(
          target,
          property,
          value,
          receiver
        );
      },
    }
  );
```

Then:

```js
proxy.name =
  "Rahul";
```

is intercepted.

---

## 6. set Must Return a Boolean

A `set` trap should return whether assignment succeeded.

```js
return true;
```

or, better:

```js
return Reflect.set(
  target,
  property,
  value,
  receiver
);
```

In strict mode, returning false can cause assignment to throw.

---

## 7. Validation with Proxy

```js
const user = {
  age: 25,
};

const proxy =
  new Proxy(
    user,
    {
      set(
        target,
        property,
        value,
        receiver
      ) {
        if (
          property ===
            "age" &&
          (
            typeof value !==
              "number" ||
            value < 0
          )
        ) {
          throw new TypeError(
            "age must be a non-negative number"
          );
        }

        return Reflect.set(
          target,
          property,
          value,
          receiver
        );
      },
    }
  );
```

Now invalid writes can be rejected.

---

## 8. has Trap

The `in` operator can be intercepted.

```js
const proxy =
  new Proxy(
    user,
    {
      has(
        target,
        property
      ) {
        console.log(
          "Checking:",
          property
        );

        return Reflect.has(
          target,
          property
        );
      },
    }
  );
```

Then:

```js
"name" in proxy;
```

uses the trap.

---

## 9. deleteProperty Trap

```js
const proxy =
  new Proxy(
    user,
    {
      deleteProperty(
        target,
        property
      ) {
        if (
          property ===
          "id"
        ) {
          return false;
        }

        return Reflect
          .deleteProperty(
            target,
            property
          );
      },
    }
  );
```

This can restrict deletion.

---

## 10. ownKeys Trap

Operations such as:

```js
Object.keys(
  proxy
);
```

can involve the `ownKeys` trap.

Example:

```js
const proxy =
  new Proxy(
    user,
    {
      ownKeys(
        target
      ) {
        return Reflect
          .ownKeys(
            target
          );
      },
    }
  );
```

---

## 11. getOwnPropertyDescriptor Trap

Property enumeration often also depends on property descriptors.

A proxy that customizes `ownKeys` may need to understand:

```js
getOwnPropertyDescriptor
```

because JavaScript enforces invariants between keys and descriptors.

This is where Proxy becomes more advanced.

---

## 12. Proxy Invariants

A Proxy cannot arbitrarily lie about everything.

Example:

If a target has a non-configurable property, certain traps cannot pretend that property does not exist.

JavaScript enforces **Proxy invariants**.

Why?

Because core object rules must remain internally consistent.

---

## 13. Example Invariant

```js
const target = {};

Object.defineProperty(
  target,
  "id",
  {
    value: 1,
    configurable:
      false,
  }
);
```

A proxy cannot safely return an `ownKeys` result that omits `id`.

The engine can throw a `TypeError`.

---

## 14. Reflect Helps Preserve Invariants

A good default pattern:

```js
get(
  target,
  property,
  receiver
) {
  // custom behavior

  return Reflect.get(
    target,
    property,
    receiver
  );
}
```

Do your extra logic, then forward using `Reflect`.

---

## 15. Reflect Is Not a Constructor

You do not write:

```js
new Reflect();
```

`Reflect` is a built-in object containing static methods.

Examples:

```js
Reflect.get(...)
Reflect.set(...)
Reflect.has(...)
Reflect.deleteProperty(...)
Reflect.ownKeys(...)
Reflect.construct(...)
Reflect.apply(...)
```

---

## 16. Reflect vs Object Methods

Many operations overlap conceptually.

Example:

```js
Reflect.has(
  object,
  "name"
);
```

is similar to:

```js
"name" in object;
```

Reflect methods are often convenient inside Proxy traps because names line up closely with intercepted operations.

---

## 17. Reflect.set Returns Success Status

```js
const success =
  Reflect.set(
    object,
    "name",
    "Vikash"
  );
```

Result:

```text
true / false
```

This is useful because `set` traps also expect boolean success semantics.

---

## 18. Reflect.deleteProperty

Instead of:

```js
delete object.name;
```

you can use:

```js
Reflect.deleteProperty(
  object,
  "name"
);
```

It returns a boolean indicating success.

---

## 19. Reflect.apply

Call a function:

```js
function greet(
  greeting
) {
  return (
    greeting +
    " " +
    this.name
  );
}
```

Use:

```js
Reflect.apply(
  greet,
  {
    name:
      "Vikash",
  },
  [
    "Hello",
  ]
);
```

Equivalent in spirit to:

```js
greet.apply(
  thisArg,
  args
);
```

---

## 20. Function Proxy with apply Trap

Functions can be proxied too.

```js
function add(
  a,
  b
) {
  return a + b;
}

const proxy =
  new Proxy(
    add,
    {
      apply(
        target,
        thisArg,
        args
      ) {
        console.log(
          "Called with",
          args
        );

        return Reflect.apply(
          target,
          thisArg,
          args
        );
      },
    }
  );
```

Then:

```js
proxy(2, 3);
```

is intercepted.

---

## 21. construct Trap

Constructor calls can be intercepted.

```js
class User {
  constructor(
    name
  ) {
    this.name =
      name;
  }
}

const ProxiedUser =
  new Proxy(
    User,
    {
      construct(
        target,
        args,
        newTarget
      ) {
        console.log(
          "Creating user"
        );

        return Reflect
          .construct(
            target,
            args,
            newTarget
          );
      },
    }
  );
```

Then:

```js
new ProxiedUser(
  "Vikash"
);
```

uses the trap.

---

## 22. Revocable Proxy

JavaScript can create a proxy that can later be disabled.

```js
const {
  proxy,
  revoke,
} =
  Proxy.revocable(
    target,
    handler
  );
```

Use:

```js
proxy.name;
```

Later:

```js
revoke();
```

Further operations on the proxy throw.

---

## 23. Use Case — Logging

```js
function createLoggedObject(
  target
) {
  return new Proxy(
    target,
    {
      get(
        target,
        property,
        receiver
      ) {
        console.log(
          "GET",
          property
        );

        return Reflect.get(
          target,
          property,
          receiver
        );
      },
    }
  );
}
```

Useful for debugging or instrumentation.

---

## 24. Use Case — Reactive Systems

Reactive libraries can intercept property access and mutation.

Conceptually:

```text
property read
↓
track dependency

property write
↓
notify dependents
```

Proxy makes this style of reactivity possible.

Vue 3 is a well-known example of Proxy-based reactivity.

---

## 25. Proxy Does Not Automatically Deep-Proxy

```js
const target = {
  profile: {
    city:
      "Bengaluru",
  },
};
```

If only `target` is proxied, accessing:

```js
proxy.profile.city
```

does not automatically mean the nested `profile` object is also proxied.

Deep reactivity requires recursively wrapping nested objects or another strategy.

---

## 26. Identity Consideration

```js
proxy === target
```

is:

```text
false
```

They are different object identities.

This can matter with:

- Maps
- Sets
- equality checks
- APIs expecting the original object

---

## 27. Private Class Fields and Proxy

Private fields can create surprising behavior because access requires the correct branded class instance.

A method called with a Proxy as `this` may fail to access:

```js
#privateField
```

unless binding/forwarding is handled carefully.

This is an advanced edge case.

---

## 28. Performance Consideration

Proxy adds interception overhead.

Do not use Proxy for ordinary objects when:

- direct properties are enough
- getters/setters are enough
- explicit functions are clearer

Use it when dynamic interception provides real value.

---

## 29. Debugging Consideration

Proxy behavior can make code less obvious.

A simple line:

```js
user.name
```

may trigger arbitrary code inside a `get` trap.

This power should be used carefully in application architecture.

---

## 30. Proxy vs Getters/Setters

Getter/setter:

```js
const user = {
  get name() {
    // ...
  },

  set name(value) {
    // ...
  },
};
```

Proxy:

```text
can intercept many properties
and many operation types dynamically
```

Getters/setters are simpler when only specific properties need custom behavior.

---

## 31. React Connection

React application code rarely needs Proxy directly.

But Proxies appear in:

- reactive state libraries
- validation libraries
- ORM/data layers
- observable systems
- instrumentation

Understanding Proxy helps you understand advanced library internals.

---

## Interview Questions

### What is a Proxy?

An object wrapper that can intercept fundamental operations performed on a target object.

### What is a trap?

A handler method such as `get`, `set`, `has`, or `deleteProperty` that intercepts an operation.

### Why use Reflect inside Proxy traps?

Reflect provides standard forwarding behavior with semantics closely matching the intercepted operations.

### Can Proxy violate all object rules?

No. JavaScript enforces Proxy invariants.

---

## Key Takeaways

- Proxy intercepts operations on objects/functions.
- Handler methods are called traps.
- Common traps include `get`, `set`, `has`, `deleteProperty`, `apply`, and `construct`.
- Reflect provides standard forwarding operations.
- Proxy traps must respect language invariants.
- Proxies have their own identity.
- Nested objects are not automatically proxied.
- Proxy is powerful but can make behavior less explicit.
