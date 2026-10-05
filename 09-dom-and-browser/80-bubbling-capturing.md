# Lesson 80 — Event Bubbling and Capturing

DOM events can travel through the DOM tree.

This propagation model has three conceptual phases:

```text
1. Capturing phase
2. Target phase
3. Bubbling phase
```

Understanding propagation is required before event delegation makes sense.

---

## 1. Example DOM Tree

```html
<div id="outer">
  <div id="inner">
    <button id="save">
      Save
    </button>
  </div>
</div>
```

Conceptually:

```text
document
↓
outer
↓
inner
↓
button
```

Suppose the button is clicked.

---

# 2. Capturing Phase

During capturing, the event travels downward toward the target.

Simplified:

```text
document
↓
outer
↓
inner
↓
button
```

Listeners configured for capture can run during this phase.

---

# 3. Target Phase

The event reaches the element where it originated.

```text
button
```

This is the target.

---

# 4. Bubbling Phase

For events that bubble, the event then travels back upward.

```text
button
↑
inner
↑
outer
↑
document
```

Normal `addEventListener` listeners use bubbling-phase behavior by default.

---

# 5. Default Bubbling Example

```js
outer.addEventListener(
  "click",
  () => {
    console.log("outer");
  }
);

inner.addEventListener(
  "click",
  () => {
    console.log("inner");
  }
);

save.addEventListener(
  "click",
  () => {
    console.log("button");
  }
);
```

Click button.

Typical output:

```text
button
inner
outer
```

The event bubbles upward.

---

# 6. Capturing Listener

Use:

```js
element.addEventListener(
  "click",
  handler,
  {
    capture: true,
  }
);
```

or old shorthand:

```js
element.addEventListener(
  "click",
  handler,
  true
);
```

---

# 7. Capturing Example

```js
outer.addEventListener(
  "click",
  () => {
    console.log(
      "outer capture"
    );
  },
  {
    capture: true,
  }
);

save.addEventListener(
  "click",
  () => {
    console.log(
      "button"
    );
  }
);
```

Clicking the button can output:

```text
outer capture
button
```

because capture travels downward before target/bubble listeners.

---

# 8. Full Propagation Diagram

For a click on button:

```text
CAPTURE

window
↓
document
↓
html
↓
body
↓
outer
↓
inner
↓

TARGET

button

BUBBLE

inner
↑
outer
↑
body
↑
html
↑
document
↑
window
```

This is a conceptual model; exact propagation paths depend on the DOM/event.

---

# 9. event.eventPhase

The event object can expose the current phase:

```js
event.eventPhase
```

Values correspond to:

- capturing
- at target
- bubbling

You rarely need this in everyday code, but it is useful for understanding/debugging.

---

# 10. event.target Does Not Change While Bubbling

Suppose the click started on:

```html
<button id="save">
```

Even when the event reaches `outer`:

```js
outer.addEventListener(
  "click",
  (event) => {
    console.log(
      event.target
    );
  }
);
```

`event.target` still identifies the original target.

---

# 11. currentTarget Changes

At different listeners:

```text
button listener
currentTarget = button

inner listener
currentTarget = inner

outer listener
currentTarget = outer
```

But:

```text
target = original clicked element
```

throughout propagation.

---

# 12. Not Every Event Bubbles

Many common events bubble:

- click
- input
- change
- keydown

But some events behave differently.

Examples:

- `focus`
- `blur`
- `mouseenter`
- `mouseleave`

Often there are bubbling alternatives:

```text
focus → focusin
blur → focusout
```

Always check the event's propagation behavior when designing delegation.

---

# 13. Why Bubbling Is Useful

Bubbling allows a parent to react to events originating from descendants.

This enables event delegation:

```text
one parent listener
↓
many child interactions
```

Lesson 81 is entirely about that pattern.

---

# 14. Why Capturing Exists

Capturing can be useful when a parent should observe an event before target/bubble listeners.

Possible uses:

- global interception
- analytics
- advanced UI infrastructure
- debugging

Most everyday application listeners use the default bubbling behavior.

---

# 15. Listener Order on Same Element

If multiple listeners are registered on the same element/phase, they generally run in registration order.

Example:

```js
button.addEventListener(
  "click",
  () => {
    console.log("A");
  }
);

button.addEventListener(
  "click",
  () => {
    console.log("B");
  }
);
```

Typical output:

```text
A
B
```

Propagation controls can affect this, which is covered in Lesson 82.

---

# 16. Nested Interactive Elements

Be careful with invalid or confusing interactive nesting.

For example, putting clickable controls inside other clickable controls can make propagation and accessibility harder to reason about.

Prefer semantic HTML first.

---

# 17. Practical Example

HTML:

```html
<div id="card">
  <button id="save">
    Save
  </button>
</div>
```

Listeners:

```js
card.addEventListener(
  "click",
  () => {
    console.log(
      "card clicked"
    );
  }
);

save.addEventListener(
  "click",
  () => {
    console.log(
      "save clicked"
    );
  }
);
```

Clicking Save:

```text
save clicked
card clicked
```

That is bubbling.

---

# 18. React Connection

React events also use propagation concepts.

Example:

```jsx
<div
  onClick={() => {
    console.log(
      "parent"
    );
  }}
>
  <button
    onClick={() => {
      console.log(
        "child"
      );
    }}
  >
    Save
  </button>
</div>
```

Clicking the button can trigger child then parent handling because of propagation.

---

# 19. Common Mistakes

### Mistake 1

Thinking event.target changes as the event bubbles.

### Mistake 2

Thinking all events bubble.

### Mistake 3

Using capture without understanding why.

### Mistake 4

Confusing propagation with browser default actions.

Propagation and default behavior are separate concepts.

---

# Interview Questions

### What is event bubbling?

The phase where a bubbling event travels from the target upward through ancestor elements.

### What is event capturing?

The earlier phase where the event travels from outer ancestors down toward the target.

### Does event.target change during bubbling?

No. It remains the original event target.

### What changes during propagation?

`currentTarget` changes based on which listener is currently executing.

---

# Key Takeaways

- DOM events can propagate through the tree.
- Main phases are capture, target, and bubble.
- Most listeners use bubbling by default.
- `capture: true` registers a capturing listener.
- `target` identifies where the event originated.
- `currentTarget` identifies the listener currently running.
- Not every event bubbles.
- Bubbling enables event delegation.
