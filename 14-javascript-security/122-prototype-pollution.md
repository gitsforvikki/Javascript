# Lesson 122 — Prototype Pollution Basics

Prototype pollution is a class of vulnerability where attacker-controlled input modifies properties on shared object prototypes.

The core danger is:

> Data intended for one object can unexpectedly change behavior for many objects through the prototype chain.

---

## 1. Prototype Chain Refresher

Consider:

```js
const user = {
  name: "Vikash",
};

console.log(
  user.toString
);
```

`toString` is usually inherited through the prototype chain.

Conceptually:

```text
user
↓
Object.prototype
↓
null
```

---

## 2. Why Prototype Pollution Is Dangerous

If a shared prototype is modified:

```js
Object.prototype.isAdmin =
  true;
```

then unrelated objects may appear to have:

```js
object.isAdmin
```

even when they never defined it themselves.

This can corrupt assumptions across an application.

---

## 3. The Pollution Mental Model

```text
untrusted key path
↓
unsafe object merge/setter
↓
prototype object modified
↓
many objects inherit polluted property
```

---

## 4. Dangerous Property Names

Security-sensitive keys often include:

```text
__proto__
constructor
prototype
```

Code that recursively merges arbitrary object keys must treat these paths carefully.

---

## 5. Unsafe Merge Pattern

A risky generic merge might do:

```js
function merge(
  target,
  source
) {
  for (
    const key
    in source
  ) {
    if (
      typeof source[key] ===
        "object" &&
      source[key] !== null
    ) {
      target[key] ??= {};

      merge(
        target[key],
        source[key]
      );
    } else {
      target[key] =
        source[key];
    }
  }

  return target;
}
```

The problem is not recursion itself.

The problem is trusting arbitrary property paths.

---

## 6. Why Object Spread Is Usually Safer Than Deep Merge

For a flat object:

```js
const result = {
  ...input,
};
```

this creates own properties on the new object.

But do not generalize this to:

> Object spread prevents all prototype-related security problems.

Deep merge utilities and dynamic setters can still introduce risk.

---

## 7. Object.create(null)

For dictionary-style objects:

```js
const dictionary =
  Object.create(
    null
  );
```

This object has no `Object.prototype` in its prototype chain.

That reduces some prototype-name collision risks.

---

## 8. Map as a Safer Dictionary

When storing arbitrary user-defined keys:

```js
const values =
  new Map();
```

may be clearer than using a plain object as a key-value dictionary.

Benefits:

- no prototype-chain keys
- explicit collection API
- arbitrary key types

---

## 9. Own Property Checks

Do not rely on:

```js
if (object.isAdmin) {
}
```

when security requires the property to be explicitly present.

Use:

```js
Object.hasOwn(
  object,
  "isAdmin"
);
```

when own-property semantics are required.

---

## 10. for...in Caution

`for...in` enumerates enumerable string properties from the object and its prototype chain.

So:

```js
for (
  const key
  in object
) {
}
```

may include inherited properties.

For own enumerable entries, often prefer:

```js
Object.keys(
  object
);

Object.entries(
  object
);
```

---

## 11. Security Check Example

Risky:

```js
if (
  options.isAdmin
) {
  grantAccess();
}
```

Safer design:

- derive authorization server-side
- do not trust client-provided flags
- use own-property checks when parsing configuration

Prototype safety is not a replacement for proper authorization.

---

## 12. Dynamic Path Setters

Utilities that support paths such as:

```text
user.profile.name
```

must validate path segments.

Do not allow dangerous segments such as:

```text
__proto__
constructor
prototype
```

to traverse into shared prototypes.

---

## 13. Parsing JSON

`JSON.parse()` itself creates data from JSON.

The greater danger often appears later when parsed data is fed into an unsafe merge or path-setting utility.

Mental model:

```text
JSON input
↓
plain parsed data
↓
unsafe deep merge
↓
prototype pollution
```

---

## 14. Library Vulnerabilities

Prototype pollution has historically affected generic:

- merge utilities
- object path setters
- configuration libraries
- query parsers

Keep dependencies updated and review security advisories.

---

## 15. Validate Object Shape

Instead of accepting arbitrary nested keys:

```js
function normalizeSettings(
  input
) {
  return {
    theme:
      input.theme ===
      "dark"
        ? "dark"
        : "light",

    pageSize:
      Number(
        input.pageSize
      ) || 20,
  };
}
```

Explicitly constructing allowed fields is safer than merging arbitrary input.

---

## 16. Allowlist Keys

If dynamic keys are necessary:

```js
const allowed =
  new Set([
    "theme",
    "language",
    "pageSize",
  ]);
```

Then reject anything else.

This is much safer than trusting arbitrary object paths.

---

## 17. Freeze Is Not a Complete Defense

```js
Object.freeze(
  Object.prototype
);
```

may break code and is not a universal practical fix.

Security should come from:

- input validation
- safe merge logic
- dependency hygiene
- authorization boundaries

---

## 18. Frontend vs Backend Impact

Prototype pollution can affect:

### Frontend

- configuration
- rendering decisions
- security-sensitive flags
- library behavior

### Backend

- authorization logic
- request handling
- configuration
- server-side behavior

Backend impact can be especially serious.

---

## 19. React / Next.js Connection

Do not merge arbitrary request/search/form data directly into trusted configuration.

Bad:

```js
const options =
  deepMerge(
    defaults,
    req.body
  );
```

Better:

```js
const options = {
  theme:
    validateTheme(
      req.body.theme
    ),

  pageSize:
    validatePageSize(
      req.body.pageSize
    ),
};
```

---

## 20. Common Mistakes

### Mistake 1

Treating any object key as harmless data.

### Mistake 2

Using deep-merge utilities on untrusted objects.

### Mistake 3

Using inherited properties in security decisions.

### Mistake 4

Assuming JSON.parse itself is the entire vulnerability.

### Mistake 5

Trusting client-side authorization flags.

---

## Interview Questions

### What is prototype pollution?

A vulnerability where attacker-controlled input modifies shared prototype properties and influences unrelated objects.

### Why are `__proto__`, `constructor`, and `prototype` sensitive?

They can provide paths toward prototype objects in unsafe merge/set logic.

### How can you reduce risk?

Validate keys, avoid unsafe deep merges, use Maps or null-prototype dictionaries where appropriate, and keep libraries updated.

---

## Key Takeaways

- Prototype pollution abuses the prototype chain.
- Arbitrary deep object merging is security-sensitive.
- Prefer explicit schemas/allowlists.
- Use `Object.hasOwn()` when own properties matter.
- Maps/null-prototype objects are useful for untrusted dictionary keys.
- Authorization must never rely on untrusted client object fields.
