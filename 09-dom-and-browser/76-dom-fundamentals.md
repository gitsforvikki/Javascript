# Lesson 76 — DOM Fundamentals

The DOM is the bridge between JavaScript and the HTML document shown in the browser.

DOM stands for:

> **Document Object Model**

The browser parses HTML and creates an in-memory tree of objects that JavaScript can read and modify.

This lesson focuses on the mental model. Later lessons will cover selecting, changing, creating, removing, and reacting to DOM elements.

---

## 1. HTML Is Not the DOM

Suppose the browser receives:

```html
<!DOCTYPE html>
<html>
  <body>
    <h1>Hello</h1>
    <p>Welcome</p>
  </body>
</html>
```

The browser parses this markup and creates a DOM tree.

Conceptually:

```text
Document
└── html
    └── body
        ├── h1
        │   └── "Hello"
        └── p
            └── "Welcome"
```

HTML is source markup.

The DOM is the browser's object representation of that document.

---

## 2. The document Object

In browser JavaScript, the current page is represented by the global `document` object.

Example:

```js
console.log(document);
```

The `document` object provides APIs to:

- find elements
- create elements
- modify content
- attach events
- inspect structure

Example:

```js
document.title;
```

returns the current document title.

---

## 3. DOM Nodes

Everything in the DOM tree is represented as a node.

Common node types include:

- Document
- Element
- Text
- Comment

Example:

```html
<h1>Hello</h1>
```

Conceptually contains:

```text
Element node: <h1>
└── Text node: "Hello"
```

The text inside an element is its own node.

---

## 4. Element Nodes

HTML tags become element nodes.

Example:

```html
<button>Save</button>
```

The browser creates a button element object.

JavaScript can later access properties such as:

```js
button.textContent;
button.id;
button.className;
button.style;
```

---

## 5. Parent, Child, and Sibling Relationships

DOM nodes form tree relationships.

Example:

```html
<ul>
  <li>JavaScript</li>
  <li>React</li>
</ul>
```

Mental model:

```text
ul
├── li
│   └── "JavaScript"
└── li
    └── "React"
```

The `ul` is the parent.

The `li` elements are children.

The two `li` elements are siblings.

---

## 6. Node Navigation

Useful node-based properties include:

```js
node.parentNode
node.childNodes
node.firstChild
node.lastChild
node.nextSibling
node.previousSibling
```

Important:

`childNodes` includes text nodes such as whitespace.

That can surprise beginners.

---

## 7. Element Navigation

When you only care about element nodes, use element-focused properties:

```js
element.parentElement
element.children
element.firstElementChild
element.lastElementChild
element.nextElementSibling
element.previousElementSibling
```

These usually make UI code easier to reason about.

---

## 8. childNodes vs children

Given:

```html
<ul>
  <li>JS</li>
  <li>React</li>
</ul>
```

`children` returns only element children.

`childNodes` may include:

- text nodes from whitespace
- element nodes

So:

```text
children
→ elements only

childNodes
→ all node types
```

This is a common interview distinction.

---

## 9. documentElement, head, and body

Useful top-level references:

```js
document.documentElement
document.head
document.body
```

Typically:

```text
document.documentElement
→ <html>

document.head
→ <head>

document.body
→ <body>
```

---

## 10. DOM Is Live Application State

After the browser creates the DOM, JavaScript can change it.

Example:

```js
document.body.textContent =
  "Updated";
```

The visible page changes because the DOM changed.

This means:

```text
JavaScript
↓
updates DOM
↓
browser reacts
↓
screen may re-render
```

---

## 11. DOM vs JavaScript Objects

DOM objects are objects exposed by the browser.

Example:

```js
const heading =
  document.querySelector("h1");

console.log(
  typeof heading
);
```

Output:

```text
object
```

But DOM objects are special host objects with browser-provided behavior.

They are not ordinary plain objects like:

