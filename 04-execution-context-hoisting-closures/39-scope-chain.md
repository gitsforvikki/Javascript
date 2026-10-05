# Lesson 39 — Scope Chain and Identifier Resolution

The **scope chain** describes how JavaScript searches lexical environments when resolving an identifier.

This lesson connects:

- lexical scope
- lexical environments
- shadowing
- TDZ
- closures

---

## 1. Basic Example

```js
const language = "JavaScript";

function outer() {
  const framework = "React";

  function inner() {
    const version = 19;

    console.log(language);
    console.log(framework);
    console.log(version);
  }

  inner();
}

outer();
```

When `inner` needs `framework`, JavaScript searches outward through lexical environments.

```text
inner environment
│
├── version
│
└── framework? NO
        ↓
outer environment
│
├── framework ✓
│
└── search stops
```

For `language`:

```text
inner
  ↓ not found
outer
  ↓ not found
global
  ↓ found
language = "JavaScript"
```

---

## 2. Identifier Resolution

Suppose JavaScript evaluates:

```js
console.log(value);
```

Conceptually, it asks:

```text
Is value in current lexical environment?
        │
        ├── YES → use it
        │
        └── NO
             ↓
        check outer environment
             ↓
        continue outward
             ↓
        found → use it
             ↓
        not found anywhere
             ↓
        ReferenceError
```

This outward lookup path is the scope chain.

---

## 3. JavaScript Searches Outward, Not Inward

```js
function outer() {
  const outerValue = 10;

  function inner() {
    const innerValue = 20;

    console.log(outerValue);
  }

  inner();

  console.log(innerValue);
}

outer();
```

Inside `inner`, `outerValue` is accessible.

But `outer` cannot access `innerValue`.

```text
inner → outer → global
```

There is no reverse lookup:

```text
outer ✕→ inner
```

Scope resolution moves outward through lexical parents.

---

## 4. Variable Shadowing

An inner scope can define the same identifier as an outer scope.

```js
const name = "Global";

function showName() {
  const name = "Local";

  console.log(name);
}

showName();
```

Output:

```text
Local
```

Lookup:

```text
showName environment
│
└── name found → "Local"

STOP
```

JavaScript does not continue looking outward after finding the binding.

This is called **shadowing**.

---

## 5. Scope Chain + TDZ

Consider:

```js
const value = "global";

function test() {
  console.log(value);

  const value = "local";
}

test();
```

This throws a `ReferenceError`.

Why doesn't JavaScript use the global `value`?

Because the local lexical environment already contains a `value` binding.

```text
test environment
│
└── value found
      ↓
   uninitialized
      ↓
     TDZ
      ↓
ReferenceError
```

Lookup stops when the binding is found, even though it cannot currently be accessed.

This is why TDZ and scope chain must be understood together.

---

## 6. Lexical Scope, Not Dynamic Scope

Consider:

```js
const value = "global";

function printValue() {
  console.log(value);
}

function execute() {
  const value = "execute";

  printValue();
}

execute();
```

Output:

```text
global
```

If JavaScript used the caller's scope dynamically, it might print `execute`.

But JavaScript uses lexical scoping.

`printValue` was defined globally:

```text
printValue
    ↓ lexical outer
global environment
```

The fact that `execute` calls it does not change that relationship.

---

## 7. Multiple Nested Scopes

```js
const a = 1;

function first() {
  const b = 2;

  function second() {
    const c = 3;

    function third() {
      const d = 4;

      console.log(a, b, c, d);
    }

    third();
  }

  second();
}

first();
```

Inside `third`:

```text
third
│ d
↓
second
│ c
↓
first
│ b
↓
global
│ a
```

Each identifier is resolved by walking outward until its binding is found.

---

## 8. Scope Chain vs Call Stack

These are frequently confused.

### Call Stack

Tracks **which functions are currently executing**.

```text
third()
second()
first()
global
```

### Scope Chain

Determines **where identifiers are looked up**.

```text
third lexical environment
        ↓
second lexical environment
        ↓
first lexical environment
        ↓
global lexical environment
```

They may look similar in simple nested examples, but they represent different concepts.

A function can be called from somewhere that is not its lexical parent.

---

## 9. Example Proving the Difference

```js
const value = "global";

function read() {
  console.log(value);
}

function caller() {
  const value = "caller";

  read();
}

caller();
```

At the moment `read()` executes, the call stack is conceptually:

```text
read
caller
global
```

But `read`'s lexical scope chain is:

```text
read environment
      ↓
global environment
```

It does not include `caller` as its lexical parent.

Therefore output is:

```text
global
```

This distinction is extremely important.

---

## 10. Scope Chain and Closures

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

When `fn()` executes later, its lexical scope still leads to the environment containing `message`.

That preserved lexical access is closure behavior.

---

## 11. React Connection

Every render of a component creates new local values.

```jsx
function Counter({ count }) {
  const message = `Count: ${count}`;

  function handleClick() {
    console.log(message);
  }

  return (
    <button onClick={handleClick}>
      Log
    </button>
  );
}
```

`handleClick` resolves `message` through the lexical environment where that particular handler was created.

This becomes critical when understanding stale closures.

---

## Interview Perspective

**What is the scope chain?**

The scope chain is the sequence of lexical environments JavaScript searches outward to resolve an identifier.

**Does JavaScript search child scopes?**

No. Identifier lookup proceeds from the current lexical environment outward through its lexical parents.

**What is the difference between call stack and scope chain?**

The call stack tracks active function execution. The scope chain controls lexical identifier lookup. A caller is not necessarily a function's lexical parent.

---

## Key Takeaways

- JavaScript resolves identifiers from the current scope outward.
- Lookup stops at the first matching binding.
- Inner declarations can shadow outer declarations.
- A binding in TDZ blocks lookup from continuing outward.
- JavaScript uses lexical, not dynamic, scope.
- Scope chain and call stack are different concepts.
- Closures preserve access through lexical scope relationships.
