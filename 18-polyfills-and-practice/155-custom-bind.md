# Lesson 155 — Implement Custom `bind`

`bind()` returns a **new function** whose `this` value is fixed to a chosen object.

It can also partially apply arguments.

This is one of the most important JavaScript interview polyfills because correct `bind` behavior involves:

- `this`
- closures
- partial application
- constructor calls
- prototype behavior

---

## 1. Native bind()

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

const user = {
  name: "Vikash",
};

const bound =
  greet.bind(
    user
  );

bound(
  "Hello"
);
```

Result:

```text
Hello Vikash
```

---

## 2. bind Returns a New Function

Important:

```js
const bound =
  greet.bind(
    user
  );
```

does not execute `greet` immediately.

It returns a function for later use.

---

## 3. Partial Application

```js
function calculate(
  a,
  b,
  c
) {
  return (
    this.base +
    a +
    b +
    c
  );
}

const bound =
  calculate.bind(
    {
      base: 10,
    },
    1,
    2
  );

bound(3);
```

Result:

```text
16
```

Arguments supplied to `bind` come before later arguments.

---

## 4. Simple customBind

```js
Function.prototype.customBind =
  function (
    context,
    ...presetArgs
  ) {
    const originalFn =
      this;

    return function (
      ...laterArgs
    ) {
      return originalFn.apply(
        context,
        [
          ...presetArgs,
          ...laterArgs,
        ]
      );
    };
  };
```

This handles normal calls and partial application.

---

## 5. Closure Connection

The returned function remembers:

- `originalFn`
- `context`
- `presetArgs`

through closure.

This is a direct real-world closure use case.

---

## 6. But Constructor Behavior Makes bind Harder

Native bound functions can still be used with:

```js
new
```

Example:

```js
function User(
  name
) {
  this.name =
    name;
}

const BoundUser =
  User.bind(
    {
      ignored: true,
    }
  );

const user =
  new BoundUser(
    "Vikash"
  );
```

When called with `new`:

> the bound `this` context is ignored.

A new instance becomes `this`.

---

## 7. Detect Constructor Invocation

Inside the returned function:

```js
this instanceof boundFunction
```

can help detect whether it was called with `new`.

---

## 8. Better customBind

```js
Function.prototype.customBind =
  function (
    context,
    ...presetArgs
  ) {
    if (
      typeof this !==
      "function"
    ) {
      throw new TypeError(
        "Bind target must be callable"
      );
    }

    const originalFn =
      this;

    function boundFunction(
      ...laterArgs
    ) {
      const isNewCall =
        this instanceof
        boundFunction;

      const thisArg =
        isNewCall
          ? this
          : context;

      return originalFn.apply(
        thisArg,
        [
          ...presetArgs,
          ...laterArgs,
        ]
      );
    }

    if (
      originalFn.prototype
    ) {
      boundFunction.prototype =
        Object.create(
          originalFn.prototype
        );
    }

    return boundFunction;
  };
```

---

## 9. Why Constructor Calls Ignore Bound Context

With:

```js
new BoundUser()
```

JavaScript creates a new object and uses it as `this`.

Native `bind` preserves constructor semantics.

So our polyfill must not force the originally bound context during `new`.

---

## 10. Prototype Connection

Suppose:

```js
User.prototype.sayHi =
  function () {
    return (
      "Hi " +
      this.name
    );
  };
```

A constructed bound instance should still inherit from:

```js
User.prototype
```

That is why prototype handling matters.

---

## 11. Test Normal Binding

```js
function show() {
  return this.name;
}

const bound =
  show.customBind({
    name: "Vikash",
  });

bound();
```

Result:

```text
Vikash
```

---

## 12. Test Partial Application

```js
function sum(
  a,
  b,
  c
) {
  return a + b + c;
}

const add12 =
  sum.customBind(
    null,
    1,
    2
  );

add12(3);
```

Result:

```text
6
```

---

## 13. Test Constructor Call

```js
function User(
  name
) {
  this.name =
    name;
}

User.prototype.sayHi =
  function () {
    return (
      "Hi " +
      this.name
    );
  };

const BoundUser =
  User.customBind(
    {
      name:
        "Wrong"
    }
  );

const user =
  new BoundUser(
    "Vikash"
  );

user.sayHi();
```

Expected:

```text
Hi Vikash
```

Bound context should not replace the new instance.

---

## 14. Return Value with Constructor

If a constructor explicitly returns an object, JavaScript uses that returned object.

A fully spec-accurate bound constructor needs to preserve that behavior too.

The `apply`-based implementation above often gets ordinary constructor logic right through JavaScript's normal function return semantics, but native `bind` has additional internal behavior that cannot be perfectly recreated with plain JavaScript.

---

## 15. Arrow Function Difference

Arrow functions cannot be constructors.

So:

```js
const fn =
  () => {};
```

cannot meaningfully become constructable just because it is bound.

Also, arrow-function lexical `this` cannot be rebound.

---

## 16. bind vs call/apply

```text
call
→ invokes immediately

apply
→ invokes immediately

bind
→ returns new function
```

---

## 17. bind vs Partial Application

`bind` can partially apply leading arguments.

But it also fixes `this`.

A generic partial utility may pre-fill arguments without changing `this`.

---

## 18. Common Wrong Implementation

```js
Function.prototype.customBind =
  function (
    context
  ) {
    return () => {
      return this.call(
        context
      );
    };
  };
```

Problems:

- no argument support
- no partial application
- no constructor behavior
- returned arrow cannot be used with `new`

---

## 19. Practical Interview Version

If interviewer only asks for basic `bind`:

```js
Function.prototype.customBind =
  function (
    context,
    ...presetArgs
  ) {
    const fn =
      this;

    return function (
      ...laterArgs
    ) {
      return fn.apply(
        context,
        [
          ...presetArgs,
          ...laterArgs,
        ]
      );
    };
  };
```

Then say:

> Native `bind` also supports constructor calls, so I would extend it to detect `new` and preserve prototype behavior.

That is a strong interview response.

---

## 20. Complexity

Ignoring original function work:

```text
Creation:
O(number of preset args)

Invocation:
O(total args)
```

---

## Interview Explanation

A strong answer:

> Basic `bind` returns a closure that remembers the original function, context, and preset arguments, then uses `apply` when invoked. A more complete implementation must also detect constructor usage because native bound functions ignore the bound context when called with `new` and preserve prototype inheritance.

---

## Key Takeaways

- `bind` returns a new function.
- It fixes `this` for normal calls.
- It supports partial application.
- Closures preserve context and preset arguments.
- Native bind must support constructor calls.
- Constructor invocation ignores the bound context.
- Prototype behavior matters in an interview-grade implementation.
