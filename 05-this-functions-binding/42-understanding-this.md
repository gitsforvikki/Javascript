# Lesson 42 — Understanding `this` from the Ground Up

The `this` keyword is one of the most confusing parts of JavaScript because its value can change depending on **how a function is called**.

The goal of this lesson is not to memorize many outputs.

The goal is to build one strong mental model that we can reuse in every later lesson.

---

# 1. What is `this`?

Inside a function, `this` is a special value supplied according to the rules of that function invocation.

Example:

```js
function showThis() {
  console.log(this);
}
```

Looking only at the function definition is usually **not enough** to determine the value of `this` for an ordinary function.

We must also inspect **how the function is called**.

---

# 2. The Most Important Rule

For an ordinary function:

> Do not ask only, “Where was this function written?”

Instead ask:

> **How was this function invoked?**

Example:

```js
function introduce() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
  introduce,
};

user.introduce();
```

Look at the call:

```js
user.introduce();
```

The function is invoked as a method of `user`.

Therefore, for this call:

```text
this → user
```

So:

```js
this.name
```

means:

```js
user.name
```

Output:

```text
Vikash
```

---

# 3. Think About the Call Site

Consider:

```js
const developer = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

developer.showName();
```

Trace it:

```text
Function
↓
showName

Call site
↓
developer.showName()

Object used to invoke function
↓
developer

Therefore
↓
this === developer
```

This call-site thinking will remove a lot of confusion.

---

# 4. `this` Is Not the Function Itself

A common misunderstanding is:

```text
this = current function
```

That is incorrect.

Example:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this === user);
  },
};

user.showName();
```

Output:

```text
true
```

Here `this` refers to `user`, not to the `showName` function.

---

# 5. `this` Is Not Automatically the Object Where the Function Was Written

Consider:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

const show = user.showName;
```

Now:

```js
show();
```

The function originally appeared inside `user), but the call is no longer:

```js
user.showName();
```

It is:

```js
show();
```

The invocation changed.

Therefore its `this` behavior changes.

This gives us a crucial rule:

> An ordinary function does not permanently belong to the object from which you obtained it.

---

# 6. Method Call vs Detached Function Call

Compare these two calls.

## Method call

```js
user.showName();
```

Trace:

```text
user.showName()
↑
object before the dot

this → user
```

## Detached call

```js
const show = user.showName;

show();
```

Trace:

```text
show()
↑
plain function call

no user receiver at call site
```

The reference to the function was copied, but the method-style call was not.

This distinction is one of the biggest sources of `this` bugs.

---

# 7. A Function Can Be Used by Different Objects

```js
function showName() {
  console.log(this.name);
}

const user1 = {
  name: "Vikash",
  showName,
};

const user2 = {
  name: "Rahul",
  showName,
};

user1.showName();
user2.showName();
```

Output:

```text
Vikash
Rahul
```

Same function:

```text
showName
```

Different call sites:

```text
user1.showName()
       ↓
this → user1

user2.showName()
       ↓
this → user2
```

This proves that an ordinary function does not have one permanently fixed `this`.

---

# 8. The Object Before the Call Matters

Consider:

```js
const company = {
  name: "OpenTech",

  employee: {
    name: "Vikash",

    showName() {
      console.log(this.name);
    },
  },
};

company.employee.showName();
```

What is `this`?

Look at the immediate receiver of the call:

```text
company.employee.showName()
        ↑
company.employee
```

Therefore:

```js
this === company.employee
```

Output:

```text
Vikash
```

It is **not** automatically the outermost `company` object.

---

# 9. `this` Does Not Follow Lexical Scope for Regular Functions

This is extremely important.

Normal variable lookup uses lexical scope.

```js
const name = "Global";

function outer() {
  const name = "Outer";

  function inner() {
    console.log(name);
  }

  inner();
}

outer();
```

`name` is resolved lexically.

But regular-function `this` follows different rules.

Do not think:

```text
normal variable lookup
=
this lookup
```

They are different concepts.

We will see this clearly in Lesson 43.

---

# 10. `this` in Global Code

The value of `this` at top level depends on the environment and script/module context.

For example, browser classic scripts, browser ES modules, and Node.js modules do not all expose the same top-level `this`.

Therefore avoid memorizing:

> “Global `this` is always window.”

That statement is not generally correct.

The more useful lesson is:

> Always consider the execution environment and whether code is a classic script or module.

---

# 11. Strict Mode Matters

For a plain regular-function call:

```js
function showThis() {
  console.log(this);
}

