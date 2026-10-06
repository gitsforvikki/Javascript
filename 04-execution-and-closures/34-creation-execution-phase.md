# Lesson 34 — Creation Phase vs Execution Phase

A JavaScript execution context is commonly explained in two conceptual stages:

1. **Creation Phase**
2. **Execution Phase**

This model is essential for understanding:

- hoisting
- `var`
- `let`
- `const`
- function declarations
- Temporal Dead Zone

---

## 1. Big Picture

When an execution context is created:

```text
Execution Context
↓
Creation Phase
↓
Execution Phase
```

Creation phase prepares the environment.

Execution phase runs the statements.

---

## 2. Creation Phase

Before ordinary statement-by-statement execution, JavaScript establishes bindings for declarations in the scope.

Conceptually:

```text
Creation Phase
│
├── prepare variable bindings
├── prepare function declarations
├── create lexical environment
├── connect outer environment
└── establish this binding where applicable
```

This is the foundation of what developers call **hoisting**.

---

## 3. Execution Phase

During execution:

```text
statements run
↓
expressions evaluate
↓
assignments happen
↓
functions may be called
```

Example:

```js
var a = 10;
```

Conceptually:

Creation:

```text
a → undefined
```

Execution:

```text
a = 10
```

---

## 4. var During Creation Phase

Example:

```js
console.log(a);

var a = 10;
```

Output:

```text
undefined
```

A useful model:

Creation phase:

```text
a
↓
initialized with undefined
```

Execution phase:

```text
console.log(a)
↓
undefined

then

a = 10
```

---

## 5. let During Creation Phase

Example:

```js
console.log(a);

let a = 10;
```

This throws:

```text
ReferenceError
```

Why?

The `let` binding is created, but it is not initialized yet.

Conceptually:

```text
Creation phase:
a exists
but is uninitialized

Execution phase:
before declaration
→ access forbidden

declaration reached
→ initialize a = 10
```

The pre-initialization interval is the Temporal Dead Zone.

---

## 6. const During Creation Phase

`const` behaves similarly to `let` regarding the TDZ.

```js
console.log(
  value
);

const value = 10;
```

Throws a `ReferenceError`.

The binding exists but remains uninitialized until the declaration executes.

---

## 7. Function Declarations

Example:

```js
greet();

function greet() {
  console.log(
    "Hello"
  );
}
```

This works because the function declaration binding is initialized with the function object during environment setup.

Conceptually:

```text
Creation Phase
greet
↓
function object
```

So it can be called before the declaration line.

---

## 8. Function Expressions with var

```js
greet();

var greet =
  function () {
    console.log(
      "Hello"
    );
  };
```

What happens?

Creation phase:

```text
greet → undefined
```

Execution:

```text
greet()
↓
TypeError
because undefined
is not callable
```

This is a classic interview question.

---

## 9. Function Expressions with let / const

```js
greet();

const greet =
  function () {
    console.log(
      "Hello"
    );
  };
```

Before declaration, `greet` is in the TDZ.

So:

```text
ReferenceError
```

not:

```text
TypeError
```

This distinction is important.

---

## 10. Arrow Function with var

```js
greet();

var greet =
  () => {
    console.log(
      "Hello"
    );
  };
```

Same behavior as any `var` function expression:

```text
greet → undefined
during creation

greet()
→ TypeError
```

The arrow function itself is not "hoisted like a function declaration."

Only the `var` binding behavior applies.

---

## 11. Example Trace

Code:

```js
console.log(a);

sayHi();

var a = 10;

function sayHi() {
  console.log(
    "Hi"
  );
}
```

Creation phase:

```text
a → undefined

sayHi
→ function object
```

Execution phase:

```text
console.log(a)
→ undefined

sayHi()
→ Hi

a = 10
```

---

## 12. let Trace

Code:

```js
console.log(a);

let a = 10;
```

Creation:

```text
a
→ binding created
→ uninitialized
```

Execution:

```text
console.log(a)
→ ReferenceError
```

The declaration line is never reached because execution stops on the error.

---

## 13. Hoisting Is Not Physical Movement

