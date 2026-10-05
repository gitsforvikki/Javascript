# Lesson 19 — Pure vs Impure Functions

Pure functions are important for writing predictable code.

This concept is especially useful in functional programming, state management, React and testing.

## What is a Pure Function?

A function is considered pure when:

1. The same inputs always produce the same output.
2. It does not create observable side effects outside the function.

Example:

```js
function add(a, b) {
  return a + b;
}
```

Calling:

```js
add(2, 3); // 5
add(2, 3); // 5
add(2, 3); // 5
```

The same arguments always produce the same result.

The function also does not modify anything outside itself.

## What is a Side Effect?

A side effect occurs when a function interacts with or changes something outside its returned result.

Common examples include:

- modifying an external variable
- mutating an object passed from outside
- modifying the DOM
- writing to storage
- making a network request
- logging to the console
- changing a database
- setting timers

Not all side effects are bad. Real applications need side effects.

The important idea is to **control where they happen**.

## Impure Function Example: External State

```js
let total = 0;

function addToTotal(value) {
  total += value;

  return total;
}
```

The result depends on external state.

```js
addToTotal(10); // 10
addToTotal(10); // 20
```

Same input, different output.

This function is impure.

## Pure Alternative

```js
function add(total, value) {
  return total + value;
}

console.log(add(0, 10)); // 10
console.log(add(0, 10)); // 10
```

All required information comes through parameters.

## Mutation Can Make a Function Impure

Consider:

```js
function updateUser(user) {
  user.name = "Kumar";

  return user;
}
```

The function modifies the original object.

```js
const user = {
  name: "Vikash",
};

updateUser(user);

console.log(user.name); // Kumar
```

The function changed data outside its own local state.

## Pure Immutable Version

```js
function updateUser(user) {
  return {
    ...user,
    name: "Kumar",
  };
}
```

Now:

```js
const user = {
  name: "Vikash",
};

const updatedUser = updateUser(user);

console.log(user.name);        // Vikash
console.log(updatedUser.name); // Kumar
```

The original object remains unchanged.

## Why Pure Functions Are Useful

### Predictability

```text
same input
    ↓
pure function
    ↓
same output
```

This makes behavior easier to reason about.

### Testing

Pure functions are easy to test.

```js
expect(add(2, 3)).toBe(5);
```

No database, browser state or external setup is needed.

### Reusability

Because they depend mainly on their inputs, pure functions can be reused safely in different places.

### Debugging

When output depends only on explicit inputs, fewer hidden dependencies need to be investigated.

## Impure Functions Are Necessary

Applications cannot be completely pure.

For example:

```js
async function saveUser(user) {
  await fetch("/api/users", {
    method: "POST",
    body: JSON.stringify(user),
  });
}
```

This function performs a network side effect.

Other necessary side effects include:

```text
DOM updates
API requests
database operations
localStorage
cookies
timers
logging
file operations
```

A good architecture usually keeps pure calculation logic separate from side-effect logic when practical.

## React Connection

React strongly benefits from pure rendering logic.

Consider:

```jsx
function Greeting({ name }) {
  return <h1>Hello {name}</h1>;
}
```

For the same `name`, the component's rendering logic should produce the same UI description.

Avoid changing unrelated external state while rendering.

React state updates also commonly use immutable patterns:

```js
setUser({
  ...user,
  name: "Kumar",
});
```

rather than:

```js
user.name = "Kumar";
setUser(user);
```

Purity and immutability are related, but they are not exactly the same concept. We will study immutability deeply later.

## Pure vs Impure

| Pure Function | Impure Function |
| --- | --- |
| Same input → same output | Output may depend on external state |
| Avoids observable side effects | May produce side effects |
| Does not mutate external data | May mutate external data |
| Easier to test | Often requires more setup |
| Easier to reason about | Can have hidden dependencies |

## Important Nuance

A function containing a local variable is not automatically impure.

```js
function calculate(a, b) {
  const total = a + b;

  return total;
}
```

The local variable exists only for the calculation and does not create an external side effect.

## Interview Perspective

**What is a pure function?**

A pure function produces the same output for the same inputs and does not create observable side effects outside the function.

**Are side effects always bad?**

No. Side effects are necessary for real applications, such as API calls and DOM updates. The goal is to isolate and manage them rather than pretend they do not exist.

## Key Takeaways

- Pure functions produce predictable results.
- Same inputs should produce the same output.
- Pure functions avoid observable external side effects.
- Mutating external objects can make functions impure.
- Pure functions are easier to test and reason about.
- Real applications still require controlled side effects.
- Purity is especially relevant to React and state management.
