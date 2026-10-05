# Lesson 09 — Conditional Statements

Conditional statements allow JavaScript to execute different code depending on a condition.

## if

```js
const age = 20;

if (age >= 18) {
  console.log("Adult");
}
```

## if...else

```js
const age = 16;

if (age >= 18) {
  console.log("Adult");
} else {
  console.log("Minor");
}
```

## else if

Use `else if` when there are multiple conditions.

```js
const score = 75;

if (score >= 90) {
  console.log("Excellent");
} else if (score >= 60) {
  console.log("Good");
} else {
  console.log("Needs improvement");
}
```

## switch

`switch` can be useful when one value has several possible cases.

```js
const role = "admin";

switch (role) {
  case "admin":
    console.log("Admin dashboard");
    break;

  case "user":
    console.log("User dashboard");
    break;

  default:
    console.log("Unknown role");
}
```

The `break` prevents execution from continuing into the next case.

## Ternary Operator

Useful when choosing between two simple values.

```js
const isLoggedIn = true;

const message = isLoggedIn ? "Welcome" : "Please login";
```

Avoid deeply nested ternary expressions because they can make code difficult to read.

## Truthy and Falsy Conditions

Conditions do not always need to contain an explicit comparison.

```js
const username = "Vikash";

if (username) {
  console.log("Username exists");
}
```

## Key Takeaways

- `if` executes code when a condition is truthy.
- `else` handles the alternative.
- `else if` handles multiple conditions.
- `switch` is useful for several cases of the same value.
- Ternaries are useful for simple conditional values.

## Interview Quick Answer

**When should you use switch instead of if/else?**

`switch` can improve readability when the same expression is being compared against several known values.
