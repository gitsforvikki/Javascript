# Lesson 42 — Understanding `this`

The keyword `this` becomes confusing when we try to memorize many outputs instead of first understanding **what decides its value**.

For a **regular function**, the most useful starting rule is:

> **Do not ask where the function was written. First ask: how is the function being called?**

`this` is a special value available during function execution. It is **not automatically the function itself**, and it is **not permanently attached to the object where a function was originally stored**.

---

## 1. Start with a Method Call

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

user.showName();
```

Trace the call:

```text
user.showName()
 ↑
 receiver

this → user
```

Therefore `this.name` reads `user.name`, so the output is:

```text
Vikash
```

---

## 2. Definition Location Does Not Fix Regular-Function this

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
  showName,
};

user.showName();
```

Although `showName` was defined outside the object, the invocation is `user.showName()`.

So for this call:

```text
this = user
```

The **call site** matters.

---

## 3. Same Function, Different this

```js
function introduce() {
  console.log(this.name);
}

const user1 = {
  name: "Vikash",
  introduce,
};

const user2 = {
  name: "Rahul",
  introduce,
};

user1.introduce();
user2.introduce();
```

Output:

```text
Vikash
Rahul
```

The function is the same:

```js
console.log(
  user1.introduce === user2.introduce
); // true
```

But the receivers differ:

```text
user1.introduce() → this = user1
user2.introduce() → this = user2
```

So `this` is not permanently stored inside a regular function.

---

## 4. this vs Lexical Scope

Do not mix these two mechanisms.

Normal identifiers use **lexical scope**:

```js
const value = "global";

function outer() {
  const value = "outer";

  function inner() {
    console.log(value);
  }

  inner();
}

outer();
```

`inner` finds `value` from where it was **defined**.

Regular-function `this` works differently:

```text
Normal variable
→ where was the code defined?
→ lexical scope chain

Regular-function this
→ how was the function invoked?
→ binding/invocation rule
```

This distinction is extremely important.

---

## 5. Nested Objects

```js
const developer = {
  name: "Vikash",

  profile: {
    name: "Frontend Developer",

    showName() {
      console.log(this.name);
    },
  },
};

developer.profile.showName();
```

The receiver of `showName()` is `developer.profile`.

Therefore:

```text
this = developer.profile
```

Output:

```text
Frontend Developer
```

Do not automatically choose the outermost object.

---

## 6. Detached Method

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this);
  },
};

const fn = user.showName;

fn();
```

Compare the calls:

```text
user.showName()
→ receiver = user

fn()
→ no receiver object
```

Assigning `user.showName` to `fn` copies the **function value**. It does not permanently carry `user` as its `this`.

For a plain regular-function call in strict mode:

```text
this = undefined
```

---

## 7. Strict Mode and Old Browser Examples

```js
"use strict";

function test() {
  console.log(this);
}

test();
```

Output:

```text
undefined
```

In a classic browser script running without strict mode, a plain regular-function call may substitute the global object (`window`).

So do **not** memorize:

> plain function means window.

Modern JavaScript modules use strict semantics, and React/Next.js code is module-based.

A safer rule is:

> A plain regular-function call has no receiver. In strict mode its `this` is `undefined`.

---

## 8. Top-Level this Is a Separate Topic

Do not confuse:

```js
console.log(this);
```

with:

```js
function test() {
  console.log(this);
}

test();
```

Top-level `this` depends on the environment/module system. Function `this` depends on function type and invocation rules.

For example, top-level `this` in an ES module is `undefined`.

---

## 9. The Complete Question Checklist

Whenever you see `this`, ask:

```text
1. Regular function or arrow function?
2. What is the exact call site?
3. Is there a receiver object?
4. Is call/apply/bind being used?
5. Is the function called with new?
6. Is an API controlling callback invocation?
```

We will learn each case separately.

For now, master this:

```text
const obj = {
  method() {
    console.log(this);
  }
};

obj.method();
↓
this = obj
```

---

## 10. React Connection

Modern React function components normally do not use `this`:

```jsx
function UserCard({ user }) {
  return <h2>{user.name}</h2>;
}
```

But `this` still matters for JavaScript interviews, objects, classes, callbacks, libraries and older class-based React code.

---

## Common Mistakes

**Wrong:** `this` means the current function.

**Wrong:** `this` always means `window`.

**Wrong:** a regular function permanently remembers the object it came from.

**Wrong:** regular-function `this` follows lexical scope like normal variables.

---

## Interview Answer

**What is `this` in JavaScript?**

`this` is a special value available during function execution. Its value depends on the function type and the applicable invocation/binding rule. For ordinary regular functions, the call site is the key starting point.

---

## Key Takeaways

- `this` is not the function itself.
- Regular-function `this` is primarily call-site dependent.
- `object.method()` normally makes `object` the receiver.
- The same function can execute with different `this` values.
- Detaching a method removes its original receiver from the later call.
- Strict-mode plain calls have `this === undefined`.
- Lexical scope and regular-function `this` are different mechanisms.
- Arrow functions will use a different rule.