```js
const user = {
  name: "Vikash",
};
```

---

## 12. DOM API Comes from the Browser

The DOM is not part of the core ECMAScript language.

That is why this works in a browser:

```js
document.querySelector(...)
```

but ordinary Node.js does not provide `document`.

This connects to your runtime lesson:

```text
JavaScript language
+
browser runtime APIs
=
browser JavaScript
```

---

## 13. DOM Tree and Rendering Tree Are Not the Same Thing

The DOM represents document structure.

The browser also builds other internal structures for rendering.

At a high level:

```text
HTML
↓
DOM

CSS
↓
style information

DOM + styles
↓
rendering-related structures
↓
layout
↓
paint
```

Do not assume the DOM tree itself is exactly what gets painted.

---

## 14. DOM Changes Can Trigger Browser Work

Changing the DOM can cause:

- style recalculation
- layout
- paint
- compositing

Example:

```js
element.style.width =
  "500px";
```

Some changes are more expensive than others.

Performance implications will be studied later.

---

## 15. DOMContentLoaded

The browser parses HTML progressively.

A common event is:

```js
document.addEventListener(
  "DOMContentLoaded",
  () => {
    console.log(
      "DOM ready"
    );
  }
);
```

This event fires when the HTML document has been parsed and the DOM is ready, without necessarily waiting for every external resource such as images to finish loading.

---

## 16. Script Placement Matters

If JavaScript runs before an element exists in the DOM:

```js
document.querySelector(
  "#save"
);
```

may return:

```text
null
```

Common solutions:

- place script near end of body
- use `defer`
- wait for DOM readiness

Modern module scripts are deferred by default in normal browser loading behavior.

---

## 17. defer vs async — High-Level Preview

For classic external scripts:

```html
<script
  src="app.js"
  defer
></script>
```

`defer` downloads in parallel but waits to execute until HTML parsing is complete.

```html
<script
  src="app.js"
  async
></script>
```

`async` executes when ready and can interrupt parsing.

For DOM-dependent application scripts, `defer` is often easier to reason about.

---

## 18. DOM Example

HTML:

```html
<div id="app">
  <h1>CareerLoop</h1>
  <p>Track applications</p>
</div>
```

Conceptual DOM:

```text
Document
└── html
    └── body
        └── div#app
            ├── h1
            │   └── CareerLoop
            └── p
                └── Track applications
```

JavaScript can locate and modify any of these nodes.

---

## 19. React Connection

React also updates the browser DOM, but you usually do not manipulate it manually.

Instead:

```jsx
function App() {
  return (
    <h1>
      CareerLoop
    </h1>
  );
}
```

React calculates what DOM updates are needed.

You still need DOM knowledge because:

- event targets are DOM elements
- refs expose DOM nodes
- browser APIs operate on DOM nodes
- accessibility depends on real HTML structure
- debugging often happens in browser DevTools

---

## 20. Common Mistakes

### Mistake 1

Thinking HTML and DOM are exactly the same thing.

### Mistake 2

Assuming every child node is an element.

### Mistake 3

Using DOM APIs in server-side code where `document` does not exist.

### Mistake 4

Trying to select an element before it has been parsed.

---

## Interview Questions

### What is the DOM?

The DOM is the browser's object-based tree representation of a document, allowing JavaScript to inspect and manipulate page structure and content.

### Is the DOM part of JavaScript?

No. It is provided by browser environments.

### What is the difference between children and childNodes?

`children` contains element children only, while `childNodes` can include text, comment, and other node types.

---

## Key Takeaways

- DOM means Document Object Model.
- HTML is parsed into a DOM tree.
- DOM entries are nodes.
- Elements, text, and comments are different node types.
- Parent/child/sibling relationships form the tree.
- `document` is the browser entry point to the page.
- DOM APIs come from the browser, not core JavaScript.
- DOM changes can affect browser rendering.
- React ultimately renders to real DOM elements.
