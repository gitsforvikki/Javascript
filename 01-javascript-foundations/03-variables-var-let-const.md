# Lesson 03 — Variables: var, let and const

Variables are used to store values.

JavaScript provides three keywords:

```js
var oldWay = "JavaScript";

let age = 25;

const name = "Vikash";
```

## let

Use `let` when the variable needs to be reassigned.

```js
let count = 1;

count = 2;

console.log(count); // 2
```

## const

Use `const` when the variable should not be reassigned.

```js
const name = "Vikash";

// name = "Rahul"; // TypeError
```

For objects and arrays, `const` prevents reassignment of the variable, but the contents can still change.

```js
const user = {
  name: "Vikash",
};

user.name = "Kumar"; // allowed
```

## var

`var` is the older way of declaring variables.

```js
var language = "JavaScript";
```

Important basic difference:

| Feature | var | let | const |
| --- | --- | --- | --- |
| Function scoped | Yes | No | No |
| Block scoped | No | Yes | Yes |
| Can reassign | Yes | Yes | No |
| Can redeclare in same scope | Yes | No | No |

We will study scope, hoisting and the Temporal Dead Zone deeply in later lessons.

## Recommended Practice

Prefer:

```js
const user = "Vikash";

let count = 0;
```

Use `const` by default and `let` when reassignment is required. Avoid `var` in modern JavaScript unless you specifically need to understand older code.

## Key Takeaways

- `var`, `let`, and `const` declare variables.
- `let` and `const` are block-scoped.
- `var` is function-scoped.
- `const` prevents reassignment.
- Prefer `const`, then `let`.

## Interview Quick Answer

**Difference between var, let and const?**

`var` is function-scoped and can be redeclared. `let` and `const` are block-scoped. `let` can be reassigned, while `const` cannot be reassigned.
