# Lesson 120 — `innerHTML` and Unsafe DOM APIs

Some DOM APIs interpret strings as HTML or executable content.

These APIs are often called **dangerous sinks** because untrusted data can become active browser content.

The most famous example is:

```js
element.innerHTML =
  value;
```

---

## 1. Why innerHTML Is Risky

`innerHTML` parses a string as HTML.

```js
element.innerHTML =
  "<strong>Hello</strong>";
```

creates a real `strong` element.

That is useful for trusted markup.

But if the string is untrusted, the browser may interpret dangerous content too.

---

## 2. Safe Alternative for Text

If you only need text:

```js
element.textContent =
  value;
```

This does not parse the value as HTML.

Rule:

> Use `textContent` unless you intentionally need HTML.

---

## 3. innerText vs textContent

Both render text-like content, but `textContent` is usually preferable for security-focused insertion because it directly sets text content without HTML parsing.

`innerText` also depends more on rendered layout/visibility behavior.

---

## 4. insertAdjacentHTML

This API also parses HTML:

```js
element.insertAdjacentHTML(
  "beforeend",
  html
);
```

So it has the same general trust problem as `innerHTML`.

---

## 5. outerHTML

```js
element.outerHTML =
  html;
```

also parses HTML and replaces the element itself.

Treat untrusted input here as dangerous.

---

## 6. document.write

Legacy code may use:

```js
document.write(
  value
);
```

This can inject parsed HTML directly into the document.

It is rarely appropriate in modern application code and can create serious security and lifecycle problems.

---

## 7. DOMParser

```js
const parser =
  new DOMParser();

const doc =
  parser.parseFromString(
    html,
    "text/html"
  );
```

Parsing HTML does not automatically mean the content is safe.

If parsed nodes later enter the live DOM, unsafe markup can become dangerous.

---

## 8. createElement Is Safer for Structured DOM

Instead of:

```js
container.innerHTML =
  `
    <button>
      ${label}
    </button>
  `;
```

prefer:

```js
const button =
  document.createElement(
    "button"
  );

button.textContent =
  label;

container.append(
  button
);
```

This keeps data and markup separated.

---

## 9. Attribute Safety

This is better than building raw HTML strings:

```js
const link =
  document.createElement(
    "a"
  );

link.textContent =
  label;

link.href =
  validatedUrl;
```

But note:

> Setting a property is not automatically safe if the value itself has dangerous semantics.

URL values still require validation.

---

## 10. URL Schemes

When accepting a URL, validate allowed protocols.

Typical allowed web navigation:

```text
https:
http:
```

Depending on your application, you may also allow specific others.

Use the `URL` constructor where appropriate rather than fragile string checks.

---

## 11. Event Handler Attributes

Avoid dynamically constructing HTML such as:

```html
<button onclick="...">
```

Inline event-handler attributes are executable code contexts.

Prefer:

```js
button.addEventListener(
  "click",
  handler
);
```

---

## 12. setAttribute Is Context-Dependent

This is not universally safe:

```js
element.setAttribute(
  name,
  value
);
```

If `name` is attacker-controlled, they may choose a dangerous attribute.

Prefer fixed attribute names and validate values.

---

## 13. style Injection

Avoid building arbitrary style strings from untrusted input.

Example:

```js
element.style.cssText =
  userValue;
```

Even when script execution is restricted, arbitrary CSS can still affect UI integrity and data exposure in surprising ways.

Use controlled properties instead.

---

## 14. Safe DOM Construction Pattern

```js
function createCard(
  user
) {
  const card =
    document.createElement(
      "article"
    );

  const title =
    document.createElement(
      "h2"
    );

  title.textContent =
    user.name;

  card.append(
    title
  );

  return card;
}
```

Untrusted text remains text.

---

## 15. Sanitizing Required HTML

Sometimes your product intentionally supports formatted user content.

Then:

```text
untrusted HTML
↓
trusted sanitizer
↓
sanitized HTML
↓
dangerous sink
```

Do not skip the sanitizer step.

---

## 16. Why Regex Is Not Enough

HTML is a real parser-driven language with:

- nested markup
- encoded characters
- malformed-but-recoverable syntax
- SVG/MathML integration
- many attributes
- URL contexts

Regex-based "remove script tags" logic is not a security boundary.

---

## 17. React Connection

Safe:

```jsx
<div>
  {userContent}
</div>
```

Unsafe escape hatch:

```jsx
<div
  dangerouslySetInnerHTML={{
    __html:
      userContent,
  }}
/>
```

Only use the second pattern with trusted/sanitized HTML.

---

## 18. Third-Party Widgets

Libraries may internally call dangerous sinks.

If a library accepts raw HTML strings, treat its input as security-sensitive.

Review library documentation and keep dependencies updated.

---

## 19. Trusted Types Connection

Trusted Types can restrict assignments to certain DOM sinks such as `innerHTML`.

A policy can require that HTML values pass through approved creation/sanitization logic first.

This reduces accidental DOM XSS in large applications.

---

## 20. DOM API Safety Mental Model

```text
Does API treat input as text?
↓
lower XSS risk

Does API parse HTML/code?
↓
high-trust boundary
```

---

## 21. Common Dangerous Sinks

Examples include:

```text
innerHTML
outerHTML
insertAdjacentHTML
document.write
eval
new Function
some URL/script contexts
```

Each sink has different semantics.

---

## 22. Common Mistakes

### Mistake 1

Using innerHTML just to display text.

### Mistake 2

Assuming createElement makes every property value safe.

### Mistake 3

Using regex sanitization.

### Mistake 4

Passing attacker-controlled attribute names.

### Mistake 5

Forgetting third-party libraries may introduce sinks.

---

## Interview Questions

### Why is innerHTML dangerous?

Because it parses strings as markup, so untrusted input can become executable active content.

### Safer alternative for plain text?

`textContent`.

### Is insertAdjacentHTML safe with user input?

Not without trusted sanitization.

---

## Key Takeaways

- HTML-parsing APIs are security-sensitive sinks.
- Prefer DOM construction and `textContent`.
- Validate URLs and fixed attributes.
- Use a real sanitizer for required untrusted HTML.
- Avoid inline event-handler strings.
- Trusted Types can add enforcement for dangerous DOM sinks.
