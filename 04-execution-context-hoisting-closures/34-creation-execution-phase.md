# Lesson 34 — Creation Phase vs Execution Phase

A common teaching model divides execution-context processing into two broad stages:

1. **Creation / setup phase**
2. **Execution phase**

This model is useful for understanding hoisting.

The ECMAScript specification uses more precise concepts such as environment records and declaration instantiation, but the two-phase model is an excellent practical mental model when used carefully.

---

# 1. Why Do We Need This Model?

Consider:

```js
console.log(a);

var a = 10;
```

Output:

```text
undefined
```

Why does JavaScript know that `a` exists before reaching:

```js
var a = 10;
```

Because declarations are processed when the execution environment is prepared.

Now compare:

```js
greet();

function greet() {
  console.log("Hello");
}
```

This works even though the function call appears before the declaration.

The setup of bindings explains these behaviors.

---

# 2. Simplified Two-Phase Model

Given:

```js
var name = "Vikash";

function greet() {
  console.log("Hello");
}

greet();
```

Conceptually:

```text
Execution Context
      │
      ├── Phase 1: Creation / Setup
      │
      └── Phase 2: Execution
```

---

# 3. Creation / Setup Phase

Before executing statements in order, JavaScript establishes bindings for declarations in the relevant environment.

For a simplified example:

```js
var name = "Vikash";

function greet() {
  console.log("Hello");
}
```

A learning model might visualize setup as:

```text
Bindings
│
├── name  → undefined
└── greet → function
```

Important: this simplification applies differently to different declaration types.

- `var` gets initialized with `undefined`
- function declarations are initialized with their function value
- `let`, `const`, and `class` bindings exist but are not initialized for access until their declaration is evaluated

That last behavior creates the **Temporal Dead Zone (TDZ)**.

---

# 4. Execution Phase

JavaScript then evaluates statements in order.

For:

```js
var name = "Vikash";

function greet() {
  console.log("Hello");
}

greet();
```

conceptually:

```text
Before statement execution:
name → undefined
greet → function

Execute:
var name = "Vikash"

Now:
name → "Vikash"

Execute:
greet()

Create function execution context
and execute its body
```

---

# 5. var Example

```js
console.log(score);

var score = 100;
```

Conceptual setup:

```text
score → undefined
```

Then execution:

```text
console.log(score)
        ↓
undefined

score = 100
```

This is why the result is `undefined`, not a `ReferenceError`.

---

# 6. Function Declaration Example

```js
sayHello();

function sayHello() {
  console.log("Hello");
}
```

During declaration setup, the function binding is initialized.

Conceptually:

```text
sayHello → function
```

Therefore it can be called before the declaration appears textually.

---

# 7. let and const Are Different

Consider:

```js
console.log(name);

let name = "Vikash";
```

This throws a `ReferenceError`.

It is inaccurate to say simply:

> let and const are not hoisted.

A better explanation is:

> Their bindings are created as part of environment setup, but they remain uninitialized until execution reaches their declarations.

The region in which the binding exists but cannot yet be accessed is called the **Temporal Dead Zone**.

Conceptually:

```text
Scope starts
   ↓
name binding exists
but is uninitialized
   ↓
Temporal Dead Zone
   ↓
let name = "Vikash"
   ↓
initialized
   ↓
safe to access
```

We will study TDZ in detail after hoisting.

---

# 8. Function Expression Example

Consider:

```js
sayHello();

var sayHello = function () {
  console.log("Hello");
};
```

During setup:

```text
sayHello → undefined
```

During execution, JavaScript reaches:

```js
sayHello();
```

which is effectively trying to call:

```js
undefined();
```

Therefore:

```text
TypeError
```

Later:

```js
sayHello = function () {
  console.log("Hello");
};
```

assigns the function value.

This distinction between function declarations and function expressions is a common interview topic.

---

# 9. Arrow Function Example

```js
greet();

const greet = () => {
  console.log("Hello");
};
```

The `greet` binding exists but is in the TDZ before its declaration is evaluated.

Therefore the call throws a `ReferenceError`.

The arrow function itself does not receive function-declaration-style hoisting.

The behavior here primarily follows the declaration used to store it: `const`.

---

# 10. Function Calls Create New Contexts

Consider:

```js
var globalValue = 10;

function calculate(a) {
  var result = a * 2;

  return result;
}

calculate(5);
```

First, the global execution context is prepared and executed.

When:

```js
calculate(5);
```

runs, a new function execution context is created for that call.

That function context also establishes its own bindings before executing the function body.

Simplified:

```text
Global Context
│
├── globalValue
└── calculate
       ↓ call

Calculate Function Context
│
├── a = 5
└── result → initially undefined
```

Then the function body executes:

```text
result = 10
return 10
```

---

# 11. Why “JavaScript Moves Declarations to the Top” Is Misleading

You may hear:

> JavaScript moves variables and functions to the top.

JavaScript does not literally rewrite:

```js
console.log(a);
var a = 10;
```

into:

```js
var a;
console.log(a);
a = 10;
```

That rewritten version can be a beginner-friendly analogy, but the better mental model is:

> JavaScript establishes declaration bindings when preparing the execution environment before evaluating statements.

This explains hoisting without imagining source code physically moving.

---

# 12. React Connection

Consider:

```jsx
function Component() {
  handleClick();

  function handleClick() {
    console.log("Clicked");
  }

  return null;
}
```

A function declaration is available within that function execution according to declaration-instantiation rules.

But:

```jsx
function Component() {
  handleClick();

  const handleClick = () => {
    console.log("Clicked");
  };

  return null;
}
```

tries to access `handleClick` while its `const` binding is still in the TDZ.

Understanding declaration setup removes the need to memorize these as unrelated exceptions.

---

# 13. Interview Perspective

**What happens during the creation/setup phase?**

JavaScript establishes bindings for declarations in the execution environment. Their initial state depends on the declaration type.

**What happens during execution?**

Statements and expressions are evaluated in order, assignments occur, and function calls create new execution contexts.

**Are let and const hoisted?**

Their bindings are created before their declaration line is evaluated, but they remain uninitialized and inaccessible during the Temporal Dead Zone.

---

# Key Takeaways

- Execution can be understood using setup and execution stages.
- Declarations are processed before normal statement evaluation.
- `var` bindings begin initialized to `undefined`.
- Function declarations are initialized with their function values.
- `let` and `const` bindings remain uninitialized before their declaration is evaluated.
- Function expressions follow the behavior of the variable declaration storing them.
- Hoisting is about declaration/binding setup, not physically moving code.
