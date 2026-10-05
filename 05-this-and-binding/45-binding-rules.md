# Lesson 45 — Implicit and Explicit Binding

Now we can organize the different `this` cases into **binding rules**.

For regular functions, instead of memorizing dozens of examples, identify which rule is controlling `this`.

This lesson focuses on:

1. default binding
2. implicit binding
3. explicit binding

We will introduce explicit binding conceptually here and study `call()`, `apply()`, and `bind()` individually in Lessons 46–48.

---

# 1. What Does "Binding this" Mean?

Binding answers:

> Which value/object will be used as `this` when this regular function executes?

Example:

```js
function showName() {
  console.log(this.name);
}
```

The function alone does not tell us the final `this`.

Different calls can bind it differently:

```js
user.showName();

showName();

showName.call(admin);
```

Same function logic can execute with different `this` values.

---

# 2. Rule 1 — Default Binding

Consider:

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis();
```

This is a plain function call.

No receiver object and no explicit binding are supplied.

In strict mode:

```text
this = undefined
```

This is commonly called **default binding**.

Conceptually:

```text
showThis()
    ↓
no receiver
no call/apply/bind
no new
    ↓
default binding
    ↓
strict mode → undefined
```

In sloppy/non-strict ordinary calls, the global object may be substituted.

---

# 3. Rule 2 — Implicit Binding

Consider:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

user.showName();
```

The object used as the receiver implicitly supplies `this`.

```text
user.showName()
 ↑
 receiver
    ↓
implicit binding
    ↓
this = user
```

This is called **implicit binding** because you did not explicitly write:

```text
this = user
```

The method-style invocation supplies it.

---

# 4. Implicit Binding Is About the Call, Not Ownership

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

Even though the function was created outside the object:

```text
user.showName()
→ implicit binding
→ this = user
```

Again:

> The object does not need to be the place where the function was originally defined.

---

# 5. Last Receiver in a Property Chain

```js
const company = {
  name: "TechCorp",

  employee: {
    name: "Vikash",

    showName() {
      console.log(this.name);
    },
  },
};

company.employee.showName();
```

The relevant receiver is:

```js
company.employee
```

So:

```text
this = company.employee
```

Output:

```text
Vikash
```

Do not automatically bind `this` to the first object in a property chain.

---

# 6. Losing Implicit Binding

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this?.name);
  },
};

const fn = user.showName;

fn();
```

Original call possibility:

```text
user.showName()
→ implicit binding
→ this = user
```

Actual later call:

```text
fn()
→ no receiver
→ default binding
→ this = undefined
```

This is called **implicit binding loss**.

---

# 7. Callback Binding Loss

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this?.name);
  },
};

function execute(callback) {
  callback();
}

execute(user.showName);
```

The expression:

```js
user.showName
```

produces the function value.

Inside `execute`:

```js
callback();
```

is a plain call.

So implicit binding has been lost.

```text
user.showName
      ↓
function passed
      ↓
callback()
      ↓
default binding
```

---

# 8. Rule 3 — Explicit Binding

Sometimes we do not want to depend on the call site's receiver.

We want to explicitly say:

> Execute this function with this specific object as `this`.

JavaScript provides:

```text
call()
apply()
bind()
```

Example with `call()`:

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
};

showName.call(user);
```

Here we explicitly specify:

```text
this = user
```

So output is:

```text
Vikash
```

---

# 9. Implicit vs Explicit Binding

Implicit:

```js
user.showName();
```

Mental model:

```text
receiver at call site
       ↓
this = user
```

Explicit:

```js
showName.call(user);
```

Mental model:

```text
explicitly supplied thisArg
       ↓
this = user
```

Both can produce the same `this`, but through different mechanisms.

---

# 10. Why Explicit Binding Exists

Suppose:

```js
function introduce(role) {
  console.log(
    `${this.name} - ${role}`
  );
}

const user = {
  name: "Vikash",
};
```

The function is not stored on `user`.

But we can still execute it with `user` as `this`:

```js
introduce.call(
  user,
  "Developer"
);
```

Output:

```text
Vikash - Developer
```

Explicit binding lets a function operate with a chosen context without first making it an object property.

---

# 11. call vs apply vs bind — Preview

All three can be used with regular functions to explicitly control `this`, but they are not identical.

### call

```js
fn.call(
  thisArg,
  arg1,
  arg2
);
```

Calls the function immediately.

### apply

```js
fn.apply(
  thisArg,
  [arg1, arg2]
);
```

Also calls immediately, but receives arguments in an array-like collection.

### bind

```js
const boundFn =
  fn.bind(thisArg);
```

Returns a new bound function for later execution.

Quick mental model:

```text
call
→ bind this + execute now

