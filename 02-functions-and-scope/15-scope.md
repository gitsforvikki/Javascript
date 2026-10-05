# Lesson 15 — Global, Function and Block Scope

Scope determines **where a variable can be accessed in your program**.

This is a foundational JavaScript concept. Closures, lexical scope, hoisting and many React behaviors depend on understanding scope correctly.

## Mental Model

Think of scope as an accessibility boundary.

```text
Variable is declared
       ↓
JavaScript determines its scope
       ↓
Only code allowed by that scope
can access the variable
```

JavaScript commonly involves:

1. Global scope
2. Function scope
3. Block scope

## 1. Global Scope

A variable declared outside functions and blocks belongs to the surrounding global/module-level scope.

```js
const appName = "CareerLoop";

function showAppName() {
  console.log(appName);
}

showAppName(); // CareerLoop
```

The function can access `appName` because the variable exists in an outer scope.

Conceptually:

```text
Global Scope
│
├── appName
│
└── showAppName()
      │
      └── can access appName
```

Avoid unnecessary global variables because many parts of an application may be able to depend on or modify shared global state.

## 2. Function Scope

Variables declared inside a function are accessible within that function, not outside it.

```js
function greet() {
  const message = "Hello";

  console.log(message);
}

greet();

console.log(message); // ReferenceError
```

Conceptually:

```text
Global Scope
│
└── greet()
     │
     └── Function Scope
          └── message
```

The global scope cannot directly access `message`.

### var Is Function-Scoped

```js
function test() {
  var name = "Vikash";

  console.log(name);
}

console.log(name); // not accessible here
```

Even though `var` has older behavior, inside a function it remains limited to that function.

## 3. Block Scope

A block is commonly created using curly braces:

```js
{
  // block
}
```

`let` and `const` are block-scoped.

```js
if (true) {
  const name = "Vikash";
  let age = 25;

  console.log(name);
  console.log(age);
}

console.log(name); // ReferenceError
console.log(age);  // ReferenceError
```

The variables exist only inside the block.

Blocks commonly appear in:

```js
if (...) {}
for (...) {}
while (...) {}
switch (...) {}
{}
```

## var and Block Scope

`var` is not block-scoped.

```js
if (true) {
  var language = "JavaScript";
}

console.log(language); // JavaScript
```

Compare with `let`:

```js
if (true) {
  let language = "JavaScript";
}

console.log(language); // ReferenceError
```

This is one important reason modern JavaScript prefers `let` and `const`.

## Nested Scope

Scopes can exist inside other scopes.

```js
const app = "CareerLoop";

function dashboard() {
  const user = "Vikash";

  if (true) {
    const role = "Developer";

    console.log(app);
    console.log(user);
    console.log(role);
  }
}
```

The inner block can access variables from outer scopes.

Visual model:

```text
Global Scope
│
├── app
│
└── dashboard Function Scope
     │
     ├── user
     │
     └── if Block Scope
          │
          └── role
```

Inside the block:

```text
role ✓
user ✓
app  ✓
```

But outer scopes cannot access variables declared only in inner scopes.

```js
function test() {
  if (true) {
    const value = 100;
  }

  console.log(value); // ReferenceError
}
```

## Variable Shadowing

An inner scope can declare a variable with the same name as an outer variable.

```js
const name = "Global";

function test() {
  const name = "Function";

  console.log(name);
}

test();            // Function
console.log(name); // Global
```

The inner variable **shadows** the outer variable within that scope.

Visual:

```text
Global Scope
name = "Global"

      ↓ enter function

Function Scope
name = "Function"
      ↑
this name is used here
```

The outer variable is not changed.

## Scope Lookup

When JavaScript encounters a variable, it first checks the current scope.

If the variable is not found, it looks outward.

```js
const language = "JavaScript";

function outer() {
  const framework = "React";

  function inner() {
    console.log(framework);
    console.log(language);
  }

  inner();
}

outer();
```

Conceptually:

```text
inner scope
   │
   │ not found?
   ↓
outer function scope
   │
   │ not found?
   ↓
global scope
```

This outward lookup forms the basis of the **scope chain**, which is the subject of the next conceptual lesson.

## Scope Is Based on Where Code Is Written

JavaScript uses lexical scoping.

That means scope relationships are determined by where functions and blocks are written in the source code.

Example:

```js
const name = "Vikash";

function outer() {
  const role = "Developer";

  function inner() {
    console.log(name);
    console.log(role);
  }

  inner();
}
```

`inner` can access `role` because it was defined inside the scope containing `role`.

We will study lexical scope and the scope chain deeply in Lesson 16.

## React Connection

Scope appears constantly in React.

```jsx
function Counter() {
  const count = 10;

  function handleClick() {
    console.log(count);
  }

  return <button onClick={handleClick}>Click</button>;
}
```

`handleClick` can access `count` because of lexical scope.

This becomes especially important when understanding **closures and stale state** in React.

## Common Mistakes

### Expecting block variables outside the block

```js
if (true) {
  const user = "Vikash";
}

console.log(user); // ReferenceError
```

### Assuming var is block-scoped

```js
for (var i = 0; i < 3; i++) {}

console.log(i); // 3
```

With `let`:

```js
for (let i = 0; i < 3; i++) {}

console.log(i); // ReferenceError
```

### Creating unnecessary globals

```js
let currentUser = "Vikash";
let currentRole = "Developer";
```

Prefer keeping values inside the smallest reasonable scope.

## Interview Perspective

**What is scope in JavaScript?**

Scope determines where variables and functions are accessible in a program.

**What is the difference between function scope and block scope?**

Function-scoped variables are available throughout their containing function. Block-scoped variables are limited to the block in which they are declared. `var` is function-scoped, while `let` and `const` are block-scoped.

**Can an inner scope access an outer variable?**

Yes. JavaScript can search outward through surrounding lexical scopes. Outer scopes cannot directly access variables that exist only inside an inner scope.

## Key Takeaways

- Scope controls variable accessibility.
- Global scope is the outermost surrounding scope.
- Functions create function scopes.
- Blocks create scope for `let` and `const`.
- `var` is function-scoped rather than block-scoped.
- Inner scopes can access appropriate outer variables.
- Outer scopes cannot directly access inner variables.
- Variable shadowing occurs when an inner scope declares the same name.
- Scope lookup proceeds outward through surrounding scopes.
- Scope is the foundation for lexical scope, scope chains and closures.
