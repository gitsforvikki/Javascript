# Lesson 43 — `this` in Regular Functions

Lesson 42 established the central rule:

> For an ordinary function, determine `this` by examining **how the function is invoked**.

Now we will apply that rule carefully to regular functions.

---

# 1. What Do We Mean by a Regular Function?

Examples:

```js
function greet() {
  console.log(this);
}
```

and:

```js
const greet = function () {
  console.log(this);
};
```

These are ordinary functions.

They have dynamic `this` behavior.

Arrow functions are different and will be studied separately.

---

# 2. Plain Function Call

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis();
```

Trace:

```text
Function
→ showThis

Call site
→ showThis()

Receiver object?
→ No

Strict mode plain call
→ this = undefined
```

Output:

```text
undefined
```

---

# 3. Why Strict Mode Is Important

Without strict mode, classic-script plain calls can substitute the global object.

Historically:

```js
function showThis() {
  console.log(this);
}

showThis();
```

in a non-strict browser classic script can produce the global object.

But modern code commonly uses ES modules, which are strict automatically.

Therefore, do not write application logic that depends on plain-function `this` becoming the global object.

A strong modern mental model is:

```text
plain regular-function call
+
strict mode
↓
this = undefined
```

---

# 4. Method Invocation

Now compare:

```js
"use strict";

const user = {
  name: "Vikash",

  showName: function () {
    console.log(this.name);
  },
};

user.showName();
```

Trace:

```text
Call site
→ user.showName()

Receiver
→ user

this
→ user

this.name
→ "Vikash"
```

Output:

```text
Vikash
```

---

# 5. Method Shorthand Does Not Change the Rule

These two forms both create ordinary methods/functions for this purpose:

```js
const user = {
  showName: function () {
    console.log(this.name);
  },
};
```

and:

```js
const user = {
  showName() {
    console.log(this.name);
  },
};
```

In both cases:

```js
user.showName();
```

uses `user` as the receiver.

---

# 6. Same Function, Different Objects

```js
"use strict";

function introduce() {
  console.log(
    `I am ${this.name}`
  );
}

const developer = {
  name: "Vikash",
  introduce,
};

const designer = {
  name: "Rahul",
  introduce,
};

developer.introduce();
designer.introduce();
```

Output:

```text
I am Vikash
I am Rahul
```

Nothing inside `introduce` changed.

Only the receiver changed.

```text
developer.introduce()
          ↓
this = developer

designer.introduce()
         ↓
this = designer
```

---

# 7. Detached Method

This is one of the most important cases.

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

const showName = user.showName;

showName();
```

What happened?

Initially:

```text
user.showName
     ↓
function reference
```

Then:

```js
const showName = user.showName;
```

copies the function reference into another variable.

It does **not** permanently copy:

```text
this = user
```

Now the call is:

```js
showName();
```

not:

```js
user.showName();
```

In strict mode:

```text
this = undefined
```

Then trying:

```js
this.name
```

causes a `TypeError`.

---

# 8. Very Important: Property Access and Function Call Are Separate Ideas

Consider:

```js
const fn = user.showName;
```

`user.showName` accesses a property and obtains a function value.

After assignment:

```text
fn ─────┐
        │
        └──→ same function object

user.showName ─→ same function object
```

But `this` is not stored as:

```text
function.showName.this = user
```

Later invocation determines the ordinary function's `this`.

---

# 9. Moving the Same Function Between Objects

```js
function showName() {
  console.log(this.name);
}

const user1 = {
  name: "Vikash",
  showName,
};

const user2 = {
  name: "Kumar",
};

user2.showName = user1.showName;

user2.showName();
```

Output:

```text
Kumar
```

Why?

Call site:

```js
user2.showName();
```

Therefore:

```text
this → user2
```

It does not matter that the function reference came through `user1`.

---

# 10. Nested Objects

```js
const company = {
  name: "Tech Corp",

  developer: {
    name: "Vikash",

    showName() {
      console.log(this.name);
    },
  },
};

company.developer.showName();
```

