# Lesson 43 — `this` in Regular Functions

Now that the basic mental model is clear, this lesson applies it to the most common **regular-function** situations.

The rule to keep repeating is:

> **Find the exact invocation before deciding `this`.**

---

## 1. Plain Regular Function

```js
"use strict";

function showThis() {
  console.log(this);
}

showThis();
```

Trace:

```text
Function: showThis
Call: showThis()
Receiver: none
Strict mode: yes

this = undefined
```

In non-strict classic browser scripts, global-object substitution can occur. Modern module code uses strict semantics.

---

## 2. Object Method

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

user.showName();
```

Trace:

```text
Function: showName
Call: user.showName()
Receiver: user

this = user
```

Output:

```text
Vikash
```

---

## 3. Method Shorthand Does Not Bind this Permanently

These are both regular functions:

```js
const a = {
  show: function () {
    console.log(this);
  },
};

const b = {
  show() {
    console.log(this);
  },
};
```

The shorter method syntax does not permanently bind `this`.

For:

```js
b.show();
```

`this` is `b` because **b is the receiver of this call**.

---

## 4. Reusing One Function

```js
function showName() {
  console.log(this.name);
}

const developer = {
  name: "Vikash",
  showName,
};

const admin = {
  name: "Admin",
  showName,
};

developer.showName();
admin.showName();
```

Output:

```text
Vikash
Admin
```

Same function:

```text
developer.showName()
→ this = developer

admin.showName()
→ this = admin
```

---

## 5. Nested Object

```js
const company = {
  name: "Company",

  employee: {
    name: "Vikash",

    showName() {
      console.log(this.name);
    },
  },
};

company.employee.showName();
```

Do not say `this = company`.

The receiver is:

```js
company.employee
```

Therefore:

```text
this = company.employee
```

Output:

```text
Vikash
```

---

## 6. Detached Method

```js
"use strict";

const user = {
  name: "Vikash",

  showName() {
    console.log(this?.name);
  },
};

const show = user.showName;

show();
```

Important transformation:

```text
user.showName()
↓
method call with user receiver

const show = user.showName
↓
only function value is copied

show()
↓
plain call
↓
this = undefined
```

Output:

```text
undefined
```

This is one of the most common `this` interview questions.

---

## 7. Moving the Function to Another Object

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

admin.showName = user.showName;

admin.showName();
```

Output:

```text
Admin
```

The function did not remember `user`.

The current invocation is:

```js
admin.showName();
```

Therefore:

```text
this = admin
```

---

## 8. Nested Regular Function

This causes major confusion:

```js
"use strict";

const user = {
  name: "Vikash",

  show() {
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

user.show();
```

Analyze the functions separately.

### show

```js
user.show();
```

So:

```text
show's this = user
```

### inner

```js
inner();
```

This is a plain regular-function call.

So in strict mode:

```text
inner's this = undefined
```

A nested **regular function does not inherit its parent's `this`**.

---

## 9. Why This Is Different from a Normal Variable

```js
const user = {
  show() {
    const message = "Hello";

    function inner() {
      console.log(message);
    }

    inner();
  },
};

user.show();
```

`message` works through lexical scope.

But regular-function `this` does not use that lexical lookup:

```text
message
→ lexical scope
→ outer environment

inner's this
→ how was inner called?
→ inner()
→ plain call
```

This distinction must be completely clear before arrow functions.

---

## 10. Passing a Method as a Callback

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

It may look like `user` should remain attached, but what is actually passed?

```text
user.showName
      ↓
function value
```

Inside `execute`:

```js
callback();
```

That is a plain call.

Therefore:

```text
this = undefined
```

The important lesson is:

> Passing a method as a callback does not automatically preserve its original receiver.

---

## 11. setTimeout Callback

```js
const user = {
  name: "Vikash",

  showName() {
    console.log(this.name);
  },
};

setTimeout(user.showName, 1000);
```

Do not reason as though the runtime later executes:

```js
user.showName();
```

You passed a function value to a host API.

The host controls how the callback is invoked, so depending on implicit `this` here is fragile.

Later we will solve this cleanly with arrow wrappers and `bind()`.

---

