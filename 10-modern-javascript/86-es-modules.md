# Lesson 86 — ES Modules: `import` and `export`

ES Modules, usually called **ESM**, are JavaScript's standard module system.

Modules let you split code into separate files with explicit dependencies.

Core syntax:

```text
export
import
```

---

## 1. Why Modules?

Without modules, large applications can become one huge file.

Modules provide:

- separation of concerns
- reusable code
- explicit dependencies
- easier testing
- better tooling

---

## 2. Named Export

```js
// math.js

export function add(
  a,
  b
) {
  return a + b;
}

export const PI =
  3.14159;
```

Import:

```js
import {
  add,
  PI,
} from "./math.js";
```

Named imports must match exported names unless renamed.

---

## 3. Export Later

```js
function add(a, b) {
  return a + b;
}

const PI = 3.14159;

export {
  add,
  PI,
};
```

Same module semantics.

---

## 4. Rename Named Export

```js
export {
  add as sum,
};
```

Import:

```js
import {
  sum,
} from "./math.js";
```

---

## 5. Rename During Import

```js
import {
  add as addNumbers,
} from "./math.js";
```

Now use:

```js
addNumbers(1, 2);
```

---

## 6. Default Export

A module can have one default export.

```js
export default function User() {
  // ...
}
```

Import:

```js
import User
  from "./User.js";
```

The importer chooses the local name.

```js
import MyUser
  from "./User.js";
```

also works.

---

## 7. Named vs Default

Named:

```js
export const add = ...;
export const subtract = ...;
```

Import:

```js
import {
  add,
  subtract,
} from "./math.js";
```

Default:

```js
export default function App() {}
```

Import:

```js
import App
  from "./App.js";
```

---

## 8. Default + Named Together

```js
export default function User() {}

export const role =
  "Developer";
```

Import:

```js
import User, {
  role,
} from "./user.js";
```

---

## 9. Namespace Import

```js
import * as math
  from "./math.js";
```

Use:

```js
math.add(1, 2);
math.PI;
```

Useful when many related exports belong under one namespace.

---

## 10. Re-export

```js
export {
  add,
  subtract,
} from "./math.js";
```

A barrel module can re-export values from multiple files.

Example:

```js
// index.js

export {
  Button,
} from "./Button.js";

export {
  Modal,
} from "./Modal.js";
```

---

## 11. export *

```js
export *
  from "./math.js";
```

This re-exports named exports.

Default exports are not re-exported by `export *` in the same way.

---

## 12. Imports Are Static

Normal import declarations are statically analyzable:

```js
import {
  add,
} from "./math.js";
```

This enables tooling such as:

- tree shaking
- dependency analysis
- bundling optimization

---

## 13. Imports Are Live Bindings

This is important.

Suppose:

```js
// counter.js
export let count = 0;

export function increment() {
  count++;
}
```

Importer:

```js
import {
  count,
  increment,
} from "./counter.js";

console.log(count);

increment();

console.log(count);
```

The imported binding reflects the updated exported value.

Imports are not simple copied snapshots.

---

## 14. Imported Bindings Are Read-Only Locally

You cannot do:

```js
import {
  count,
} from "./counter.js";

count = 100;
```

The importing module cannot reassign that imported binding.

The exporting module controls it.

---

## 15. Module Scope

Variables declared inside a module are module-scoped.

```js
const secret =
  "internal";
```

This does not automatically become a global variable.

Only exported values become part of the module's public API.

---

## 16. ES Modules Are Strict Mode

ES modules execute in strict mode automatically.

So:

```js
this
```

at top level in an ES module is:

```text
undefined
```

This connects directly to your `this` lessons.

---

## 17. Browser ESM

HTML:

```html
<script
  type="module"
  src="./main.js"
></script>
```

Then inside `main.js`:

```js
import {
  add,
} from "./math.js";
```

Module scripts are deferred by default in browser loading behavior.

---

## 18. File Extensions in Browsers

Browser-native ESM usually requires explicit resolvable paths:

```js
import {
  add,
} from "./math.js";
```

A bare specifier such as:

```js
import React from "react";
```

normally requires tooling/import maps/runtime support.

Frameworks and bundlers resolve package imports for you.

---

## 19. Dynamic import()

Dynamic import returns a Promise.

```js
const module =
  await import(
    "./heavy-module.js"
  );
```

Use:

```js
module.someFunction();
```

Useful for:

- code splitting
- lazy loading
- conditional loading

---

## 20. Static vs Dynamic Import

Static:

```js
import {
  add,
} from "./math.js";
```

Resolved as part of module loading.

Dynamic:

```js
const module =
  await import(
    "./math.js"
  );
```

Loaded at runtime and returns a Promise.

---

## 21. Circular Dependencies

Module A imports B.

Module B imports A.

This is called a circular dependency.

ESM can support cycles because imports are live bindings, but initialization order can become confusing.

Best practice:

- avoid unnecessary cycles
- extract shared dependencies
- keep module boundaries clear

---

## 22. Tree Shaking

Because ESM imports/exports are statically analyzable, bundlers can often remove unused exports.

Example:

```js
import {
  add,
} from "./math.js";
```

If `subtract` is never imported/used, a production bundler may exclude it.

Actual tree-shaking effectiveness depends on tooling and side effects.

---

## 23. React / Next.js Connection

React:

```js
import {
  useState,
  useEffect,
} from "react";
```

Next.js:

```js
import Link
  from "next/link";
```

Your application is already built from modules.

Understanding ESM explains:

- file boundaries
- imports
- named/default exports
- tree shaking
- dynamic imports

---

## 24. Common Mistakes

### Mistake 1

Confusing default and named import syntax.

### Mistake 2

Trying to reassign an imported binding.

### Mistake 3

Assuming imports are copied values.

### Mistake 4

Creating circular dependencies without realizing initialization order matters.

---

## Interview Questions

### Named export vs default export?

Named exports use fixed export names and curly-brace imports. A module can have one default export, and the importer chooses its local name.

### Are ES module imports copies?

No. They are live bindings.

### Is ESM strict mode?

Yes.

### What does dynamic import return?

A Promise resolving to a module namespace object.

---

## Key Takeaways

- ESM is JavaScript's standard module system.
- Use `export` to expose module APIs.
- Use `import` to consume dependencies.
- Named and default exports behave differently.
- Imports are live bindings.
- Imported bindings cannot be reassigned locally.
- ES modules are strict mode.
- Dynamic `import()` enables runtime loading.
- Static structure helps bundlers optimize code.