Trace the immediate receiver:

```text
company.developer.showName()
        └───────┬───────┘
                ↓
       company.developer
```

Therefore:

```text
this = company.developer
```

Output:

```text
Vikash
```

Not:

```text
Tech Corp
```

---

# 11. Nested Regular Function — Major Confusion Point

Consider:

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

Many developers think:

```text
showName has this = user

therefore

inner also has this = user
```

That is incorrect for a regular nested function.

Analyze each function call separately.

Outer call:

```js
user.showName();
```

Therefore:

```text
inside showName
this = user
```

But inner call:

```js
inner();
```

That is a plain regular-function call.

In strict mode:

```text
inside inner
this = undefined
```

Full trace:

```text
user.showName()
      ↓
this = user

      inside showName

inner()
  ↓
plain call
  ↓
this = undefined
```

This is extremely important.

> A nested regular function does not automatically inherit `this` from its outer regular function.

---

# 12. `this` vs Lexical Variables in a Nested Function

Compare:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    const role = "Developer";

    function inner() {
      console.log(role);
      console.log(this);
    }

    inner();
  },
};

user.showName();
```

Inside `inner`:

```text
role
↓
resolved lexically
↓
"Developer"
```

But:

```text
this
↓
determined from inner()
call
↓
undefined
```

This gives us a powerful comparison:

```text
ordinary variable
→ lexical scope

regular-function this
→ call site
```

Do not mix these two systems.

---

# 13. Saving Outer `this` Manually — Historical Pattern

Before arrow functions became common, code sometimes used:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    const self = this;

    function inner() {
      console.log(self.name);
    }

    inner();
  },
};

user.showName();
```

Why does this work?

Inside `showName`:

```text
this = user
```

Then:

```js
const self = this;
```

creates an ordinary lexical variable.

The nested function resolves `self` through lexical scope.

```text
inner()
  ↓
self not local
  ↓
look outward
  ↓
self = user
```

This old pattern helps reveal the difference between lexical variables and dynamic regular-function `this`.

Later we will see why arrow functions are often used instead.

---

# 14. Passing a Method as a Callback

Consider:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

function execute(callback) {
  callback();
}

execute(user.showName);
```

It is tempting to think:

```text
user.showName was passed
therefore this = user
```

But inspect what `execute` actually does:

```js
callback();
```

That is a plain function call.

The original receiver is gone.

Conceptually:

```text
user.showName
     ↓
function value passed
     ↓
callback
     ↓
callback()
     ↓
plain call
     ↓
this = undefined
in strict mode
```

This is a very common real-world source of `this` bugs.

---

# 15. Callback Example with setTimeout

Consider:

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

setTimeout(user.showName, 1000);
```

Do not assume that because the function came from `user.showName`, it will necessarily execute later as:

```js
user.showName();
```

A callback API controls how it invokes the callback.

The safe principle is:

> Passing a method as a function value does not automatically preserve its original receiver.

Later, `bind()` and arrow functions will give us explicit solutions.

---

# 16. Assignment Can Change the Call Form

Look carefully:

```js
const user = {
  name: "Vikash",

  show() {
    console.log(this.name);
  },
};

user.show();
```

Here:

```text
this = user
```

Now:

```js
const fn = user.show;

fn();
```

The function is the same.

The call form is different.

That means:

```text
same function
+
different invocation
=
possibly different this
```

This single idea explains many interview questions.

---

# 17. Do Not Determine `this` from the Variable Name

```js
const user = {
  name: "Vikash",

  show() {
    console.log(this.name);
  },
};

const userShow = user.show;

userShow();
```

The variable is named `userShow`, but that does not matter.

JavaScript does not inspect variable names to determine `this`.

It inspects the invocation semantics.

---

# 18. Regular Function Decision Process

When you see:

```js
function example() {
  console.log(this);
}
```

do not immediately decide the value.

Find the call.

### Case 1

```js
obj.example();
```

Think:

```text
method-style call
↓
this = obj
```

### Case 2

```js
example();
```

Think:

```text
plain function call
↓
strict mode: this = undefined
```

### Case 3

```js
const fn = obj.example;

fn();
```

Think:

```text
method detached
↓
plain call
↓
strict mode: this = undefined
```

Do this before thinking about the function body.

---

# 19. Important Preview: More Binding Rules Are Coming

Regular functions can also be invoked using:

```js
fn.call(...)
fn.apply(...)
```

or permanently wrapped with:

```js
fn.bind(...)
```

They can also be invoked as constructors:

```js
new User()
```

Those calls have their own `this` rules.

Do not mix them into today's rule yet.

We will learn them individually and then build one final precedence model.

---

# 20. React Connection

Suppose an object method is passed as an event callback:

```jsx
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

function App() {
  return (
    <button onClick={user.showName}>
      Show Name
    </button>
  );
}
```

The expression:

```js
user.showName
```

retrieves the function.

It does not execute:

```js
user.showName()
```

at that moment.

React receives a callback function and invokes it later according to React's event system.

Therefore, do not rely on the original method receiver being preserved.

This is one reason modern React function-component code typically uses lexical variables and closures rather than object-method `this`.

---

# 21. Interview Problem 1

Predict:

```js
"use strict";

const user = {
  name: "Vikash",

  print() {
    console.log(this.name);
  },
};

const print = user.print;

user.print();
print();
```

First call:

```text
user.print()
↓
this = user
↓
Vikash
```

Second call:

```text
print()
↓
plain function call
↓
this = undefined
↓
reading this.name throws TypeError
```

---

# 22. Interview Problem 2

```js
"use strict";

const user = {
  name: "Vikash",

  outer() {
    console.log(
      "outer:",
      this.name
    );

    function inner() {
      console.log(
        "inner:",
        this
      );
    }

    inner();
  },
};

user.outer();
```

Trace:

```text
user.outer()
↓
outer this = user

inner()
↓
plain regular-function call
↓
inner this = undefined
```

A regular nested function does not inherit outer `this`.

---

# 23. Interview Problem 3

```js
function show() {
  console.log(this.name);
}

const a = {
  name: "A",
  show,
};

const b = {
  name: "B",
  show,
};

a.show();
b.show();
```

Output:

```text
A
B
```

Reason:

```text
same function

a.show()
→ this = a

b.show()
→ this = b
```

---

# Interview Perspective

## How is `this` determined in a regular function?

Primarily by how the function is invoked. A method-style call uses its receiver as `this`; a plain function call has no such receiver, and in strict mode its `this` is `undefined`.

## Does a nested regular function inherit `this`?

No. Each regular-function invocation determines its own `this`.

## Why does extracting an object method often lose `this`?

Because extracting the method copies the function value. Calling that value later as `fn()` is a plain function invocation rather than `object.method()`.

## What is the difference between lexical variables and regular-function `this`?

Normal identifiers are resolved through lexical scope. Regular-function `this` is determined by invocation rules.

---

# Final Mental Model

Whenever you encounter a regular function:

```text
STOP
 ↓
Do not inspect only
where function was written
 ↓
Find its CALL SITE
 ↓
How is it invoked?
```

Then:

```text
user.show()
    ↓
this = user
```

But:

```text
const fn = user.show;

fn()
 ↓
plain call
 ↓
strict mode
 ↓
this = undefined
```

And:

```text
user.outer()
↓
outer this = user

inside:
inner()
↓
new regular-function call
↓
inner does NOT inherit outer this
```

---

# Key Takeaways

- Regular functions have dynamic `this`.
- Determine `this` from the function's invocation.
- `object.method()` uses the receiver object as `this`.
- A detached method loses its method-style receiver.
- The same function can have different `this` values in different calls.
- Nested regular functions do not inherit outer `this`.
- Normal variables use lexical scope; regular-function `this` does not.
- Passing a method as a callback does not automatically preserve its receiver.
- Strict-mode plain function calls have `this === undefined`.
- Arrow functions, `call`, `apply`, `bind`, and `new` will add more rules in the upcoming lessons.
