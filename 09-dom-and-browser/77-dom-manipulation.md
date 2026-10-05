# Lesson 77 — Selecting and Manipulating DOM Elements

Before changing the DOM, you usually need to select an element.

This lesson focuses on:

- finding elements
- reading content
- changing content
- attributes
- classes
- styles

Creating and removing elements is reserved for Lesson 78.

---

## 1. getElementById()

HTML:

```html
<h1 id="title">
  CareerLoop
</h1>
```

JavaScript:

```js
const title =
  document.getElementById(
    "title"
  );
```

If the element does not exist:

```text
null
```

---

## 2. querySelector()

`querySelector()` uses CSS selector syntax.

```js
const title =
  document.querySelector(
    "#title"
  );
```

Examples:

```js
document.querySelector(
  ".card"
);

document.querySelector(
  "button"
);

document.querySelector(
  '[data-id="10"]'
);
```

It returns the first matching element.

---

## 3. querySelectorAll()

```js
const cards =
  document.querySelectorAll(
    ".card"
  );
```

It returns a static `NodeList` of all matches.

You can iterate:

```js
cards.forEach((card) => {
  console.log(card);
});
```

---

## 4. getElementsByClassName()

```js
const cards =
  document
    .getElementsByClassName(
      "card"
    );
```

This returns an `HTMLCollection`.

Important difference:

```text
querySelectorAll
→ static NodeList

getElementsByClassName
→ live HTMLCollection
```

A live collection updates automatically as matching DOM elements change.

---

## 5. getElementsByTagName()

```js
const buttons =
  document
    .getElementsByTagName(
      "button"
    );
```

This also returns a live `HTMLCollection`.

---

## 6. querySelector vs getElementById

```js
document.getElementById(
  "title"
);
```

is specialized for IDs.

```js
document.querySelector(
  "#title"
);
```

is more flexible because it accepts CSS selectors.

In modern code, `querySelector` and `querySelectorAll` are extremely common.

---

## 7. Selecting Within an Element

Selection does not always need to start from `document`.

```js
const card =
  document.querySelector(
    ".card"
  );

const button =
  card.querySelector(
    "button"
  );
```

This searches only inside the selected card.

Useful for component-like DOM structures.

---

## 8. textContent

HTML:

```html
<h1 id="title">
  Old Title
</h1>
```

JavaScript:

```js
const title =
  document.querySelector(
    "#title"
  );

title.textContent =
  "New Title";
```

This changes text content.

---

## 9. innerText

```js
element.innerText
```

is influenced by rendered layout and visibility.

`textContent` reflects text content more directly from the DOM tree.

For many manipulation tasks:

```js
textContent
```

is preferred because it is predictable and does not depend on layout calculation in the same way.

---

## 10. innerHTML

```js
element.innerHTML =
  "<strong>Hello</strong>";
```

This parses the string as HTML.

Result:

```html
<strong>Hello</strong>
```

Be careful:

> Never insert untrusted user input through `innerHTML` without safe sanitization.

It can create XSS vulnerabilities.

Security will be covered later.

---

## 11. textContent vs innerHTML

```js
element.textContent =
  "<strong>Hello</strong>";
```

displays the literal text:

```text
<strong>Hello</strong>
```

But:

```js
element.innerHTML =
  "<strong>Hello</strong>";
```

creates a real `strong` element.

---

## 12. Reading Attributes

HTML:

```html
<a
  id="docs"
  href="/docs"
  target="_blank"
>
  Docs
</a>
```

Read:

```js
const link =
  document.querySelector(
    "#docs"
  );

link.getAttribute(
  "href"
);
```

---

## 13. Setting Attributes

```js
link.setAttribute(
  "href",
  "/guide"
);
```

Check:

```js
link.hasAttribute(
  "target"
);
```

Remove:

```js
link.removeAttribute(
  "target"
);
```

---

## 14. Properties vs Attributes

This distinction matters.

HTML attributes initialize element state, but DOM properties can represent the current live state.

For example, an input can have:

```html
<input value="hello" />
```

Then JavaScript may change:

```js
input.value =
  "updated";
```

The live `value` property can differ from the original `value` attribute.

