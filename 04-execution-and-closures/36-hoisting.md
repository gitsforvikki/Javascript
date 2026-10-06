# Lesson 36 — Hoisting

Hoisting is one of the most misunderstood JavaScript concepts.

A common but misleading explanation is:

> JavaScript moves declarations to the top.

That is **not** what physically happens.

A better explanation is:

> During execution-context setup, JavaScript creates bindings for declarations before normal statement execution begins.

This lesson connects directly to:

- execution context
- creation phase
- TDZ
- function declarations
- `var`, `let`, and `const`

---

## 1. The Correct Mental Model

Source code:

```js
console.log(a);

var a = 10;
```

JavaScript does not rewrite it as:

```js
var a;

console.log(a);

a = 10;
```

That rewritten version is only a teaching approximation.

What really matters conceptually is:

```text
Creation phase
↓
binding for a created
↓
a initialized to undefined

Execution phase
↓
console.log(a)
↓
undefined

then
↓
a = 10
```

---

## 2. var Hoisting

```js
console.log(
  count
);

var count = 5;
```

Output:

```text
undefined
```

Why?

Because `var count` is created before execution and initialized with `undefined`.

The assignment happens only when execution reaches that line.

---

## 3. let Hoisting

```js
console.log(
  count
);

let count = 5;
```

Result:

```text
ReferenceError
```

`let` is hoisted in the sense that its binding is created before execution, but it is not initialized immediately.

It remains uninitialized until the declaration is executed.

That interval is the **Temporal Dead Zone**.

---

## 4. const Hoisting

`const` behaves like `let` with respect to hoisting.

```js
console.log(
  value
);

const value = 10;
```

Result:

```text
ReferenceError
```

The binding exists but is uninitialized.

---

## 5. Function Declaration Hoisting

```js
greet();

function greet() {
  console.log(
    "Hello"
  );
}
```

This works because the function declaration binding is initialized with the function object during context setup.

---

## 6. Function Expression with var

```js
greet();

var greet =
  function () {
    console.log(
      "Hello"
    );
  };
```

Creation phase:

```text
greet
→ undefined
```

Execution:

```text
greet()
↓
TypeError
```

because `undefined` is not callable.

---

## 7. Function Expression with const

```js
greet();

const greet =
  function () {
    console.log(
      "Hello"
    );
  };
```

Result:

```text
ReferenceError
```

because `greet` is still in the TDZ.

---

## 8. Arrow Function with var

```js
greet();

var greet =
  () => {
    console.log(
      "Hello"
    );
  };
```

Result:

```text
TypeError
```

The arrow function itself is not available until assignment executes.

---

## 9. var Is Function-Scoped

```js
if (true) {
  var value = 10;
}

console.log(
  value
);
```

Output:

```text
10
```

`var` does not create block scope.

---

## 10. let and const Are Block-Scoped

```js
if (true) {
  let value = 10;
}

console.log(
  value
);
```

Result:

```text
ReferenceError
```

---

## 11. Shadowing + TDZ Trap

```js
let value = 100;

{
  console.log(
    value
  );

  let value = 200;
}
```

This throws:

```text
ReferenceError
```

The inner `value` binding shadows the outer one for the whole block, but is uninitialized before its declaration.

---

## 12. Function Scope Trap

```js
var value = "global";

function test() {
  console.log(
    value
  );

  var value =
    "local";
}

test();
```

Output:

```text
undefined
```

The local `value` binding is created during function-context setup and shadows the global binding.

---

## 13. Class Hoisting

```js
const user =
  new User();

class User {}
```

Result:

```text
ReferenceError
```

Class declarations have TDZ behavior similar to `let` and `const`.

---

## 14. Hoisting Is Scope-Specific

Each execution context / lexical scope performs its own declaration setup.

Hoisting is not one global pass over the whole program.

---

## 15. Parameters and Hoisting

```js
function test(
  value
) {
  console.log(
    value
  );

  var value = 20;
}

test(10);
```

Output:

```text
10
```

The parameter is already initialized when the function begins.

---

## 16. Interview Table

| Declaration | Binding Created Before Execution? | Initial State Before Declaration |
| --- | --- | --- |
| `var` | Yes | `undefined` |
| `let` | Yes | Uninitialized / TDZ |
| `const` | Yes | Uninitialized / TDZ |
| Function declaration | Yes | Function object |
| Class declaration | Yes | Uninitialized / TDZ |

---

## 17. Common Interview Outputs

```js
console.log(a);
var a = 10;
```

→ `undefined`

```js
console.log(a);
let a = 10;
```

→ `ReferenceError`

```js
hello();
function hello() {}
```

→ works

```js
hello();
var hello = function () {};
```

→ `TypeError`

---

## Interview Answer — 30 Seconds

> Hoisting is the effect of declaration bindings being created during execution-context or lexical-environment setup before statement execution begins. `var` is initialized to `undefined`, function declarations are initialized with their function objects, while `let`, `const`, and classes remain uninitialized until their declarations execute.

---

## Key Takeaways

- Hoisting is binding setup, not physical source movement.
- `var` is initialized to `undefined`.
- `let`, `const`, and classes remain uninitialized.
- Function declarations are initialized with function objects.
- Function expressions follow the rules of their variable declaration.
- TDZ and shadowing explain many tricky outputs.
