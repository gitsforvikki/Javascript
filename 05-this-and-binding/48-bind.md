# Lesson 48 — `bind()`

`bind()` completes the most important explicit-binding trio:

- `call()`
- `apply()`
- `bind()`

The major difference is:

> **`bind()` does not immediately execute the original function. It returns a new function with a bound `this`.**

This makes `bind()` extremely useful for callbacks and detached methods.

---

# 1. Basic Syntax

```js
const boundFunction =
  originalFunction.bind(
    thisArg,
    optionalArg1,
    optionalArg2
  );
```

Then later:

```js
boundFunction();
```

Mental model:

```text
original function
      +
chosen this
      ↓
bind()
      ↓
new bound function
      ↓
execute later
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

const boundShowName =
  showName.bind(user);
```

Nothing has been printed yet.

Now:

```js
boundShowName();
```

Output:

```text
Vikash
```

Why?

```text
showName.bind(user)
       ↓
returns new bound function
       ↓
boundShowName()
       ↓
this = user
```

---

# 3. call vs bind

### call

```js
showName.call(user);
```

```text
bind this
+
execute immediately
```

### bind

```js
const fn =
  showName.bind(user);

fn();
```

```text
create bound function now
+
execute later
```

This is the most important difference.

---

# 4. apply vs bind

`apply()`:

```js
introduce.apply(
  user,
  ["Developer", "Bengaluru"]
);
```

executes immediately.

`bind()`:

```js
const introduceUser =
  introduce.bind(user);

introduceUser(
  "Developer",
  "Bengaluru"
);
```

returns a reusable function first.

---

# 5. Solving Detached Method Problems

Recall:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

const fn = user.showName;

fn();
```

The original receiver is lost.

We can preserve the intended context:

```js
const fn =
  user.showName.bind(user);

fn();
```

Output:

```text
Vikash
```

This is one of the most practical uses of `bind()`.

---

# 6. bind() Creates a New Function

```js
function show() {}

const user = {};

const bound =
  show.bind(user);

console.log(
  bound === show
);
```

Output:

```text
false
```

`bind()` returns a **new bound function object**.

It does not mutate the original function.

---

# 7. Reusable Bound Function

```js
function showName() {
  console.log(this.name);
}

const user = {
  name: "Vikash",
};

const showUser =
  showName.bind(user);

showUser();
showUser();
showUser();
```

Each call uses the bound context:

```text
this = user
```

That is different from `call()`, where you explicitly supply the context for each invocation.

---

# 8. Can call() Change an Already Bound this?

This is an important interview question.

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

const bound =
  showName.bind(user);

bound.call(admin);
```

Output:

```text
Vikash
```

Why not `Admin`?

The function returned by `bind()` has its bound `this`.

A later ordinary `call(admin)` cannot replace that bound `this`.

Conceptually:

```text
showName.bind(user)
       ↓
bound this = user

bound.call(admin)
       ↓
admin cannot replace
the bound this
       ↓
this = user
```

This is sometimes called **hard binding**.

---

# 9. Partial Application with bind()

`bind()` can bind not only `this`, but also initial arguments.

```js
function calculate(
  tax,
  price
) {
  return price +
    price * tax;
}

const addGST =
  calculate.bind(
    null,
    0.18
  );

console.log(
  addGST(1000)
);
```

Conceptually:

```text
calculate(tax, price)

bind:
tax = 0.18

later:
addGST(1000)

price = 1000
```

Result:

```text
1180
```

This is **partial application**.

We will study the broader functional-programming concept later.

---

# 10. Binding Some Arguments and Passing the Rest Later

```js
function introduce(
  greeting,
  role
) {
  console.log(
    `${greeting}, I am ${this.name}, a ${role}`
  );
}

const user = {
  name: "Vikash",
};

const greetUser =
  introduce.bind(
    user,
    "Hello"
  );

greetUser("Developer");
```

Binding stage:

```text
this = user
greeting = "Hello"
```

Later call:

```text
role = "Developer"
```

Output:

```text
Hello, I am Vikash, a Developer
```

---

# 11. bind() with Callbacks

Suppose:

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

function execute(callback) {
  callback();
}
```

This loses the receiver:

```js
execute(user.showName);
```

But:

```js
execute(
  user.showName.bind(user)
);
```

creates a bound function before passing it.

So when `execute` later performs:

```js
callback();
```

the function still uses:

```text
this = user
```

---

# 12. setTimeout with bind()

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

setTimeout(
  user.showName.bind(user),
  1000
);
```

