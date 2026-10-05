# Lesson 36 — Hoisting

Hoisting is one of JavaScript's most frequently misunderstood concepts.

A weak explanation is:

> JavaScript moves declarations to the top.

A better explanation is:

> Before normal statement execution, JavaScript establishes bindings for declarations in their lexical environment. The binding's initial state depends on the declaration type.

Nothing needs to be imagined as physically moving in the source code.

---

# 1. Why Hoisting Exists as a Concept

Consider:

```js
console.log(name);

var name = "Vikash";
```

Output:

```text
undefined
```

But:

```js
console.log(name);

let name = "Vikash";
```

throws a `ReferenceError`.

And:

```js
greet();

function greet() {
  console.log("Hello");
}
```

works.

These different behaviors come from how declaration bindings are created and initialized.

---

# 2. var Hoisting

Consider:

```js
console.log(score);

var score = 100;
```

Simplified setup:

```text
score → undefined
```

Execution:

```text
console.log(score)
        ↓
undefined

score = 100
```

Therefore:

```js
console.log(score); // undefined

var score = 100;

console.log(score); // 100
```

## Declaration vs Assignment

This line:

```js
var score = 100;
```

contains conceptually:

```text
declaration → var score
assignment  → score = 100
```

The declaration is processed during environment setup.

The assignment occurs when execution reaches that statement.

---

# 3. Function Declaration Hoisting

```js
greet();

function greet() {
  console.log("Hello");
}
```

This works because the function declaration binding is initialized with the function during declaration setup.

Conceptually:

```text
Before normal execution:

greet → function
```

Therefore:

```js
greet();
```

can invoke it before its textual declaration.

---

# 4. Function Expression with var

```js
greet();

var greet = function () {
  console.log("Hello");
};
```

This does **not** behave like a function declaration.

During setup:

```text
greet → undefined
```

Then execution reaches:

```js
greet();
```

which tries to call an undefined value.

Result:

```text
TypeError
```

Only later does execution assign:

```js
greet = function () {
  console.log("Hello");
};
```

---

# 5. let Hoisting and TDZ

Consider:

```js
console.log(age);

let age = 25;
```

This throws:

```text
ReferenceError
```

It is common to hear:

> let is not hoisted.

That is an oversimplification.

The binding exists from the beginning of its scope, but it remains **uninitialized** until its declaration is evaluated.

Conceptually:

```text
Enter scope
    ↓
age binding created
but uninitialized
    ↓
Temporal Dead Zone
    ↓
let age = 25
    ↓
age initialized
```

Access during the TDZ throws a `ReferenceError`.

---

# 6. const Hoisting

`const` follows the same important TDZ principle.

```js
console.log(role);

const role = "Developer";
```

Result:

```text
ReferenceError
```

The binding exists but cannot be accessed before initialization.

A `const` declaration must also receive an initializer:

```js
const role;
```

is invalid syntax.

---

# 7. class Declarations

Class declarations also cannot be accessed before their declaration is evaluated.

```js
const user = new User();

class User {}
```

This throws a `ReferenceError`.

Class declarations have TDZ-like behavior similar to `let` and `const`.

---

# 8. Compare Declaration Types

A practical comparison:

| Declaration | Binding established before line? | Initial state before declaration evaluation | Access before line |
| --- | --- | --- | --- |
| `var` | Yes | `undefined` | Returns `undefined` |
| `let` | Yes | Uninitialized | ReferenceError |
| `const` | Yes | Uninitialized | ReferenceError |
| Function declaration | Yes | Function value | Callable |
| `class` | Yes | Uninitialized | ReferenceError |

This table captures the behavior much better than simply saying some declarations “hoist” and others do not.

---

# 9. Hoisting Inside Functions

Hoisting follows scope.

```js
function test() {
  console.log(value);

  var value = 10;
}

test();
```

Output:

```text
undefined
```

The `value` binding belongs to the function's environment.

Simplified:

```text
test execution context
│
├── setup
│   └── value → undefined
│
└── execution
    ├── console.log(value)
    └── value = 10
```

---

# 10. var Is Function-Scoped

Consider:

```js
function test() {
  if (true) {
    var message = "Hello";
  }

  console.log(message);
}

test();
```

Output:

```text
Hello
```

`var` does not create block scope for an `if` block.

Compare:

```js
function test() {
  if (true) {
    let message = "Hello";
  }

  console.log(message);
}

test();
```

