# Lesson 47 — `apply()`

`apply()` is very similar to `call()`.

Both:

- explicitly provide `this`
- invoke the function immediately

The main difference is **how arguments are supplied**.

---

# 1. Syntax

```js
functionName.apply(
  thisArg,
  argsArray
);
```

Compare:

```js
fn.call(
  obj,
  arg1,
  arg2
);
```

with:

```js
fn.apply(
  obj,
  [arg1, arg2]
);
```

Mental model:

```text
call
→ arguments individually

apply
→ arguments grouped in an array-like value
```

---

# 2. Basic Example

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
};

showName.apply(user);
```

Output:

```text
Vikash
```

Like `call()`:

```text
this = user
function executes immediately
```

---

# 3. apply() with Arguments

```js
function introduce(
  role,
  city
) {
  console.log(
    `${this.name} is a ${role} from ${city}`
  );
}

const user = {
  name: "Vikash",
};

introduce.apply(
  user,
  [
    "Developer",
    "Bengaluru",
  ]
);
```

Output:

```text
Vikash is a Developer from Bengaluru
```

Inside:

```text
this = user
role = "Developer"
city = "Bengaluru"
```

---

# 4. call() vs apply()

Using `call()`:

```js
introduce.call(
  user,
  "Developer",
  "Bengaluru"
);
```

Using `apply()`:

```js
introduce.apply(
  user,
  [
    "Developer",
    "Bengaluru",
  ]
);
```

The function receives the same arguments.

Only the argument-passing syntax differs.

---

# 5. Why Was apply() Useful?

Suppose arguments already exist in an array:

```js
const details = [
  "Developer",
  "Bengaluru",
];
```

With `apply()`:

```js
introduce.apply(
  user,
  details
);
```

Historically, this was especially useful when you already had a collection of arguments.

---

# 6. Modern Spread Syntax

Modern JavaScript can often replace the argument-spreading use case of `apply()`.

Instead of:

```js
introduce.apply(
  user,
  details
);
```

you can write:

```js
introduce.call(
  user,
  ...details
);
```

This does **not** make `apply()` useless.

You should still understand it because:

- it appears in existing code
- it is common in interviews
- it explains explicit binding
- some APIs/array-like argument scenarios may use it

---

# 7. Classic Math.max Example

Without spread:

```js
const numbers = [
  10,
  50,
  20,
  80,
];

const max =
  Math.max.apply(
    null,
    numbers
  );

console.log(max);
```

Output:

```text
80
```

Why `null`?

`Math.max` does not depend on a meaningful `this` value for this use.

Modern equivalent:

```js
Math.max(...numbers);
```

This is usually clearer today.

---

# 8. Method Borrowing with apply()

```js
const user1 = {
  name: "Vikash",

  introduce(role, city) {
    console.log(
      `${this.name} - ${role} - ${city}`
    );
  },
};

const user2 = {
  name: "Rahul",
};

user1.introduce.apply(
  user2,
  ["Designer", "Pune"]
);
```

Output:

```text
Rahul - Designer - Pune
```

The method is borrowed and executed with:

```text
this = user2
```

---

# 9. apply() Executes Immediately

```js
function greet(greeting) {
  console.log(
    `${greeting}, ${this.name}`
  );
}

const user = {
  name: "Vikash",
};

greet.apply(
  user,
  ["Hello"]
);
```

The function runs immediately.

Like `call()`, `apply()` does not create a reusable bound function.

That is the job of `bind()`.

---

# 10. apply() Is Not Permanent

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
};

const admin = {
  name: "Admin",
};

showName.apply(user);
showName.apply(admin);
```

Output:

```text
Vikash
Admin
```

Each invocation can explicitly supply a different `this`.

---

# 11. apply() and Arrow Functions

```js
const arrow = () => {
  console.log(this);
};

arrow.apply({
  name: "Vikash",
});
```

`apply()` cannot replace the arrow's lexical `this`.

Exactly as with `call()`:

```text
arrow
→ no own this
→ explicit rebinding does not replace lexical this
```

---

# 12. call vs apply vs Spread

Suppose:

```js
function sum(a, b, c) {
  return a + b + c;
}

const numbers = [10, 20, 30];
```

### call

```js
sum.call(
  null,
  10,
  20,
  30
);
```

### apply

```js
sum.apply(
  null,
  numbers
);
```

### Modern spread

```js
sum(...numbers);
```

If no special `this` is required, spread is often the cleanest modern solution.

If explicit `this` is also required:

```js
sum.call(
  obj,
  ...numbers
);
```

is another modern option.

---

# 13. Easy Memory Trick

Do not rely only on tricks, but this can help initially:

```text
call
→ comma-separated arguments

apply
→ array-like argument collection
```

The deeper distinction is:

```text
call(obj, a, b)

apply(obj, [a, b])
```

Both execute immediately.

---

# 14. Interview Comparison

| Feature | call() | apply() |
| --- | --- | --- |
| Explicitly sets `this` | Yes | Yes |
| Executes immediately | Yes | Yes |
| Arguments | Individually | Array-like collection |
| Returns new bound function | No | No |
| Can rebind arrow `this` | No | No |

---

# Interview Answer

**What is the difference between `call()` and `apply()`?**

Both immediately invoke a function with an explicitly supplied `this`. With `call()`, arguments are passed individually; with `apply()`, they are provided as an array-like argument list.

---

# Key Takeaways

- `apply()` provides explicit binding.
- It executes immediately.
- Its first argument is the `thisArg`.
- Function arguments are supplied as an array-like collection.
- `call()` and `apply()` mainly differ in argument passing.
- Modern spread syntax replaces many old argument-spreading uses of `apply()`.
- `apply()` remains important for understanding existing JavaScript and interviews.
- It does not permanently bind a function.
- It cannot rebind arrow-function `this`.