showThis();
```

The result depends on strictness/environment details.

In strict mode:

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis();
```

For this plain function call:

```text
this → undefined
```

In non-strict classic-script behavior, a plain function call can substitute the global object for `this`.

Modern JavaScript applications frequently use modules, and modules are strict by default.

So for modern reasoning, never depend on accidental global-object `this`.

---

# 12. A Better Mental Algorithm

Whenever you see `this` inside an ordinary function, do this:

```text
Step 1
Find the function containing this.

Step 2
Find how that function is invoked.

Step 3
Classify the call.

Examples:
object.method()
plainFunction()
call/apply/bind
new Constructor()

Step 4
Apply the corresponding this rule.
```

For now we are focusing primarily on:

```text
object.method()
vs
plainFunction()
```

Later lessons will add:

```text
arrow functions
call()
apply()
bind()
new
classes
```

---

# 13. Example — Predict Before Running

```js
const person = {
  name: "Vikash",

  greet() {
    console.log(this.name);
  },
};

person.greet();
```

Trace:

```text
Function
→ greet

Call
→ person.greet()

Receiver
→ person

this
→ person

this.name
→ person.name
→ "Vikash"
```

---

# 14. Change Only the Call

```js
const person = {
  name: "Vikash",

  greet() {
    console.log(this?.name);
  },
};

const greet = person.greet;

greet();
```

The function implementation did not change.

Only the call changed:

```text
Before:
person.greet()

After:
greet()
```

That change is enough to change `this`.

This is the central idea of this entire section.

---

# 15. Why Developers Get Confused

Developers often look at:

```js
const user = {
  name: "Vikash",

  show() {
    console.log(this.name);
  },
};
```

and mentally attach:

```text
this = user forever
```

That is the wrong model.

Instead:

```text
Function definition
does not permanently fix this
for an ordinary function.

Invocation
determines this according
to the applicable call rule.
```

---

# 16. React Connection — Preview

In React, passing functions around is extremely common.

```jsx
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

<button onClick={user.showName}>
  Show
</button>
```

The function is being passed as a callback.

React later invokes the callback; it is not the same expression as directly writing:

```js
user.showName();
```

Therefore relying on method-style `this` can cause confusion.

Modern React function components generally avoid depending heavily on dynamic `this`.

Later lessons will make the callback behavior and arrow-function differences clear.

---

# 17. Do Not Mix `this` with Closures

Closure:

```js
function outer() {
  const name = "Vikash";

  return function () {
    console.log(name);
  };
}
```

Here `name` is found through lexical scope.

`this`:

```js
function showName() {
  console.log(this.name);
}
```

For an ordinary function, `this` is determined using invocation rules.

Remember:

```text
normal variables
→ lexical scope

regular-function this
→ invocation / call-site rules
```

This distinction is essential.

---

# Interview Perspective

## What is `this` in JavaScript?

`this` is a special value whose behavior depends on the type of function and how that function is invoked. For ordinary functions, the call site is usually the key to determining `this`.

## Is `this` determined by where a regular function is defined?

Usually no. Ordinary-function `this` is primarily determined by how the function is called.

## Does a method permanently belong to an object?

No. A function can be extracted and called differently, which can change its `this` value.

---

# Final Mental Model

Do not memorize:

```text
this = object where function exists
```

Memorize:

```text
Regular function
      ↓
look at invocation
      ↓
how was it called?
      ↓
apply this rule
```

For a method-style call:

```text
user.showName()
 ↑
receiver

this → user
```

For a detached/plain call:

```text
const fn = user.showName;

fn()
↑
no user receiver

this is NOT automatically user
```

---

# Key Takeaways

- `this` is a special execution value.
- For ordinary functions, focus on the **call site**.
- `object.method()` usually gives the method `this === object`.
- Extracting a method can remove that receiver relationship.
- The same regular function can receive different `this` values.
- `this` is not ordinary lexical variable lookup.
- Strict mode affects plain-function `this`.
- Arrow functions, explicit binding, and constructors have additional rules that we will study separately.
