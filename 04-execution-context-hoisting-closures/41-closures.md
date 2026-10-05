# Lesson 41 — Closures

Closures are one of the most important JavaScript concepts.

They appear everywhere in:

- React
- event handlers
- callbacks
- timers
- modules
- factory functions
- memoization
- private state
- asynchronous code

A closure is not a special syntax. It is a consequence of **lexical scoping** and functions retaining access to their lexical environment.

---

## 1. Definition

A practical definition:

> A closure is created when a function retains access to bindings from the lexical environment in which it was created, even when that function executes elsewhere or later.

Start with:

```js
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message);
  }

  inner();
}

outer();
```

`inner` can access `message` because it is lexically defined inside `outer`.

But the power of closures becomes clearer when `inner` escapes.

---

## 2. Function Returned from Another Function

```js
function outer() {
  const message = "Hello";

  return function inner() {
    console.log(message);
  };
}

const fn = outer();

fn();
```

Output:

```text
Hello
```

Important sequence:

```text
outer() called
     ↓
message = "Hello"
     ↓
inner function created
     ↓
inner returned
     ↓
outer() finishes
     ↓
later: fn() called
     ↓
inner still accesses message
```

How?

Because the returned function retains the lexical access it needs.

---

## 3. Closure Mental Model

```text
const fn = outer();

fn
│
└──→ inner function
        │
        └── closure / lexical access
                 │
                 ↓
          message binding
          "Hello"
```

The important point is not that the old function call remains on the call stack.

It does not.

`outer()` has completed and its active execution context has been popped.

What remains reachable is the lexical environment/bindings required by the returned function.

---

## 4. Closures Remember Bindings, Not Frozen Snapshots

This is a very important detail.

```js
function createCounter() {
  let count = 0;

  return function () {
    count++;

    return count;
  };
}

const counter = createCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

The closure accesses the same `count` binding across calls.

It does not receive a permanently frozen copy of `0`.

```text
closure
   ↓
count binding
   ↓
0 → 1 → 2 → 3
```

---

## 5. Independent Closures

```js
function createCounter() {
  let count = 0;

  return () => ++count;
}

const counterA = createCounter();
const counterB = createCounter();

console.log(counterA()); // 1
console.log(counterA()); // 2

console.log(counterB()); // 1
```

Why does `counterB` start from 1?

Each call to `createCounter()` creates a separate lexical environment.

```text
counterA
   ↓
environment A
count = 2

counterB
   ↓
environment B
count = 1
```

This is a powerful pattern for encapsulated state.

---

## 6. Private State with Closures

```js
function createBankAccount(
  initialBalance
) {
  let balance = initialBalance;

  return {
    deposit(amount) {
      balance += amount;
    },

    getBalance() {
      return balance;
    },
  };
}

const account =
  createBankAccount(1000);

account.deposit(500);

console.log(
  account.getBalance()
); // 1500
```

Outside code cannot directly access the local `balance` binding.

But returned methods can access it through closure.

Conceptually:

```text
account
│
├── deposit ─────┐
│                │
└── getBalance ──┤
                 ↓
          shared lexical environment
                 ↓
            balance = 1500
```

This provides encapsulation.

---

## 7. Closures with Function Factories

```js
function multiplyBy(multiplier) {
  return function (number) {
    return number * multiplier;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

Each returned function closes over its own `multiplier`.

```text
double → multiplier = 2
triple → multiplier = 3
```

This is called a **function factory** pattern.

---

## 8. Closures with Event Handlers

```js
function setupButton(userId) {
  const button =
    document.querySelector("#button");

  button.addEventListener(
    "click",
    function () {
      console.log(
        "User:",
        userId
      );
    }
  );
}

setupButton(123);
```

The click callback runs later.

Yet it still accesses `userId`.

That is closure behavior.

---

## 9. Closures with setTimeout

```js
function greet(name) {
  setTimeout(() => {
    console.log(
      `Hello ${name}`
    );
  }, 1000);
}

greet("Vikash");
```

The `greet` function finishes before the timer callback executes.

Still, the callback can access `name`.

```text
greet("Vikash")
     ↓
callback created
     ↓
callback closes over name
     ↓
greet finishes
     ↓
timer fires later
     ↓
callback reads name
```

---

## 10. Classic var Loop Closure Problem

Consider:

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}
```

A common result is:

```text
3
3
3
```

Why?

`var` is function-scoped, so the callbacks share the same `i` binding.

By the time callbacks run, the loop has completed and `i` is `3`.

Conceptually:

```text
callback 1 ──┐
callback 2 ──┼──→ same i binding → 3
callback 3 ──┘
```

---

## 11. let Fixes the Loop Case

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 0);
}
```

Output:

```text
0
1
2
```

For a `for` loop with a lexical declaration, JavaScript creates per-iteration lexical bindings as required by the language semantics.

Conceptually:

```text
callback 1 → i = 0
callback 2 → i = 1
callback 3 → i = 2
```

Each callback closes over the corresponding iteration binding.

---

## 12. Old-School IIFE Fix

Before `let` was available, developers often created a new function scope:

```js
for (var i = 0; i < 3; i++) {
  (function (value) {
    setTimeout(() => {
      console.log(value);
    }, 0);
  })(i);
}
```

Each IIFE invocation receives a separate `value` parameter.

Modern code generally uses `let` for this case.

---

## 13. Closure Does Not Mean "Function After Parent Returns"

Closures exist because functions have lexical environments.

A function does not need to be returned to technically participate in closure behavior.

Example:

```js
function outer() {
  const message = "Hello";

  function inner() {
    console.log(message);
  }

  inner();
}
```

`inner` accesses a binding from its surrounding lexical environment.

Returning the function simply makes closure behavior easier to observe because the function continues using that lexical access after the outer call finishes.

---

## 14. Closures and Memory

```js
function createHandler() {
  const data = {
    name: "Vikash",
  };

  return function () {
    console.log(data.name);
  };
}

const handler = createHandler();
```

As long as `handler` remains reachable and depends on `data`, the relevant binding/data must remain reachable.

This connects closures directly with Lesson 40.

But remember:

> Closure retention is normal behavior, not automatically a memory leak.

It becomes a problem only when references are retained unintentionally.

---

# Closures in React

This is one of the most important practical applications.

## 15. Every Render Creates New Local Values

Consider conceptually:

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  const handleClick = () => {
    console.log(count);
  };

  return (
    <button onClick={handleClick}>
      {count}
    </button>
  );
}
```

Each component invocation/render creates a new `handleClick` function.

That function closes over values from that render.

Simplified:

```text
Render 1
count = 0
handleClick #1 → sees count 0