The timer receives a bound function.

When it invokes that callback later, the function retains:

```text
this = user
```

Another common solution is an arrow wrapper:

```js
setTimeout(
  () => user.showName(),
  1000
);
```

These work for different reasons:

```text
bind
→ creates a function with bound this

arrow wrapper
→ later performs user.showName()
→ method call supplies user as receiver
```

---

# 13. React Class Connection

Older React class components commonly needed binding.

Example:

```jsx
class Counter extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      count: 0,
    };

    this.handleClick =
      this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({
      count:
        this.state.count + 1,
    });
  }

  render() {
    return (
      <button
        onClick={this.handleClick}
      >
        Increment
      </button>
    );
  }
}
```

Why bind?

Passing:

```js
this.handleClick
```

passes the function as a callback.

Binding creates a function whose `this` remains the component instance.

Modern function components usually avoid this pattern entirely.

---

# 14. bind() and Arrow Functions

```js
const arrow = () => {
  console.log(this);
};

const bound =
  arrow.bind({
    name: "Vikash",
  });

bound();
```

`bind()` returns a function, but it cannot replace the arrow's lexical `this`.

Why?

```text
arrow
→ no own this
→ lexical this
```

So do not use `bind()` expecting to change arrow-function `this`.

---

# 15. Important new + bind Nuance

You learned that a bound function normally keeps its bound `this`.

But constructor invocation introduces an important special case.

```js
function User(name) {
  this.name = name;
}

const obj = {
  name: "Ignored",
};

const BoundUser =
  User.bind(obj);

const user =
  new BoundUser("Vikash");

console.log(user.name);
```

Output:

```text
Vikash
```

When a bound function is used as a constructor with `new`, the constructor's newly created object becomes the relevant `this`; the bound `thisArg` is ignored.

This is why the complete binding-priority model needs `new`.

We will study constructor functions and `new` carefully in the next lesson.

---

# 16. Complete call / apply / bind Comparison

| Feature | `call()` | `apply()` | `bind()` |
| --- | --- | --- | --- |
| Explicit `this` | Yes | Yes | Yes |
| Executes immediately | Yes | Yes | No |
| Returns new bound function | No | No | Yes |
| Arguments | Individually | Array-like list | Can pre-bind individually |
| Useful for method borrowing | Yes | Yes | Yes |
| Useful for later callback | Not by itself | Not by itself | Yes |
| Rebind arrow `this` | No | No | No |

---

# 17. One Example Showing All Three

```js
function introduce(
  role,
  city
) {
  console.log(
    `${this.name} - ${role} - ${city}`
  );
}

const user = {
  name: "Vikash",
};
```

### call

```js
introduce.call(
  user,
  "Developer",
  "Bengaluru"
);
```

Execute now; arguments individually.

### apply

```js
introduce.apply(
  user,
  [
    "Developer",
    "Bengaluru",
  ]
);
```

Execute now; arguments grouped.

### bind

```js
const introduceUser =
  introduce.bind(
    user,
    "Developer"
  );

introduceUser(
  "Bengaluru"
);
```

Create a bound function now; execute later.

---

# 18. Final Mental Model

```text
call()
│
├── explicit this
├── args individually
└── RUN NOW

apply()
│
├── explicit this
├── args as array-like list
└── RUN NOW

bind()
│
├── explicit this
├── can pre-bind arguments
├── RETURNS NEW FUNCTION
└── RUN LATER
```

---

# Interview Answers

**What does `bind()` do?**

`bind()` returns a new function whose `this` is bound to the supplied value for normal calls. It can also pre-bind initial arguments.

**What is the main difference between `call()` and `bind()`?**

`call()` invokes immediately. `bind()` returns a new bound function that can be invoked later.

**Can `call()` normally override the `this` of an already bound function?**

No. A normally invoked bound function retains its bound `this`.

**What happens if a bound constructible function is invoked with `new`?**

The bound `thisArg` is ignored for the constructor call, and `new` supplies the newly created instance as `this`.

---

# Key Takeaways

- `bind()` returns a new function.
- It does not immediately execute the original function.
- The returned function retains its bound `this` for normal calls.
- A later `call()` or `apply()` cannot normally replace that bound `this`.
- `bind()` can pre-bind arguments.
- Pre-binding arguments is a form of partial application.
- `bind()` is useful for callbacks and detached methods.
- It historically appeared heavily in React class components.
- `bind()` cannot change an arrow's lexical `this`.
- Constructor invocation with `new` is an important special case for bound functions.
