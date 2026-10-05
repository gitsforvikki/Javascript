# Lesson 78 — Creating, Updating and Removing Elements

The DOM is dynamic.

JavaScript can:

- create elements
- insert them
- move them
- replace them
- remove them

This lesson focuses only on structural DOM changes.

---

## 1. document.createElement()

Create an element:

```js
const button =
  document.createElement(
    "button"
  );
```

At this point, the button exists in memory but is not visible on the page.

---

## 2. Configure the New Element

```js
button.textContent =
  "Save";

button.classList.add(
  "btn",
  "btn-primary"
);

button.type = "button";
```

Then insert it into the DOM.

---

## 3. append()

```js
const container =
  document.querySelector(
    "#actions"
  );

container.append(
  button
);
```

The button becomes a child of the container.

`append()` can accept:

- nodes
- strings
- multiple arguments

---

## 4. appendChild()

Older but still common:

```js
container.appendChild(
  button
);
```

Difference:

```text
append()
→ nodes + strings
→ multiple arguments
→ no useful return value

appendChild()
→ one Node
→ returns appended node
```

---

## 5. prepend()

```js
container.prepend(
  button
);
```

This inserts as the first child.

---

## 6. before() and after()

Given:

```js
const card =
  document.querySelector(
    ".card"
  );
```

Insert before:

```js
card.before(
  newElement
);
```

Insert after:

```js
card.after(
  newElement
);
```

These insert relative to the selected element.

---

## 7. Creating Text Nodes

```js
const text =
  document.createTextNode(
    "Hello"
  );
```

Then:

```js
element.append(text);
```

Often, assigning `textContent` is simpler.

---

## 8. Build a Complete Element

```js
const card =
  document.createElement(
    "article"
  );

card.className =
  "job-card";

const title =
  document.createElement(
    "h2"
  );

title.textContent =
  "Frontend Developer";

const company =
  document.createElement(
    "p"
  );

company.textContent =
  "Example Inc.";

card.append(
  title,
  company
);
```

Then:

```js
document.body.append(
  card
);
```

---

## 9. Moving Existing Elements

Important:

> Appending an element that already exists moves it.

Example:

```js
const button =
  document.querySelector(
    "#save"
  );

otherContainer.append(
  button
);
```

The browser does not duplicate the same DOM node.

It moves it to the new parent.

---

## 10. cloneNode()

To make a copy:

```js
const copy =
  element.cloneNode(
    true
  );
```

`true` means deep clone, including descendants.

```js
element.cloneNode(
  false
);
```

copies only the element itself.

Important:

Event listeners added with `addEventListener` are not generally copied by `cloneNode()`.

---

## 11. replaceWith()

```js
oldElement.replaceWith(
  newElement
);
```

The old node is replaced in the DOM.

---

## 12. replaceChildren()

```js
container.replaceChildren(
  newChild1,
  newChild2
);
```

This removes current children and inserts the supplied new children.

Useful for refreshing a section.

---

## 13. remove()

```js
element.remove();
```

The element is removed from its parent.

Simple and common.

---

## 14. removeChild()

Older API:

```js
parent.removeChild(
  child
);
```

This requires a parent reference.

Modern code often prefers:

```js
child.remove();
```

---

## 15. Clearing an Element

Option:

```js
container.replaceChildren();
```

This removes all children.

You may also see:

```js
container.innerHTML = "";
```

but `replaceChildren()` expresses the structural intent clearly.

---

## 16. insertAdjacentHTML()

```js
element.insertAdjacentHTML(
  "beforeend",
  "<p>Hello</p>"
);
```

Positions:

- `beforebegin`
- `afterbegin`
- `beforeend`
- `afterend`

Useful, but remember:

> Do not inject untrusted HTML strings.

---

## 17. insertAdjacentElement()

For existing nodes:

```js
element.insertAdjacentElement(
  "afterend",
  newElement
);
```

This avoids parsing an HTML string.

---

## 18. DocumentFragment

When building several nodes:

```js
const fragment =
  document.createDocumentFragment();
```

Add children:

```js
for (const job of jobs) {
  const item =
    document.createElement(
      "li"
    );

  item.textContent =
    job.title;

  fragment.append(
    item
  );
}
```

