# Lesson 08 — Equality: == vs === and Object.is

JavaScript provides different ways to compare values.

## Loose Equality — ==

The `==` operator compares values after allowing type coercion.

```js
console.log(5 == "5"); // true
```

The string `"5"` is converted during comparison.

## Strict Equality — ===

The `===` operator compares values without performing type coercion.

```js
console.log(5 === "5"); // false
console.log(5 === 5);   // true
```

In modern JavaScript, prefer `===` for most comparisons because its behavior is more predictable.

## Inequality

```js
5 != "5";  // false
5 !== "5"; // true
```

Prefer strict inequality `!==` for the same reason.

## Objects and Equality

Objects are compared by reference.

```js
const user1 = { name: "Vikash" };
const user2 = { name: "Vikash" };

console.log(user1 === user2); // false
```

These are two separate objects.

```js
const user1 = { name: "Vikash" };
const user2 = user1;

console.log(user1 === user2); // true
```

Now both variables reference the same object.

## Object.is()

`Object.is()` is another value comparison method.

```js
Object.is(10, 10); // true
Object.is("JS", "JS"); // true
```

For everyday code, `===` is usually what you need.

One notable difference:

```js
NaN === NaN;          // false
Object.is(NaN, NaN);  // true
```

## Recommended Practice

Use:

```js
value === expectedValue
value !== expectedValue
```

unless you intentionally need another comparison behavior.

## Key Takeaways

- `==` allows type coercion.
- `===` compares without type coercion.
- Prefer `===` and `!==` in most application code.
- Objects are compared by reference.
- `Object.is()` handles a few special cases differently.

## Interview Quick Answer

**What is the difference between == and ===?**

`==` performs type coercion before comparison when needed, while `===` compares values without coercing their types.