So:

```text
attribute
→ markup-related value

property
→ current DOM object state
```

The exact relationship varies by property.

---

## 15. className

```js
element.className =
  "card active";
```

This replaces the whole class string.

That can be useful, but it is easy to accidentally remove existing classes.

---

## 16. classList

A safer API:

```js
element.classList.add(
  "active"
);

element.classList.remove(
  "hidden"
);

element.classList.toggle(
  "selected"
);

element.classList.contains(
  "active"
);
```

This is usually easier than manually editing `className`.

---

## 17. toggle with a Boolean

```js
element.classList.toggle(
  "active",
  isActive
);
```

If `isActive` is true, the class is present.

If false, it is removed.

This is useful for state-based UI logic.

---

## 18. Inline Styles

```js
element.style.color =
  "red";

element.style.backgroundColor =
  "black";
```

CSS property names become camelCase.

Example:

```text
background-color
→ backgroundColor
```

---

## 19. Avoid Excessive Inline Styling

For larger UI changes, prefer classes.

Instead of:

```js
element.style.color =
  "white";

element.style.background =
  "green";
```

prefer:

```js
element.classList.add(
  "success"
);
```

This keeps presentation logic in CSS.

---

## 20. data-* Attributes

HTML:

```html
<button
  data-job-id="123"
>
  View
</button>
```

Access through:

```js
button.dataset.jobId;
```

Output:

```text
"123"
```

Important:

Dataset values are strings.

---

## 21. Updating data Attributes

```js
button.dataset.jobId =
  "456";
```

This updates:

```html
data-job-id="456"
```

Very useful with event delegation.

---

## 22. Form Element Values

Input:

```js
const input =
  document.querySelector(
    "#name"
  );

console.log(
  input.value
);
```

Checkbox:

```js
checkbox.checked;
```

Select:

```js
select.value;
```

These are DOM properties representing current UI state.

---

## 23. Disabled State

```js
button.disabled = true;
```

This is better than treating boolean attributes like ordinary strings.

Boolean DOM properties often communicate current state clearly.

---

## 24. Safe Selection

A selector may return `null`.

Bad:

```js
document
  .querySelector(
    "#missing"
  )
  .textContent = "Hi";
```

Safer:

```js
const element =
  document.querySelector(
    "#missing"
  );

if (element) {
  element.textContent =
    "Hi";
}
```

---

## 25. Practical Example

HTML:

```html
<div
  class="job-card"
  data-job-id="42"
>
  <h2 class="title">
    Frontend Developer
  </h2>

  <button class="save">
    Save
  </button>
</div>
```

JavaScript:

```js
const card =
  document.querySelector(
    ".job-card"
  );

const title =
  card.querySelector(
    ".title"
  );

const button =
  card.querySelector(
    ".save"
  );

title.textContent =
  "React Developer";

card.classList.add(
  "highlighted"
);

button.disabled = true;
```

---

## 26. React Connection

React discourages most direct DOM manipulation.

Instead of:

```js
element.textContent =
  "Saved";
```

React prefers:

```jsx
<button>
  {saved
    ? "Saved"
    : "Save"}
</button>
```

But direct DOM access is still needed for:

- focus management
- measuring elements
- third-party libraries
- canvas/video APIs
- imperative browser APIs

Usually React refs are used for those cases.

---

## 27. Interview Questions

### querySelector vs querySelectorAll?

`querySelector` returns the first match; `querySelectorAll` returns all matches as a static NodeList.

### innerHTML vs textContent?

`innerHTML` parses markup; `textContent` treats content as text and is safer for untrusted strings.

### className vs classList?

`className` replaces the complete class string; `classList` provides add/remove/toggle/contains operations.

---

## Key Takeaways

- Use selectors to find DOM elements.
- `querySelector` accepts CSS selectors.
- `querySelectorAll` returns a static NodeList.
- Some older collection APIs return live HTMLCollections.
- `textContent` is safer than `innerHTML` for plain text.
- Attributes and DOM properties are related but not identical.
- `classList` is ideal for class manipulation.
- `dataset` exposes `data-*` attributes.
- Always handle the possibility of `null` selectors.