Then insert once:

```js
list.append(fragment);
```

The fragment itself disappears; its children are inserted.

---

## 19. Why DocumentFragment Can Help

It lets you assemble a group of nodes before attaching them to the live document.

This can make code cleaner and may reduce unnecessary intermediate DOM interaction.

Modern browsers are highly optimized, so use it when it improves structure—not as a magical performance rule.

---

## 20. Avoid Repeated Expensive Layout Reads

Bad pattern:

```js
for (...) {
  element.style.width =
    "...";

  console.log(
    element.offsetWidth
  );
}
```

Mixing writes and layout reads repeatedly can force extra browser work.

A better general strategy is:

```text
group reads
↓
group writes
```

You will study rendering performance later.

---

## 21. createElement vs innerHTML

Using `innerHTML`:

```js
container.innerHTML =
  `
    <button>
      Save
    </button>
  `;
```

Using DOM APIs:

```js
const button =
  document.createElement(
    "button"
  );

button.textContent =
  "Save";

container.append(
  button
);
```

DOM APIs are especially useful when:

- values are dynamic
- you want explicit structure
- you want to avoid HTML-string security risks

---

## 22. Rendering a List

```js
const jobs = [
  {
    id: 1,
    title: "React Developer",
  },
  {
    id: 2,
    title: "Next.js Developer",
  },
];

const list =
  document.querySelector(
    "#jobs"
  );

const fragment =
  document.createDocumentFragment();

for (const job of jobs) {
  const item =
    document.createElement(
      "li"
    );

  item.textContent =
    job.title;

  item.dataset.jobId =
    String(job.id);

  fragment.append(
    item
  );
}

list.append(fragment);
```

---

## 23. Updating Existing Elements vs Recreating

Sometimes you should update:

```js
title.textContent =
  newTitle;
```

instead of removing and recreating the entire component subtree.

Why?

Recreating nodes may also discard:

- focus state
- selection
- manually attached event listeners
- other DOM state

Choose the smallest reasonable update.

---

## 24. Event Listeners and Removal

If you remove an element from the DOM, references to that element may still exist in JavaScript.

Example:

```js
const button =
  document.querySelector(
    "button"
  );

button.remove();

console.log(button);
```

The JavaScript variable still references the node.

Garbage collection can happen only when the node is no longer reachable.

---

## 25. Memory Leak Connection

Detached DOM nodes can contribute to memory problems when application code keeps unnecessary references to them.

Example:

```text
DOM node removed
but
JavaScript cache still references it
↓
cannot be garbage collected
```

This connects to your memory-management lessons.

---

## 26. React Connection

React normally handles creation, insertion, updating, and removal for you.

Example:

```jsx
{jobs.map((job) => (
  <li key={job.id}>
    {job.title}
  </li>
))}
```

React decides which DOM nodes should be created, updated, moved, or removed.

Manual DOM APIs remain useful for:

- portals/integration code
- third-party widgets
- refs
- non-React pages
- interview fundamentals

---

## 27. Common Mistakes

### Mistake 1

Creating an element but never appending it.

### Mistake 2

Appending an existing element and expecting a duplicate.

### Mistake 3

Using `innerHTML` with untrusted user content.

### Mistake 4

Recreating large DOM trees when a small update is enough.

---

## Interview Questions

### What does document.createElement() do?

It creates a new element node in memory. The element becomes visible only after insertion into the document.

### append vs appendChild?

`append()` accepts multiple nodes or strings. `appendChild()` accepts one node and returns it.

### What happens if you append an element that is already in the DOM?

The existing node is moved to the new location.

### What does cloneNode(true) mean?

Create a deep clone including descendant nodes.

---

## Key Takeaways

- `createElement()` creates DOM elements.
- Created elements are not visible until inserted.
- `append`, `prepend`, `before`, and `after` control insertion.
- Existing nodes are moved, not duplicated.
- `cloneNode()` creates copies.
- `remove()` removes an element.
- `replaceWith()` and `replaceChildren()` help replace structure.
- `DocumentFragment` is useful for assembling groups of nodes.
- DOM removal and JavaScript reachability are separate concerns.
