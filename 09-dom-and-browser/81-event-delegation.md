# Lesson 81 — Event Delegation

Event delegation is a powerful pattern built on event bubbling.

The idea is:

> Instead of attaching a listener to every child, attach one listener to a common parent and determine which child triggered the event.

---

## 1. The Problem

Suppose:

```html
<ul id="jobs">
  <li>
    <button data-id="1">
      Save
    </button>
  </li>

  <li>
    <button data-id="2">
      Save
    </button>
  </li>
</ul>
```

You could attach a listener to every button.

```js
const buttons =
  document.querySelectorAll(
    "#jobs button"
  );

buttons.forEach((button) => {
  button.addEventListener(
    "click",
    handleSave
  );
});
```

This works.

But event delegation can be simpler.

---

# 2. Delegate to the Parent

```js
const list =
  document.querySelector(
    "#jobs"
  );

list.addEventListener(
  "click",
  (event) => {
    console.log(
      event.target
    );
  }
);
```

Because click events bubble, clicks on buttons can reach the list.

---

# 3. Identify Matching Children

Basic version:

```js
list.addEventListener(
  "click",
  (event) => {
    if (
      event.target.matches(
        "button[data-id]"
      )
    ) {
      console.log(
        event.target.dataset.id
      );
    }
  }
);
```

One parent listener handles many children.

---

# 4. The Nested-Element Problem

Suppose:

```html
<button data-id="1">
  <span>
    Save
  </span>
</button>
```

If the user clicks the `span`:

```text
event.target
→ span
```

Then:

```js
event.target.matches(
  "button"
)
```

is false.

This is why `closest()` is often better.

---

# 5. Use closest()

```js
list.addEventListener(
  "click",
  (event) => {
    const button =
      event.target.closest(
        "button[data-id]"
      );

    if (!button) {
      return;
    }

    console.log(
      button.dataset.id
    );
  }
);
```

Now clicks on nested content still find the intended button.

---

# 6. Boundary Check

`closest()` can theoretically find an ancestor outside the intended delegated container in some structures.

A robust pattern:

```js
const button =
  event.target.closest(
    "button[data-id]"
  );

if (
  !button ||
  !list.contains(button)
) {
  return;
}
```

Now the match is guaranteed to belong inside the delegated area.

---

# 7. Why Delegation Works

Because of bubbling:

```text
button clicked
↓
event target = button
↓
event bubbles
↓
parent list listener runs
↓
inspect target
↓
handle correct child
```

Without propagation, delegation would not work for normal bubbling events.

---

# 8. Dynamic Elements Benefit

This is one of the biggest benefits.

Suppose later:

```js
const button =
  document.createElement(
    "button"
  );

button.dataset.id =
  "3";

button.textContent =
  "Save";

list.append(button);
```

No new click listener is needed.

The existing parent listener already handles future matching descendants.

---

# 9. Compare Direct Listeners

Direct:

```text
button 1 → listener
button 2 → listener
button 3 → listener
...
```

Delegated:

```text
parent
└── one listener
    handles children
```

This can simplify event management.

---

# 10. Delegation Is Not Always Better

Use direct listeners when:

- only one/few elements exist
- event does not bubble appropriately
- child behavior is isolated
- delegation would make logic harder to understand

Delegation is a pattern, not a rule that every UI must use.

---

# 11. Form List Example

```html
<ul id="applications">
  <li data-id="10">
    <button
      data-action="delete"
    >
      Delete
    </button>
  </li>

  <li data-id="11">
    <button
      data-action="delete"
    >
      Delete
    </button>
  </li>
</ul>
```

JavaScript:

```js
const list =
  document.querySelector(
    "#applications"
  );

list.addEventListener(
  "click",
  (event) => {
    const button =
      event.target.closest(
        '[data-action="delete"]'
      );

    if (
      !button ||
      !list.contains(button)
    ) {
      return;
    }

    const item =
      button.closest(
        "[data-id]"
      );

    console.log(
      item.dataset.id
    );
  }
);
```

---

# 12. Multiple Actions with One Listener

HTML:

```html
<button
  data-action="edit"
>
  Edit
</button>

<button
  data-action="delete"
>
  Delete
</button>
```

Delegated handler:

```js
container.addEventListener(
  "click",
  (event) => {
    const button =
      event.target.closest(
        "[data-action]"
      );

    if (!button) {
      return;
    }

    const action =
      button.dataset.action;

    if (
      action === "edit"
    ) {
      editItem();
    }

    if (
      action === "delete"
    ) {
      deleteItem();
    }
  }
);
```

---

# 13. Event Delegation with Tables

Delegation is common with:

- tables
- menus
- lists
- cards
- dynamic forms

Example:

```js
table.addEventListener(
  "click",
  (event) => {
    const row =
      event.target.closest(
        "tr[data-id]"
      );

    if (!row) {
      return;
    }

    console.log(
      row.dataset.id
    );
  }
);
```

---

# 14. target vs currentTarget in Delegation

Inside:

```js
list.addEventListener(
  "click",
  (event) => {
    console.log(
      event.target
    );

    console.log(
      event.currentTarget
    );
  }
);
```

Usually:

```text
target
→ clicked descendant

currentTarget
→ list
```

This distinction is the core of delegation.

---

# 15. closest() vs matches()

Use:

```js
event.target.matches(
  selector
)
```

when the actual target must match.

Use:

```js
event.target.closest(
  selector
)
```

when nested elements should resolve to a containing actionable element.

For buttons with icons/spans, `closest()` is often more robust.

---

# 16. Events That Do Not Bubble

Delegation depends on propagation.

For non-bubbling events, normal bubbling delegation may not work.

Sometimes you can:

- use a bubbling alternative
- use capture
- attach direct listeners

Example:

```text
focus
→ use focusin for bubbling delegation
```

---

# 17. Performance Perspective

Delegation can reduce the number of listeners.

But do not make exaggerated claims such as:

> one listener is always dramatically faster.

Modern browsers handle many listeners well.

The bigger benefits are often:

- simpler dynamic-element handling
- easier cleanup
- centralized logic

---

# 18. React Connection

React developers rarely manually delegate routine click events because React manages event handling at a higher level.

But understanding delegation helps with:

- native event integrations
- third-party widgets
- large non-React DOM areas
- understanding React's event architecture historically
- interview questions

---

# 19. Common Mistakes

### Mistake 1

Checking only `event.target` when child elements can be clicked.

### Mistake 2

Forgetting container-boundary checks.

### Mistake 3

Trying to delegate an event that does not bubble.

### Mistake 4

Using one giant document-level listener for unrelated behavior.

Delegate at the nearest sensible stable parent.

---

# Interview Questions

### What is event delegation?

A pattern where a parent listener handles events from descendant elements using event bubbling.

### Why is event delegation useful for dynamically added elements?

Because new matching children automatically bubble events to the existing parent listener.

### Why is closest() useful in delegation?

It handles clicks on nested descendants inside an actionable element.

---

# Key Takeaways

- Delegation depends on bubbling.
- One stable parent can handle many children.
- `target` identifies the event origin.
- `currentTarget` is the delegated parent.
- `closest()` helps resolve nested click targets.
- Dynamic children usually require no new delegated listener.
- Delegate at a sensible container boundary.
