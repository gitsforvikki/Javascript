# Lesson 44 — `this` in Arrow Functions

Lessons 42–43 established the most important rule for **regular functions**:

> For a regular function, first inspect **how the function is called**.

Arrow functions are different.

> **An arrow function does not create its own `this`. It uses `this` from the surrounding lexical context.**

This difference is the key to understanding almost every regular-function vs arrow-function `this` question.

---

## 1. Regular Function vs Arrow Function

Regular function:

```js
function show() {
  console.log(this);
}
```

Its `this` is determined by the applicable invocation/binding rule.

Arrow function:

```js
const show = () => {
  console.log(this);
};
```

The arrow function does **not** receive a new `this` because of how it is called.

Instead, it closes over the surrounding `this`.

Mental model:

```text
Regular function
      ↓
creates/receives its own this
according to invocation rules

Arrow function
      ↓
NO own this
      ↓
uses surrounding lexical this
```

---

# 2. Why "Lexical this"?

You already learned lexical scope.

A normal variable inside an inner function can come from an outer lexical environment:

```js
function outer() {
  const name = "Vikash";

  function inner() {
    console.log(name);
  }

  inner();
}
```

Arrow-function `this` follows a similar **lexical** idea.

It does not ask:

```text
Who called me?
```

for its own `this`.

Instead:

```text
Arrow function
      ↓
Where was I created?
      ↓
Use this from surrounding context
```

---

# 3. Arrow Inside an Object — Common Confusion

Consider:

```js
const user = {
  name: "Vikash",

  showName: () => {
    console.log(this?.name);
  },
};

user.showName();
```

Many developers expect:

```text
Vikash
```

because the call is:

```js
user.showName();
```

But this is an **arrow function**.

The normal regular-function method rule does not give the arrow its own `this`.

The object literal itself does not create a `this` binding for the arrow to capture.

So:

```text
user.showName()
       ↓
arrow function
       ↓
does not create its own this
       ↓
looks to surrounding lexical this
```

In an ES module, top-level `this` is `undefined`.

Therefore this pattern should generally **not** be used when a method needs the object as `this`.

Prefer:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};
```

---

# 4. Arrow Inside a Regular Method

This is where arrow functions become very useful.

```js
const user = {
  name: "Vikash",

  showName() {
    const inner = () => {
      console.log(this.name);
    };

    inner();
  },
};

user.showName();
```

Trace it carefully.

### Step 1 — Regular method

```js
user.showName();
```

Therefore:

```text
showName's this = user
```

### Step 2 — Arrow created inside showName

```js
const inner = () => {
  console.log(this.name);
};
```

The arrow has no own `this`.

It captures the surrounding `this`:

```text
inner arrow
    ↓
surrounding this
    ↓
showName's this
    ↓
user
```

So output is:

```text
Vikash
```

---

# 5. Compare with Nested Regular Function

Regular nested function:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    function inner() {
      console.log(this);
    }

    inner();
  },
};

user.showName();
```

Trace:

```text
user.showName()
→ showName this = user

inner()
→ plain regular-function call
→ inner this = undefined
```

Now arrow version:

```js
const user = {
  name: "Vikash",

  showName() {
    const inner = () => {
      console.log(this.name);
    };

    inner();
  },
};

user.showName();
```

Trace:

```text
user.showName()
→ showName this = user

inner arrow
→ no own this
→ uses showName's this
→ user
```

This comparison is essential.

---

# 6. Arrow Functions in Callbacks

Consider:

```js
const user = {
  name: "Vikash",
  skills: ["JavaScript", "React"],

  showSkills() {
    this.skills.forEach((skill) => {
      console.log(
        this.name,
        skill
      );
    });
  },
};

user.showSkills();
```

Output:

```text
Vikash JavaScript
Vikash React
```

Why?

```text
user.showSkills()
      ↓
showSkills this = user
      ↓
arrow callback created
      ↓
arrow captures surrounding this
      ↓
this = user
```

This is one reason arrow callbacks are convenient.

---

# 7. setTimeout Example

Regular callback:

```js
const user = {
  name: "Vikash",

  showLater() {
    setTimeout(function () {
      console.log(this?.name);
    }, 1000);
  },
};
```

The timer API invokes that regular callback according to the host's callback behavior. It does not automatically inherit `showLater`'s `this`.

Arrow version:

```js
const user = {
  name: "Vikash",

  showLater() {
    setTimeout(() => {
      console.log(this.name);
    }, 1000);
  },
};

user.showLater();
```

The arrow captures `showLater`'s `this`.

Since:

```js
user.showLater();
```

gives:

```text
showLater this = user
```

the arrow also sees:

```text
this = user
```

Output:

```text
Vikash
```

---

# 8. Arrow Function Does Not Care About Its Own Call Site for this

Consider:

```js
const obj = {
  name: "Object",
};

const arrow = () => {
  console.log(this);
};

obj.arrow = arrow;

obj.arrow();
```

A regular function invoked as:

```js
obj.method();
```

would normally receive `obj` as `this`.

But an arrow function does not.

Assigning an arrow to an object does not create a new `this` for it.

```text
arrow created
    ↓
captures surrounding this

later:
obj.arrow()
    ↓
call site cannot replace arrow's lexical this
```

---

# 9. call(), apply() and bind() Cannot Rebind Arrow this

Consider:

```js
const user = {
  name: "Vikash",
};

const arrow = () => {
  console.log(this?.name);
};

arrow.call(user);
```

`call(user)` cannot give the arrow a new `this`.

Similarly:

```js
arrow.apply(user);
```

and:

```js
const bound = arrow.bind(user);
bound();
```

do not replace the arrow's lexical `this`.

Why?

Because:

> Arrow functions do not have their own `this` binding to rebind.

We will study `call`, `apply`, and `bind` deeply in Lessons 46–48.

---

# 10. Arrow Functions Cannot Be Constructors

Regular constructor-style function:

```js
function User(name) {
  this.name = name;
}

const user = new User("Vikash");
```

But:

```js
const User = (name) => {
  this.name = name;
};

const user = new User("Vikash");
```

throws a `TypeError`.

Arrow functions are not constructible and cannot be used with `new`.

This connects directly to the fact that they do not provide constructor-style `this`.

---

# 11. Arrow Functions Do Not Have Their Own arguments

Regular function:

```js
function test() {
  console.log(arguments);
}

test(10, 20);
```

A regular function has an `arguments` object.

Arrow functions do not create their own `arguments`.

Use rest parameters instead:

```js
const test = (...args) => {
  console.log(args);
};

test(10, 20);
```

Output:

```text
[10, 20]
```

---

# 12. When Should You Use an Arrow?

Arrow functions are excellent when you want:

- short callback syntax
- lexical `this`
- functional transformations
- event-handler wrappers
- array callbacks

Example:

```js
numbers.map((number) => {
  return number * 2;
});
```

They are especially useful inside a regular method when a callback should use the method's `this`.

---

# 13. When Should You Avoid Arrow Methods?

Avoid this when the method needs dynamic receiver-based `this`:

```js
const user = {
  name: "Vikash",

  showName: () => {
    console.log(this.name);
  },
};
```

Prefer:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};
```

because `showName` should receive `user` when called as:

```js
user.showName();
```

---

# 14. React Connection

Arrow functions are extremely common in React:

```jsx
function Counter() {
  const handleClick = () => {
    console.log("clicked");
  };

  return (
    <button onClick={handleClick}>
      Click
    </button>
  );
}
```

Modern function components generally do not use component `this`, but arrows remain useful because they:

- provide concise callbacks
- capture lexical variables through closures
- do not introduce another dynamic `this`

Example:

```jsx
items.map((item) => (
  <Item
    key={item.id}
    item={item}
  />
));
```

---

# 15. Regular vs Arrow — Core Comparison

| Behavior | Regular Function | Arrow Function |
| --- | --- | --- |
| Own `this` behavior | Yes, based on invocation rules | No |
| `this` from call site | Usually important | Does not create/rebind `this` |
| Lexical `this` | No | Yes |
| `call/apply/bind` can set `this` | Yes | No |
| Can use `new` | If constructible | No |
| Own `arguments` | Yes | No |
| Good object method needing receiver `this` | Yes | Usually no |
| Good nested callback needing outer `this` | Can require binding | Yes |

---

# 16. Interview Tracing

```js
const user = {
  name: "Vikash",

  regular() {
    const arrow = () => {
      console.log(this.name);
    };

    arrow();
  },
};

user.regular();
```

Trace:

```text
regular is a regular method
        ↓
user.regular()
        ↓
regular this = user
        ↓
arrow created inside regular
        ↓
arrow has no own this
        ↓
captures regular's this
        ↓
this = user
        ↓
"Vikash"
```

---

# Interview Answers

**How is `this` different in an arrow function?**

An arrow function does not create its own `this`; it uses `this` from its surrounding lexical context.

**Can `call()`, `apply()`, or `bind()` change an arrow function's `this`?**

No. They cannot replace its lexical `this`.

**Should arrow functions be used as object methods when the method needs the object as `this`?**

Usually no. A regular method is appropriate when receiver-based `this` is required.

---

# Key Takeaways

- Arrow functions do not have their own `this`.
- They use `this` from the surrounding lexical context.
- Regular functions and arrows must not be analyzed with the same `this` rule.
- An arrow stored as an object property does not automatically get that object as `this`.
- Arrow callbacks are useful when they need the surrounding method's `this`.
- `call`, `apply`, and `bind` cannot rebind arrow-function `this`.
- Arrow functions cannot be used as constructors.
- Arrow functions do not create their own `arguments`.
