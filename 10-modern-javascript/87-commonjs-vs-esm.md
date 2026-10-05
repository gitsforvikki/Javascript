# Lesson 87 — CommonJS vs ES Modules

JavaScript has two major module systems you will encounter:

- CommonJS
- ES Modules

CommonJS became popular through Node.js.

ES Modules are the standard JavaScript module system.

---

## 1. CommonJS Basic Syntax

Export:

```js
// math.js

function add(a, b) {
  return a + b;
}

module.exports = {
  add,
};
```

Import:

```js
const {
  add,
} = require(
  "./math"
);
```

Core keywords:

```text
require
module.exports
exports
```

---

## 2. ESM Syntax

Export:

```js
export function add(
  a,
  b
) {
  return a + b;
}
```

Import:

```js
import {
  add,
} from "./math.js";
```

Core keywords:

```text
import
export
```

---

## 3. Main Syntax Comparison

```text
CommonJS
require()
module.exports

ESM
import
export
```

---

## 4. CommonJS Loading Model

`require()` is historically synchronous in Node.js.

Example:

```js
const config =
  require(
    "./config"
  );
```

Node loads/evaluates the module and returns its exported value.

---

## 5. ESM Loading Model

ESM has a statically analyzable dependency structure.

Example:

```js
import config
  from "./config.js";
```

The runtime can analyze module dependencies before evaluation.

This supports modern tooling and optimization.

---

## 6. Static vs Dynamic Nature

CommonJS:

```js
if (condition) {
  const mod =
    require(
      "./feature"
    );
}
```

Historically this kind of conditional loading is straightforward.

ESM static imports:

```js
import feature
  from "./feature.js";
```

must appear at module top level.

For dynamic ESM loading:

```js
const mod =
  await import(
    "./feature.js"
  );
```

---

## 7. Export Semantics

CommonJS exports an object/value.

```js
module.exports = {
  count,
};
```

ESM exports bindings.

```js
export let count = 0;
```

ESM imports are live bindings.

This semantic difference matters.

---

## 8. CommonJS Exports Can Behave Like Snapshots

Example:

```js
let count = 0;

module.exports = {
  count,
};
```

If internal `count` changes later, the previously assigned primitive property does not automatically behave like an ESM live binding.

CommonJS can expose live behavior through functions/objects, but the model is different.

---

## 9. this at Top Level

CommonJS modules historically have module-wrapper behavior in Node.

ES modules use true module scope, and top-level:

```js
this
```

is:

```text
undefined
```

Do not assume top-level `this` is the same in both systems.

---

## 10. File Recognition in Node.js

Node.js can determine module type through configuration such as:

```json
{
  "type": "module"
}
```

in `package.json`.

Typical extensions also matter:

```text
.mjs → ESM
.cjs → CommonJS
```

The exact behavior depends on Node/package configuration.

---

## 11. Importing Packages

CommonJS:

```js
const express =
  require("express");
```

ESM:

```js
import express
  from "express";
```

Modern packages may support:

- ESM only
- CommonJS only
- both

Always check package documentation when interoperability matters.

---

## 12. Interoperability Is Nuanced

Mixing module systems can work, but behavior depends on:

- runtime
- package format
- package exports
- bundler
- transpiler

Avoid simplistic assumptions such as:

```text
require and import are always interchangeable
```

They are not.

---

## 13. ESM and Tree Shaking

ESM's static structure makes tree shaking easier for bundlers.

CommonJS is harder to analyze statically because exports/imports can be more dynamic.

That does not mean CommonJS can never be optimized, but ESM is a better fit for modern bundling.

---

## 14. Browser Support

Browsers support ES Modules natively.

```html
<script
  type="module"
  src="./main.js"
></script>
```

Browsers do not natively provide Node-style `require()`.

Bundlers can transform CommonJS for browser usage.

---

## 15. Modern Direction

For new JavaScript/TypeScript projects, ESM is generally the modern standard.

CommonJS remains important because:

- many Node.js projects use it
- many packages still expose it
- legacy code uses it heavily

You should understand both.

---

## 16. Next.js / React Context

Modern Next.js and frontend tooling are ESM-oriented.

Your application code commonly uses:

```js
import React from "react";

export default function Page() {
  return <div />;
}
```

Backend tooling or older config files may still use CommonJS syntax.

---

## 17. Quick Comparison Table

| Feature | CommonJS | ESM |
| --- | --- | --- |
| Import | `require()` | `import` |
| Export | `module.exports` | `export` |
| Standard JS module system | No | Yes |
| Native browser support | No | Yes |
| Static analysis | Harder | Strong |
| Live bindings | Not the same model | Yes |
| Dynamic loading | `require()` patterns | `import()` |

---

## 18. Migration Example

CommonJS:

```js
const {
  add,
} = require(
  "./math"
);

module.exports =
  function calculate() {
    return add(1, 2);
  };
```

ESM:

```js
import {
  add,
} from "./math.js";

export default function calculate() {
  return add(1, 2);
}
```

---

## 19. Common Mistakes

### Mistake 1

Mixing `require` and `import` without checking runtime configuration.

### Mistake 2

Assuming CommonJS export semantics are identical to ESM live bindings.

### Mistake 3

Forgetting file/module type configuration in Node.

### Mistake 4

Thinking browsers support Node's `require()` natively.

---

## Interview Questions

### CommonJS vs ESM?

CommonJS uses `require()` and `module.exports`; ESM uses `import` and `export` and is the standard JavaScript module system.

### Why is ESM easier to tree-shake?

Its static import/export structure is easier for tooling to analyze.

### Can browsers use CommonJS directly?

Not natively in the Node.js sense.

---

## Key Takeaways

- CommonJS and ESM are different module systems.
- CommonJS is historically associated with Node.js.
- ESM is the modern standard.
- ESM imports are statically analyzable and live bindings.
- CommonJS loading/export behavior differs.
- Browser JavaScript natively supports ESM, not Node-style `require()`.
- Interoperability depends on runtime/tooling.
