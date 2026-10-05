# Lesson 121 — `eval` and Dynamic Code Risks

JavaScript provides APIs that can execute code created from strings.

The most famous is:

```js
eval(
  codeString
);
```

The core rule is:

> Never treat untrusted data as executable JavaScript.

---

## 1. What eval Does

```js
eval(
  "2 + 2"
);
```

returns:

```text
4
```

The string is parsed and executed as JavaScript.

That means the string has crossed from:

```text
data
```

into:

```text
code
```

---

## 2. Why This Is Dangerous

If any attacker-controlled value reaches `eval`, it may become executable code.

Mental model:

```text
untrusted input
↓
string concatenation
↓
eval()
↓
arbitrary JavaScript execution
```

That is a critical security boundary failure.

---

## 3. Do Not Build Code from User Input

Bad architecture:

```js
const expression =
  userInput;

eval(
  expression
);
```

If users need configurable behavior, build a controlled parser or command system instead of executing arbitrary JavaScript.

---

## 4. new Function()

Another dynamic code API:

```js
const fn =
  new Function(
    "a",
    "b",
    "return a + b"
  );
```

This also compiles strings as JavaScript.

Treat it with the same caution as `eval`.

---

## 5. setTimeout String Form

Legacy browsers allow patterns like:

```js
setTimeout(
  "doSomething()",
  1000
);
```

Avoid this.

Use a real function:

```js
setTimeout(
  doSomething,
  1000
);
```

String callbacks act like dynamic code execution.

---

## 6. Why eval Hurts Maintainability Too

Even with trusted strings, `eval` makes code harder to:

- analyze
- refactor
- type-check
- bundle
- optimize
- secure

Dynamic code hides dependencies from tooling.

---

## 7. Scope Behavior

Direct `eval` can interact with local scope in complicated ways.

That creates confusing behavior and makes optimization harder.

Avoid relying on it.

---

## 8. CSP Connection

A strict Content Security Policy can block dynamic code execution.

CSP directives can restrict:

- inline scripts
- `eval`
- dynamic function construction

Allowing:

```text
'unsafe-eval'
```

weakens CSP protections.

---

## 9. JSON Is Not JavaScript

To parse JSON:

Bad:

```js
const data =
  eval(
    "(" +
      json +
      ")"
  );
```

Correct:

```js
const data =
  JSON.parse(
    json
  );
```

Use data parsers for data formats.

---

## 10. Dynamic Property Access Does Not Need eval

Bad:

```js
eval(
  "user." +
    property
);
```

Correct:

```js
user[property];
```

Bracket notation already supports dynamic property names safely.

---

## 11. Dynamic Function Selection

Bad:

```js
eval(
  command + "()"
);
```

Better:

```js
const commands = {
  save,
  delete:
    deleteItem,
};

const fn =
  commands[command];

if (fn) {
  fn();
}
```

This creates an explicit allowlist.

---

## 12. Mathematical Expressions

If users must enter formulas, do not execute them as JavaScript.

Use:

- a dedicated expression parser
- a restricted grammar
- a safe evaluation library

The feature should support only the operations you intentionally allow.

---

## 13. Template Engines

Avoid template systems that execute arbitrary code from user-controlled templates.

Prefer systems that separate:

- data
- presentation
- allowed helpers

---

## 14. Server-Side eval Is Also Dangerous

Node.js code can be vulnerable too.

Executing user-controlled code on the server can expose:

- filesystem
- network
- environment variables
- process secrets
- databases

Server-side dynamic execution can be even more severe.

---

## 15. vm Is Not a Magic Sandbox

Node's VM-related features require deep security care.

Do not assume:

```text
"run in VM"
=
"safe untrusted sandbox"
```

Secure sandboxing is a specialized problem.

---

## 16. Deserialization Connection

Some insecure systems turn serialized input into executable objects/functions.

The safer rule is:

> Parse data as data, and validate its structure.

Avoid formats that reconstruct arbitrary executable behavior from untrusted input.

---

## 17. Dynamic import Is Different

```js
await import(
  "./feature.js"
);
```

is module loading, not the same as evaluating an arbitrary string of JavaScript source.

Still, do not let attackers freely choose arbitrary module paths.

Validate dynamic module selection.

---

## 18. Safe Registry Pattern

```js
const modules = {
  reports:
    () =>
      import(
        "./reports.js"
      ),

  profile:
    () =>
      import(
        "./profile.js"
      ),
};
```

Then:

```js
const loader =
  modules[name];

if (!loader) {
  throw new Error(
    "Unknown module"
  );
}

await loader();
```

Explicit allowlists are safer.

---

## 19. Common Mistakes

### Mistake 1

Using eval to parse JSON.

### Mistake 2

Using eval for dynamic object properties.

### Mistake 3

Thinking new Function is safer than eval.

### Mistake 4

Allowing unsafe-eval in CSP without a strong reason.

### Mistake 5

Building "sandboxed code execution" casually.

---

## Interview Questions

### Why is eval dangerous?

Because strings become executable JavaScript, so untrusted input can lead to arbitrary code execution.

### Alternatives to eval?

Use explicit data structures, parsers, registries, bracket notation, JSON.parse, and safe interpreters.

### Is new Function safe?

It has similar dynamic-code risks and should not receive untrusted input.

---

## Key Takeaways

- Keep data and code separate.
- Avoid `eval`, `new Function`, and string-based timer callbacks.
- Use explicit allowlists and parsers.
- Dynamic code weakens tooling and CSP.
- Never execute untrusted JavaScript casually in browser or server environments.