This throws a `ReferenceError` because `let` is block-scoped.

Hoisting must always be understood together with **scope**.

---

# 11. A Classic Interview Question

What happens here?

```js
var a = 10;

function test() {
  console.log(a);

  var a = 20;
}

test();
```

Many developers expect:

```text
10
```

Actual output:

```text
undefined
```

Why?

The `var a` inside `test` creates a local function-scoped binding.

Simplified setup of `test`:

```text
local a → undefined
```

Therefore:

```js
console.log(a);
```

uses the local `a`, not the global one.

Only afterward:

```js
a = 20;
```

runs.

Mental model:

```text
Global:
a = 10

test context:
a = undefined

console.log(a)
       ↓
local a found
       ↓
undefined

Scope lookup stops.
Global a is not used.
```

This also demonstrates **variable shadowing**.

---

# 12. Function Declaration vs Arrow Function

Function declaration:

```js
sayHello();

function sayHello() {
  console.log("Hello");
}
```

Works.

Arrow function stored in `const`:

```js
sayHello();

const sayHello = () => {
  console.log("Hello");
};
```

Throws a `ReferenceError`.

Why?

The arrow function syntax itself is not what gets function-declaration-style initialization.

`sayHello` is a `const` binding and remains inaccessible in the TDZ until its declaration is evaluated.

---

# 13. typeof and TDZ

Normally:

```js
console.log(
  typeof somethingNotDeclared
);
```

returns:

```text
"undefined"
```

But:

```js
console.log(typeof value);

let value = 10;
```

throws a `ReferenceError`.

Why?

`value` is a real binding in the current scope, but it is still in the TDZ.

This is a useful advanced interview detail.

---

# 14. Hoisting Is Scope-Sensitive

Consider:

```js
let value = "global";

function test() {
  console.log(value);

  let value = "local";
}

test();
```

This does not print `"global"`.

It throws a `ReferenceError`.

Why?

Inside `test`, the local `value` binding exists for the entire function block's lexical scope.

Before:

```js
let value = "local";
```

it is in the TDZ.

The engine finds the local binding during scope resolution and does not continue outward to the global `value`.

```text
test scope
   ↓
value binding found
   ↓
currently uninitialized
   ↓
ReferenceError

It does NOT continue to global value
```

This is a very important connection between hoisting, TDZ and scope chain.

---

# 15. Best Practices

Even though function declarations can be called before their textual declaration, code is often easier to read when declarations and dependencies are organized clearly.

Prefer:

```js
const calculateTotal = () => {
  // ...
};

calculateTotal();
```

when that organization suits the codebase.

Use `let` and `const` rather than relying on confusing `var` hoisting behavior in modern application code.

Most importantly:

> Understand hoisting so you can reason about existing code and interview questions, not so you can intentionally write confusing code.

---

# 16. React Connection

Consider:

```jsx
function UserCard() {
  console.log(handleClick);

  const handleClick = () => {
    console.log("Clicked");
  };

  return null;
}
```

This attempts to access `handleClick` during its TDZ and throws a `ReferenceError`.

Moving the access below initialization works:

```jsx
function UserCard() {
  const handleClick = () => {
    console.log("Clicked");
  };

  console.log(handleClick);

  return null;
}
```

Understanding hoisting also helps when reading closures and callbacks created during component execution.

---

# 17. Interview Perspective

**What is hoisting?**

Hoisting describes the observable behavior caused by JavaScript establishing declaration bindings before normal statement execution. Different declaration types have different initialization rules.

**Is var hoisted?**

Yes. Its binding is created and initialized to `undefined` before its assignment statement executes.

**Are let and const hoisted?**

Their bindings are established before the declaration line executes, but remain uninitialized and inaccessible in the Temporal Dead Zone until their declarations are evaluated.

**Why can function declarations be called before their declaration?**

Their bindings are initialized with the function value during declaration instantiation/setup.

**Does JavaScript physically move declarations to the top?**

No. That is only a teaching analogy.

---

# Key Takeaways

- Hoisting does not mean source code is physically moved.
- Declaration bindings are established before normal statement execution.
- `var` begins as `undefined`.
- Function declarations are initialized with their function values.
- `let`, `const`, and classes remain inaccessible before initialization.
- The inaccessible region is the Temporal Dead Zone.
- Function expressions follow the declaration behavior of the variable storing them.
- Hoisting is tightly connected to scope and execution contexts.
