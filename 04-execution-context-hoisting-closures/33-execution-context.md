# Lesson 33 — Execution Context

Execution context is one of the most important JavaScript concepts.

It explains how JavaScript keeps track of:

- variables
- functions
- scope
- currently executing code
- outer lexical relationships

Understanding execution context makes hoisting, scope, closures and the call stack much easier.

---

# 1. What is an Execution Context?

An **execution context** is the environment in which JavaScript code is evaluated and executed.

A useful mental model is:

```text
Execution Context
│
├── variables / bindings
├── functions
├── lexical environment
├── outer-scope relationship
├── this-related information
└── currently executing code
```

This is a conceptual model. JavaScript engines may implement these details differently internally.

---

# 2. Global Execution Context

When a JavaScript script begins running, a global execution context is established.

Example:

```js
const name = "Vikash";

function greet() {
  console.log("Hello");
}

console.log(name);
```

Conceptually:

```text
Global Execution Context
│
├── name
├── greet
└── currently executing global code
```

There is one global execution context for the relevant script/global realm execution.

---

# 3. Function Execution Context

Whenever a function is called, JavaScript creates a new function execution context for that invocation.

```js
function greet(name) {
  const message = `Hello ${name}`;

  console.log(message);
}

greet("Vikash");
```

Conceptually:

```text
Global Execution Context
│
└── greet()
       ↓ function called

Function Execution Context
│
├── name = "Vikash"
├── message
└── function body execution
```

When the function finishes, its execution context is removed from active execution.

But some data associated with its lexical environment can remain reachable when closures require it. We will study that carefully later.

---

# 4. Every Function Call Gets Its Own Context

Consider:

```js
function add(a, b) {
  const result = a + b;

  return result;
}

const first = add(10, 20);
const second = add(5, 7);
```

The two calls are separate executions.

```text
add(10, 20)
│
├── a = 10
├── b = 20
└── result = 30

add(5, 7)
│
├── a = 5
├── b = 7
└── result = 12
```

Each invocation gets its own function execution context.

This is why recursive functions can have many active calls to the same function at once.

---

# 5. Nested Function Calls

Consider:

```js
function first() {
  const a = 10;

  second();
}

function second() {
  const b = 20;

  console.log(b);
}

first();
```

Execution begins globally.

Then:

```text
Global Context
     ↓
first() called
     ↓
First Function Context
     ↓
second() called
     ↓
Second Function Context
```

JavaScript needs a structure to track which context is currently active.

That structure is the **call stack**, which we will study in Lesson 35.

---

# 6. Execution Context and Scope Are Related but Different

Do not treat these terms as identical.

**Scope** answers:

> Where can this identifier be accessed?

**Execution context** answers more broadly:

> What environment is currently being used to execute this code?

Example:

```js
const globalValue = 10;

function outer() {
  const outerValue = 20;

  function inner() {
    const innerValue = 30;

    console.log(
      globalValue,
      outerValue,
      innerValue
    );
  }

  inner();
}

outer();
```

The `inner()` invocation has its own execution context, while lexical scoping determines that it can also resolve variables from its outer lexical environments.

---

# 7. Lexical Environment Inside the Model

A key part of execution is the lexical environment.

Simplified:

```text
Function Execution Context
│
├── local bindings
│   ├── parameters
│   ├── variables
│   └── declarations
│
└── outer lexical reference
        ↓
    parent lexical environment
```

This outer relationship is determined lexically—based on where the function was defined.

That is the foundation for:

- scope chain
- lexical scope
- closures

We already introduced lexical scope earlier and will go deeper later in this section.

---

# 8. Execution Context Lifecycle

For learning purposes, execution of a context is often explained in two broad stages:

```text
Execution Context
      │
      ├── Creation / Setup
      │
      └── Execution
```

During setup, bindings are established according to JavaScript declaration rules.

During execution, statements run and values are assigned/evaluated.

This model helps explain why:

```js
console.log(a);

var a = 10;
```

behaves differently from:

```js
console.log(b);

let b = 20;
```

We will study these phases in the next lesson.

---

# 9. Execution Context and this

Execution contexts also involve `this` behavior.

However, `this` follows rules based largely on function type and invocation style.

Because `this` deserves careful treatment, it has its own later section.

For now, remember that execution context is broader than simply a collection of local variables.

---

# 10. React Connection

Consider:

```jsx
function UserCard({ user }) {
  const displayName = user.name;

  function handleClick() {
    console.log(displayName);
  }

  return (
    <button onClick={handleClick}>
      {displayName}
    </button>
  );
}
```

Every component function invocation is JavaScript function execution.

During execution, local variables and functions are created for that render invocation.

The event handler can later retain access to values from the lexical environment in which it was created.

That is a closure.

This connection becomes very important for understanding stale closures in React.

---

# 11. Interview Perspective

**What is an execution context?**

An execution context is the environment JavaScript uses to evaluate and execute code, containing the bindings and execution information needed for that code.

**When is a function execution context created?**

A new function execution context is created each time a function is invoked.

**Are scope and execution context the same?**

No. Scope determines identifier accessibility, while execution context represents the environment associated with currently executing code.

---

# Key Takeaways

- JavaScript executes code inside execution contexts.
- Global code runs in a global execution context.
- Every function invocation gets its own function execution context.
- Separate calls have separate local execution state.
- Lexical environments connect execution with scope resolution.
- Execution context and scope are related but not identical.
- The call stack tracks active execution contexts.
- Execution contexts are the foundation for hoisting and closures.
