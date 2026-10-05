# Lesson 85 — Template Literals and Modern Syntax

Modern JavaScript introduced syntax that makes code shorter, clearer, and easier to maintain.

This lesson focuses on practical syntax you will see constantly in React, Next.js, Node.js, and interviews.

---

## 1. Template Literals

Template literals use backticks:

```js
const name = "Vikash";

const message =
  `Hello ${name}`;
```

Output:

```text
Hello Vikash
```

---

## 2. String Interpolation

Old style:

```js
const message =
  "Hello " +
  name +
  ", you are a " +
  role;
```

Modern:

```js
const message =
  `Hello ${name}, you are a ${role}`;
```

Interpolation can contain expressions:

```js
const total = 100;

console.log(
  `Final: ${total * 1.18}`
);
```

---

## 3. Multiline Strings

```js
const text = `
Line one
Line two
Line three
`;
```

No manual newline concatenation is required.

---

## 4. Expressions Inside ${}

```js
const user = {
  firstName: "Vikash",
  lastName: "Kumar",
};

const fullName =
  `${user.firstName} ${user.lastName}`;
```

You can call functions too:

```js
const result =
  `Total: ${calculateTotal()}`;
```

Keep expressions readable; do not put large business logic inside templates.

---

## 5. Tagged Template Literals

A function can process a template literal.

```js
function tag(
  strings,
  ...values
) {
  console.log(strings);
  console.log(values);
}

const name = "Vikash";
const role = "Developer";

tag`User: ${name}, Role: ${role}`;
```

Tagged templates are advanced but useful in libraries for:

- styling
- localization
- SQL/query builders
- safe formatting

---

## 6. Object Property Shorthand

Old:

```js
const name = "Vikash";
const role = "Developer";

const user = {
  name: name,
  role: role,
};
```

Modern:

```js
const user = {
  name,
  role,
};
```

---

## 7. Method Shorthand

Old:

```js
const user = {
  greet: function () {
    console.log("Hello");
  },
};
```

Modern:

```js
const user = {
  greet() {
    console.log("Hello");
  },
};
```

---

## 8. Computed Property Names

```js
const key = "role";

const user = {
  name: "Vikash",
  [key]: "Developer",
};
```

Result:

```js
{
  name: "Vikash",
  role: "Developer"
}
```

---

## 9. Destructuring Review

Array:

```js
const [
  first,
  second,
] = ["React", "Next.js"];
```

Object:

```js
const {
  name,
  role,
} = user;
```

Rename:

```js
const {
  name: userName,
} = user;
```

Default:

```js
const {
  city = "Bengaluru",
} = user;
```

---

## 10. Spread Syntax

Array:

```js
const a = [1, 2];

const b = [
  ...a,
  3,
  4,
];
```

Object:

```js
const updatedUser = {
  ...user,
  role: "Full Stack Developer",
};
```

Spread is shallow.

Nested references are still shared unless explicitly copied.

---

## 11. Rest Syntax

Function parameters:

```js
function sum(
  ...numbers
) {
  return numbers.reduce(
    (total, value) =>
      total + value,
    0
  );
}
```

Destructuring:

```js
const {
  id,
  ...rest
} = user;
```

---

## 12. Default Parameters

```js
function greet(
  name = "Guest"
) {
  return `Hello ${name}`;
}
```

Default applies when the argument is `undefined`.

```js
greet(undefined);
```

uses the default.

But:

```js
greet(null);
```

does not.

---

## 13. Optional Chaining

```js
const city =
  user.profile?.address?.city;
```

If an intermediate value is `null` or `undefined`, evaluation stops and returns `undefined`.

Function call:

```js
user.onSave?.();
```

---

## 14. Nullish Coalescing

```js
const value =
  input ?? "default";
```

Fallback is used only when the left side is:

- `null`
- `undefined`

Unlike `||`, values such as:

- `0`
- `false`
- `""`

are preserved.

---

## 15. Logical Assignment Operators

```js
value ||= fallback;
value ??= fallback;
value &&= nextValue;
```

Use them when they improve readability.

---

## 16. Numeric Separators

```js
const salary =
  1_000_000;

const billion =
  1_000_000_000;
```

The underscores improve readability and do not change the numeric value.

---

## 17. React Connection

Modern syntax appears constantly:

```jsx
function UserCard({
  name,
  role = "Developer",
}) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
    </div>
  );
}
```

State updates often use spread:

```js
setUser((prev) => ({
  ...prev,
  role: "Developer",
}));
```

---

## Interview Questions

### Template literals vs normal strings?

Template literals support interpolation, multiline content, and tagged templates.

### Spread vs rest?

Both use `...`, but spread expands values while rest collects values.

### `||` vs `??`?

`||` falls back on any falsy value. `??` falls back only on `null` or `undefined`.

---

## Key Takeaways

- Template literals use backticks.
- ${} evaluates expressions.
- Modern object syntax reduces repetition.
- Spread expands; rest collects.
- Optional chaining safely accesses nullable paths.
- Nullish coalescing preserves valid falsy values.
- Modern syntax improves clarity but does not replace understanding the underlying concepts.
