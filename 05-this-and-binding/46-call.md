# Lesson 46 — `call()`

In Lesson 45, you learned **explicit binding**.

`call()` is the first tool that lets us explicitly tell a regular function:

> **Execute now, and use this value as `this`.**

---

## 1. Basic Syntax

```js
functionName.call(
  thisArg,
  arg1,
  arg2
);
```

The first argument controls `this`.

The remaining arguments are passed to the function normally.

---

## 2. Basic Example

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
};

showName.call(user);
```

Trace:

```text
showName.call(user)
         ↓
explicit binding
         ↓
this = user
         ↓
showName executes immediately
```

Output:

```text
Vikash
```

---

## 3. call() with Function Arguments

```js
function introduce(role, city) {
  console.log(
    `${this.name} is a ${role} from ${city}`
  );
}

const user = {
  name: "Vikash",
};

introduce.call(
  user,
  "Developer",
  "Bengaluru"
);
```

Inside `introduce`:

```text
this = user
role = "Developer"
city = "Bengaluru"
```

Important:

```text
call(
  thisArg,
  argument1,
  argument2,
  ...
)
```

---

## 4. call() Executes Immediately

```js
function greet() {
  console.log(
    `Hello ${this.name}`
  );
}

const user = {
  name: "Vikash",
};

greet.call(user);
```

The function runs at this line.

`call()` does **not** return a permanently bound function.

That distinction becomes important when we study `bind()`.

---

## 5. Reusing One Function with Different Objects

```js
function showProfile() {
  console.log(
    this.name,
    this.role
  );
}

const developer = {
  name: "Vikash",
  role: "Developer",
};

const admin = {
  name: "Rahul",
  role: "Admin",
};

showProfile.call(developer);
showProfile.call(admin);
```

Output:

```text
Vikash Developer
Rahul Admin
```

The function itself is unchanged.

Only the explicitly supplied `this` changes.

---

# 6. Method Borrowing

Suppose one object has a useful method:

```js
const user1 = {
  name: "Vikash",

  introduce() {
    console.log(
      `I am ${this.name}`
    );
  },
};

const user2 = {
  name: "Rahul",
};
```

We can borrow `user1`'s method:

```js
user1.introduce.call(user2);
```

Output:

```text
I am Rahul
```

Why?

```text
function:
user1.introduce

explicit this:
user2
```

The method's original object does not permanently own its `this`.

---

# 7. call() vs Implicit Binding

Implicit:

```js
user.showName();
```

```text
this = user
because user is receiver
```

Explicit:

```js
showName.call(user);
```

```text
this = user
because user was explicitly supplied
```

Both may produce the same `this`, but the binding mechanism differs.

---

# 8. Explicit Binding Wins over Ordinary Implicit Binding

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "User",
  showName,
};

const admin = {
  name: "Admin",
};

user.showName.call(admin);
```

Output:

```text
Admin
```

Although `showName` is accessed through `user`, the actual invocation uses:

```js
.call(admin)
```

So explicit binding controls the function's `this`.

---

# 9. Using call() on a Detached Method

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

const fn = user.showName;
```

A plain call can lose the receiver:

```js
fn();
```

But we can explicitly restore it:

```js
fn.call(user);
```

Now:

```text
this = user
```

---

# 10. call() Does Not Permanently Bind the Function

```js
function showName() {
  console.log(this?.name);
}

const user = {
  name: "Vikash",
};

showName.call(user);
```

This invocation uses `user`.

But that does not modify `showName` permanently.

A later invocation can use another context:

```js
showName.call({
  name: "Rahul",
});
```

Output:

```text
Rahul
```

Think:

```text
call()
→ binding for this invocation
```

not:

```text
call()
→ permanently changes function
```

---

# 11. call() and Arrow Functions

```js
const user = {
  name: "Vikash",
};

const showName = () => {
  console.log(this?.name);
};

showName.call(user);
```

`call()` cannot replace an arrow function's lexical `this`.

Why?

```text
arrow
 ↓
has no own this
 ↓
uses lexical this
```

So explicit `this` binding is meaningful for regular functions, not for rebinding an arrow's `this`.

---

# 12. null and undefined as thisArg

You may see:

```js
fn.call(null);
fn.call(undefined);
```

For strict-mode regular functions, the supplied value is not automatically replaced with the global object.

In sloppy mode, `null` or `undefined` can trigger global-object substitution.

For modern code, avoid relying on sloppy-mode coercion behavior.

---

# 13. Practical Example

```js
function logActivity(action) {
  console.log(
    `${this.username}: ${action}`
  );
}

const currentUser = {
  username: "vikash",
};

logActivity.call(
  currentUser,
  "logged in"
);
```

Output:

```text
vikash: logged in
```

The function remains reusable with other objects.

---

# 14. Interview Output Question

```js
function calculate(a, b) {
  return (
    this.base + a + b
  );
}

const config = {
  base: 10,
};

console.log(
  calculate.call(
    config,
    20,
    30
  )
);
```

Trace:

```text
this = config
this.base = 10
a = 20
b = 30

10 + 20 + 30
= 60
```

Output:

```text
60
```

---

# Interview Answer

**What does `call()` do?**

`call()` invokes a function immediately while explicitly specifying the value to use as `this`. Additional function arguments are supplied individually after the `thisArg`.

---

# Key Takeaways

- `call()` provides explicit `this` binding.
- It invokes the function immediately.
- The first argument is the `thisArg`.
- Remaining arguments are passed individually.
- One function can be reused with many objects.
- `call()` enables method borrowing.
- It can restore a desired context to a detached regular function.
- The binding applies to that invocation; it does not permanently modify the original function.
- `call()` cannot rebind an arrow function's lexical `this`.