apply
→ bind this + execute now
→ arguments grouped

bind
→ create a new bound function
→ execute later
```

Do not memorize all details yet. The next three lessons cover them individually.

---

# 12. Explicit Binding Overrides Ordinary Implicit Binding

Consider:

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
  showName,
};

const admin = {
  name: "Admin",
};

user.showName.call(admin);
```

If we only looked at:

```text
user.showName
```

we might expect `user`.

But the invocation uses:

```js
.call(admin)
```

so the explicit binding chooses:

```text
this = admin
```

Output:

```text
Admin
```

This introduces an important idea:

> When multiple apparent rules are present, some binding rules have higher priority.

---

# 13. Binding Priority — Initial Mental Model

For regular functions, a useful priority model is:

```text
new binding
    ↓ higher priority

explicit binding
call / apply / bind
    ↓

implicit binding
object.method()
    ↓

default binding
plainFunction()
```

We have not studied `new` yet, so for now focus on:

```text
explicit
   >
implicit
   >
default
```

Example:

```js
user.show.call(admin);
```

Explicit `admin` wins over the apparent implicit `user` receiver.

We will refine the full priority model after constructors and `new`.

---

# 14. Arrow Functions Are Outside This Dynamic Binding Model

This priority discussion mainly concerns regular functions.

Consider:

```js
const arrow = () => {
  console.log(this);
};

arrow.call({
  name: "Vikash",
});
```

The object passed to `call` does not become the arrow's `this`.

Why?

```text
arrow
 ↓
no own this
 ↓
lexical this
```

There is no dynamic arrow-function `this` binding for `call` to replace.

So always perform the first check:

```text
Regular or arrow?
```

before applying binding rules.

---

# 15. A Common Interview Problem

```js
"use strict";

function show() {
  console.log(this?.name);
}

const user = {
  name: "User",
  show,
};

const admin = {
  name: "Admin",
};

const fn = user.show;

user.show();
fn();
show.call(admin);
```

Analyze each separately.

### 1

```js
user.show();
```

```text
implicit binding
this = user
output = User
```

### 2

```js
fn();
```

```text
default binding
strict mode
this = undefined
output = undefined
```

### 3

```js
show.call(admin);
```

```text
explicit binding
this = admin
output = Admin
```

Same underlying function, three different results.

---

# 16. Practical Debugging Method

When `this` gives an unexpected result, write the exact call:

```text
What function?
      ↓
Regular or arrow?
      ↓
How is it invoked?
      ↓
new?
      ↓
call/apply/bind?
      ↓
object.method()?
      ↓
plain call?
      ↓
API callback?
```

Do not inspect only the object definition.

The invocation is often where the answer lives.

---

# 17. React / Application Connection

Modern React function components rarely rely on dynamic `this`, but binding knowledge is still useful when working with:

- JavaScript classes
- SDKs and libraries
- legacy React class components
- object-oriented utilities
- event/callback APIs
- borrowed methods
- interview problems

Older React class code often contained:

```js
this.handleClick =
  this.handleClick.bind(this);
```

After Lessons 46–48, you will understand exactly why that pattern existed.

---

# 18. Binding Rules Summary

```text
REGULAR FUNCTION

1. Plain call
   fn()
   ↓
   default binding

2. Method call
   obj.fn()
   ↓
   implicit binding
   this = obj

3. Explicit call
   fn.call(obj)
   fn.apply(obj)
   bound = fn.bind(obj)
   ↓
   explicit binding

4. Constructor call
   new Fn()
   ↓
   new binding
   [Lesson 49]
```

Arrow:

```text
Arrow function
↓
does not use these rules to create its own this
↓
lexical this
```

---

# Interview Answers

**What is implicit binding?**

Implicit binding occurs when a regular function is invoked as a method of an object, such as `obj.method()`, causing the receiver object to be used as `this`.

**What is explicit binding?**

Explicit binding means deliberately specifying a regular function's `this`, typically with `call()`, `apply()`, or `bind()`.

**What is implicit binding loss?**

It occurs when a method is detached or passed as a callback and later invoked without its original receiver.

**Which has higher priority: implicit or explicit binding?**

Explicit binding takes priority over ordinary implicit binding.

---

# Key Takeaways

- Binding determines the value used as `this` for regular-function execution.
- Plain calls use default binding.
- `object.method()` uses implicit binding.
- Extracting a method can cause implicit binding loss.
- `call`, `apply`, and `bind` provide explicit binding.
- Explicit binding has higher priority than ordinary implicit binding.
- `new` introduces another higher-priority rule that we will study later.
- Arrow functions do not participate in dynamic `this` binding because they use lexical `this`.
- Always identify regular vs arrow before applying binding rules.