Bad explanation:

```text
JavaScript moves declarations
to the top of the file.
```

Better explanation:

> During execution-context setup, bindings for declarations are created before normal statement execution begins.

Nothing is physically moved in the source code.

---

## 14. Function Context Also Has These Phases

Example:

```js
function test() {
  console.log(a);

  var a = 20;
}

test();
```

When `test()` is called:

```text
Function Execution Context created
↓
Creation Phase
a → undefined
↓
Execution Phase
console.log(a)
→ undefined
↓
a = 20
```

The model applies inside functions too.

---

## 15. Parameter Initialization

Example:

```js
function greet(
  name
) {
  console.log(
    name
  );
}

greet(
  "Vikash"
);
```

Function parameters are initialized as part of function-call setup.

So the function execution context begins with:

```text
name → "Vikash"
```

---

## 16. Function Declaration vs Function Expression

### Function Declaration

```js
function greet() {}
```

Available before its source line in the scope.

### Function Expression

```js
var greet =
  function () {};
```

Only the variable declaration behavior applies before assignment.

This difference is asked frequently in interviews.

---

## 17. Class Declarations

Classes behave more like `let` / `const` regarding initialization.

Example:

```js
const user =
  new User();

class User {}
```

This throws before the class declaration executes.

Class declarations are in a TDZ.

---

## 18. Temporal Dead Zone Connection

For `let`, `const`, and `class`:

```text
scope begins
↓
binding exists but uninitialized
↓
TDZ
↓
declaration executes
↓
binding initialized
```

This is more precise than saying:

```text
let is not hoisted
```

It is better to say:

> `let` and `const` bindings are created during scope setup but remain uninitialized until execution reaches the declaration.

---

## 19. Why var Gives undefined

`var` is initialized with:

```text
undefined
```

during environment setup.

So reading it early is allowed.

This is why:

```js
console.log(a);
var a = 10;
```

prints `undefined`.

---

## 20. Why let Gives ReferenceError

The binding exists, but access is not allowed while uninitialized.

So:

```js
console.log(a);
let a = 10;
```

throws instead of returning `undefined`.

---

## 21. Important Error Difference

### var function expression

```js
sayHi();

var sayHi =
  function () {};
```

Result:

```text
TypeError
```

because:

```text
sayHi === undefined
```

### let/const function expression

```js
sayHi();

const sayHi =
  function () {};
```

Result:

```text
ReferenceError
```

because the binding is still uninitialized.

---

## 22. React Connection

Inside a React component:

```jsx
function Component() {
  console.log(
    value
  );

  const value =
    10;

  return null;
}
```

Each component invocation creates a new function execution context.

Inside that call, `value` follows normal `const` TDZ rules.

React does not change JavaScript execution semantics.

---

## 23. Interview Table

| Declaration | Binding Created Before Execution? | Initial State Before Declaration Line |
| --- | --- | --- |
| `var` | Yes | `undefined` |
| `let` | Yes | Uninitialized / TDZ |
| `const` | Yes | Uninitialized / TDZ |
| Function declaration | Yes | Function object |
| Class declaration | Yes | Uninitialized / TDZ |

---

## Interview Questions

### What are the creation and execution phases?

Creation phase prepares bindings and execution environment; execution phase runs statements and assignments.

### Why does var print undefined before declaration?

Because its binding is initialized to `undefined` during context setup.

### Why does let throw?

Because the binding exists but is uninitialized in the TDZ.

### Are declarations physically moved?

No.

---

## Interview Answer — 30 Seconds

> When an execution context is created, JavaScript first sets up the environment before running statements. `var` bindings are initialized to `undefined`, function declarations are initialized with their function objects, while `let`, `const`, and classes are created but remain uninitialized in the TDZ. Then the execution phase runs the code line by line and performs assignments and function calls.

---

## Key Takeaways

- Execution context setup happens before normal statement execution.
- `var` is initialized with `undefined`.
- `let` and `const` are created but uninitialized.
- Function declarations are available before their textual line.
- Function expressions follow their variable declaration semantics.
- Hoisting is environment setup, not physical source-code movement.
