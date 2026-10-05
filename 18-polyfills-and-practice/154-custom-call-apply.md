# Lesson 154 — Implement Custom `call` and `apply`

`call()` and `apply()` let you invoke a function with an explicitly chosen `this` value.

These are highly important interview questions because they test:

- `this`
- explicit binding
- function invocation
- arguments
- temporary property techniques
- primitive/null handling

---

## 1. Native call()

```js
function greet(
  greeting,
  punctuation
) {
  return (
    greeting +
    " " +
    this.name +
    punctuation
  );
}

const user = {
  name: "Vikash",
};

greet.call(
  user,
  "Hello",
  "!"
);
```

Result:

```text
Hello Vikash!
```

---

## 2. Native apply()

```js
greet.apply(
  user,
  [
    "Hello",
    "!",
  ]
);
```

Same result.

Main difference:

```text
call
→ arguments passed individually

apply
→ arguments passed as array-like/list
```

---

## 3. Core Polyfill Idea

If a function becomes a temporary method of an object:

```js
user.temp =
  greet;

user.temp();
```

then during that method call:

```js
this === user
```

This is the classic implementation trick.

---

## 4. Avoid Property Collision

Do not use:

```js
context.fn =
  this;
```

because `fn` may already exist.

Use a unique Symbol:

```js
const key =
  Symbol(
    "temporaryFunction"
  );
```

---

## 5. Basic customCall

```js
Function.prototype.customCall =
  function (
    context,
    ...args
  ) {
    const target =
      context == null
        ? globalThis
        : Object(context);

    const key =
      Symbol(
        "fn"
      );

    target[key] =
      this;

    const result =
      target[key](
        ...args
      );

    delete target[key];

    return result;
  };
```

---

## 6. Why Function.prototype?

We want usage like:

```js
greet.customCall(
  user,
  "Hello"
);
```

So the method must live on:

```js
Function.prototype
```

Inside the custom method:

```js
this
```

refers to the function being invoked.

---

## 7. Validate the Receiver

A safer implementation can check:

```js
if (
  typeof this !==
  "function"
) {
  throw new TypeError(
    "customCall must be called on a function"
  );
}
```

Normally this method is inherited by functions, but the check makes intent explicit.

---

## 8. Null and Undefined Context

Classic non-strict `call` semantics often map:

```text
null / undefined
→ global object
```

A learning polyfill commonly uses:

```js
globalThis
```

Important caveat:

Native strict-mode semantics differ.

A complete spec-level polyfill cannot perfectly reproduce every engine-level `this` behavior using only ordinary JavaScript.

For interviews, explain the limitation.

---

## 9. Primitive Context

```js
function show() {
  return typeof this;
}

show.call(10);
```

In non-strict-style behavior, primitives may be boxed.

So a learning polyfill often uses:

```js
Object(context)
```

for non-null values.

---

## 10. Better customCall with Cleanup

Use `try/finally` so temporary property is removed even if the function throws.

```js
Function.prototype.customCall =
  function (
    context,
    ...args
  ) {
    const target =
      context == null
        ? globalThis
        : Object(context);

    const key =
      Symbol(
        "fn"
      );

    target[key] =
      this;

    try {
      return target[key](
        ...args
      );
    } finally {
      delete target[key];
    }
  };
```

This is much safer.

---

## 11. customApply

```js
Function.prototype.customApply =
  function (
    context,
    args
  ) {
    const target =
      context == null
        ? globalThis
        : Object(context);

    const key =
      Symbol(
        "fn"
      );

    target[key] =
      this;

    const list =
      args == null
        ? []
        : Array.from(
            args
          );

    try {
      return target[key](
        ...list
      );
    } finally {
      delete target[key];
    }
  };
```

---

## 12. Why Array.from(args)?

Native `apply` accepts array-like values, not only actual arrays.

Example:

```js
const args = {
  0: "Hello",
  1: "!",
  length: 2,
};
```

`Array.from()` converts such values into a normal iterable array for spread.

---

## 13. Test customCall

```js
function introduce(
  role
) {
  return (
    this.name +
    " - " +
    role
  );
}

const user = {
  name: "Vikash",
};

introduce.customCall(
  user,
  "Developer"
);
```

Result:

```text
Vikash - Developer
```

---

## 14. Test customApply

```js
introduce.customApply(
  user,
  [
    "Developer",
  ]
);
```

---

## 15. Method Borrowing

```js
const person1 = {
  name: "A",

  greet() {
    return this.name;
  },
};

const person2 = {
  name: "B",
};

person1.greet.call(
  person2
);
```

Result:

```text
B
```

The same function is executed with a different receiver.

---

## 16. call/apply Do Not Permanently Bind

```js
greet.call(
  user
);
```

affects only that invocation.

Next call can use another object.

Permanent binding is what `bind()` is for.

---

## 17. Arrow Function Limitation

```js
const greet =
  () => {
    console.log(
      this
    );
  };
```

Calling:

```js
greet.call(user);
```

does not replace lexical `this` of the arrow function.

`call` and `apply` cannot rebind arrow-function `this`.

---

## 18. Why Symbol Is Better Than Random String

A random string can theoretically collide.

A Symbol creates a unique property key.

```js
const key =
  Symbol();
```

This makes temporary attachment safer.

---

## 19. Polyfill Limitation

The temporary-property technique cannot perfectly reproduce all native behavior for:

- non-extensible objects
- frozen objects
- proxies
- strict-mode primitive/null semantics
- spec-level internal slots

Interview answer should acknowledge:

> This is an educational implementation, not a complete ECMAScript replacement.

That is a strong signal of understanding.

---

## 20. Common Wrong Implementation

```js
Function.prototype.customCall =
  function (
    context
  ) {
    context.fn =
      this;

    return context.fn();
  };
```

Problems:

- property collision
- no arguments
- no cleanup
- crashes on null/undefined
- fails on primitives
- cleanup skipped if function throws

---

## 21. Complexity

Ignoring target function work:

```text
Time:
O(number of arguments)

Extra space:
O(number of arguments)
```

for argument collection/spreading.

---

## Interview Explanation

A strong answer:

> The classic polyfill temporarily stores the function on the target context under a unique Symbol, calls it as a method so `this` becomes that context, then removes the temporary property in a `finally` block. `call` collects arguments individually, while `apply` accepts an array-like argument list.

---

## Key Takeaways

- `call` and `apply` perform explicit `this` binding.
- `call` takes individual arguments.
- `apply` takes an argument list.
- Temporary method invocation creates the desired `this`.
- Use Symbol to avoid key collisions.
- Use `try/finally` for cleanup.
- Arrow-function `this` cannot be rebound.
- Educational polyfills cannot perfectly reproduce all native semantics.