## 12. DOM Event Listener Special Case

Some APIs deliberately choose a callback `this`.

For a browser DOM event listener registered with a regular function:

```js
button.addEventListener(
  "click",
  function (event) {
    console.log(this);
    console.log(event.currentTarget);
  }
);
```

The DOM event API invokes the listener so that `this` corresponds to the registered element, matching `event.currentTarget`.

This is an **API-defined callback behavior**, not a rule saying every callback receives an object as `this`.

---

## 13. forEach Callback

```js
"use strict";

const user = {
  name: "Vikash",
  skills: ["JS", "React"],

  showSkills() {
    this.skills.forEach(
      function (skill) {
        console.log(
          this,
          skill
        );
      }
    );
  },
};

user.showSkills();
```

The callback is another regular function.

It does not automatically inherit `showSkills`'s `this`.

`forEach` supports an optional `thisArg`:

```js
user.skills.forEach(
  function (skill) {
    console.log(
      this.name,
      skill
    );
  },
  user
);
```

Later, arrow functions will provide another common solution.

---

## 14. Returned Regular Function

```js
"use strict";

const user = {
  name: "Vikash",

  getFunction() {
    return function () {
      console.log(this);
    };
  },
};

const fn = user.getFunction();

fn();
```

There are two calls.

First:

```text
user.getFunction()
→ getFunction's this = user
```

Later:

```text
fn()
→ returned regular function
→ plain call
→ this = undefined
```

The returned regular function does not lexically inherit `getFunction`'s `this`.

---

## 15. Reliable Tracing Formula

For every regular-function question, write:

```text
FUNCTION
↓
Which function contains this?

CALL SITE
↓
What exact expression invokes it?

RECEIVER / RULE
↓
object.method()?
plain function?
call/apply?
bind?
new?
API callback?

RESULT
↓
Determine this
```

Example:

```js
const user = {
  name: "Vikash",

  show() {
    console.log(this.name);
  },
};

user.show();
```

Trace:

```text
Function: show
Call: user.show()
Receiver: user
this: user
this.name: "Vikash"
```

---

## 16. Practice

### Example A

```js
const obj = {
  value: 10,

  getValue() {
    return this.value;
  },
};

console.log(obj.getValue());
```

Answer:

```text
10
```

because `obj` is the receiver.

### Example B

```js
"use strict";

const obj = {
  value: 10,

  getValue() {
    return this?.value;
  },
};

const fn = obj.getValue;

console.log(fn());
```

Answer:

```text
undefined
```

because `fn()` is a plain call.

### Example C

```js
"use strict";

const obj = {
  value: 10,

  outer() {
    function inner() {
      return this?.value;
    }

    return inner();
  },
};

console.log(obj.outer());
```

Answer:

```text
undefined
```

Trace:

```text
obj.outer()
→ outer this = obj

inner()
→ plain regular-function call
→ inner this = undefined
```

---

## Regular-Function Mental Model

```text
object.method()
→ this = receiver object

plainFunction()
→ strict mode: this = undefined

const fn = object.method;
fn();
→ original receiver is lost

nested regular function:
inner();
→ does not inherit outer this

callback:
→ depends on how the caller/API invokes it
```

Later lessons add:

```text
arrow functions
call()
apply()
bind()
new
```

---

## Interview Answers

**What determines `this` in a regular function?**

Primarily how the function is invoked, with additional rules for explicit binding, constructor calls and API-controlled callbacks.

**Why does a detached method lose its `this`?**

Because extracting the method gives you the function value. A later plain invocation no longer has the original object as its receiver.

**Does a nested regular function inherit the outer function's `this`?**

No. Its `this` is determined by its own invocation.

---

## Key Takeaways

- Always inspect the call site.
- `object.method()` normally sets `this` to the receiver.
- The same regular function can use different receivers.
- Nested objects do not automatically make the outermost object `this`.
- Detached methods lose their original receiver.
- Nested regular functions do not inherit outer `this`.
- Passing a method as a callback does not preserve its receiver automatically.
- APIs can define how callbacks are invoked.
- Strict-mode plain calls have `this === undefined`.
- Arrow functions behave differently and come next.