Render 2
count = 1
handleClick #2 → sees count 1

Render 3
count = 2
handleClick #3 → sees count 2
```

This is fundamental to React's function-component model.

---

## 16. Stale Closure Example

Consider:

```jsx
function Counter() {
  const [count, setCount] =
    useState(0);

  function handleLater() {
    setTimeout(() => {
      console.log(count);
    }, 3000);
  }

  return (
    <>
      <button
        onClick={() =>
          setCount((c) => c + 1)
        }
      >
        Increment
      </button>

      <button onClick={handleLater}>
        Log later
      </button>
    </>
  );
}
```

Suppose:

```text
count = 0
↓
user clicks "Log later"
↓
callback created with lexical access
to that render's count
↓
user increments count
↓
new render has count = 1
↓
old timeout callback executes
↓
it still belongs to the earlier render
```

It may log:

```text
0
```

This is commonly described as a **stale closure**.

The callback did not magically update to use variables from a future render.

---

## 17. Why Dependency Arrays Matter

Closures are also part of understanding hooks such as:

```js
useEffect
useCallback
useMemo
```

Example:

```jsx
useEffect(() => {
  console.log(count);
}, [count]);
```

The effect callback is created during a render and closes over that render's values.

Dependencies help React know when the effect should be synchronized again using a callback from a newer render.

We will study React-specific closure behavior more deeply in the later **JavaScript Behind React** section.

---

## 18. Closure vs Scope

Scope tells us:

> Where can an identifier be resolved?

Closure describes:

> A function retaining access to its lexical environment/bindings when the function is used.

They are deeply related:

```text
Lexical Scope
      ↓
Lexical Environment
      ↓
Scope Chain
      ↓
Function retains lexical access
      ↓
Closure
```

This is why the previous lessons were taught before closures.

---

## 19. Full Section Mental Model

You can now connect Lessons 32–41:

```text
JavaScript Engine / Runtime
          ↓
Execution Context
          ↓
Creation + Execution
          ↓
Hoisting
          ↓
TDZ
          ↓
Lexical Environment
          ↓
Scope Chain
          ↓
Closures
          ↓
Retained reachable data
          ↓
Memory / Garbage Collection
```

And separately:

```text
Function call
    ↓
Execution Context
    ↓
Call Stack
    ↓
function completes
    ↓
stack frame removed

BUT

if a returned/reachable function
needs outer lexical bindings
    ↓
those bindings remain reachable
through closure
```

This distinction is extremely important.

---

## Interview Perspective

**What is a closure?**

A closure is the behavior by which a function retains access to bindings from its lexical environment even when the function executes outside or after the surrounding code's execution.

**Does a closure store a frozen copy of variables?**

No. A closure retains access to lexical bindings. Those bindings can change.

**Why does the var loop print 3 three times?**

The callbacks close over the same function-scoped `var i` binding. By the time they execute, the loop has finished and that binding contains `3`.

**Why does let behave differently in the loop?**

Lexical `let` loop declarations provide per-iteration bindings, so each callback closes over the binding for its iteration.

**Why are closures important in React?**

Event handlers, effects and other callbacks created during a render close over values from that render. This explains behavior such as stale closures and why hook dependencies matter.

---

## Key Takeaways

- Closures come from lexical scoping.
- Functions retain access to lexical bindings they need.
- The outer function does not need to remain on the call stack.
- Closures retain bindings, not frozen value snapshots.
- Different factory calls can create independent closure state.
- Closures enable private state and function factories.
- Callbacks, timers and event handlers frequently use closures.
- `var` and `let` behave differently in loop closure scenarios.
- Closures are fundamental to React function components and hooks.
- Closure retention is normal; it is not automatically a memory leak.
