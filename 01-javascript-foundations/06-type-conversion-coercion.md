# Lesson 06 — Type Conversion and Type Coercion

JavaScript sometimes needs to change a value from one data type to another.

There are two common ways this happens:

- **Type conversion** — we explicitly convert the value.
- **Type coercion** — JavaScript automatically converts the value.

## Type Conversion

We manually convert a value using functions such as `String()`, `Number()`, and `Boolean()`.

```js
const age = "25";

const numberAge = Number(age);

console.log(numberAge); // 25
console.log(typeof numberAge); // "number"
```

More examples:

```js
String(100);      // "100"
Number("50");     // 50
Boolean(1);       // true
Boolean(0);       // false
```

## Type Coercion

JavaScript can automatically convert values during an operation.

```js
console.log("5" + 2); // "52"
```

Here JavaScript converts `2` into a string.

Another example:

```js
console.log("5" - 2); // 3
```

JavaScript converts `"5"` into a number.

## Truthy and Falsy Values

JavaScript can convert values to booleans when evaluating conditions.

Common falsy values include:

```js
false
0
""
null
undefined
NaN
```

Most other values are truthy.

```js
if ("JavaScript") {
  console.log("This runs");
}
```

## Recommended Practice

Prefer explicit conversion when possible because it makes code easier to understand.

```js
const price = Number("100");
```

We will study coercion rules and tricky output questions more deeply during interview preparation.

## Key Takeaways

- Type conversion is explicit.
- Type coercion happens automatically.
- `String()`, `Number()`, and `Boolean()` are commonly used for conversion.
- Truthy and falsy values are important in conditions.

## Interview Quick Answer

**What is the difference between type conversion and type coercion?**

Type conversion is performed explicitly by the developer, while type coercion happens automatically by JavaScript.
