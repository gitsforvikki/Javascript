# Lesson 79 — Browser Events and Event Listeners

Browser applications are event-driven.

An **event** represents something that happened in the browser.

Examples:

- user clicked a button
- user typed in an input
- form was submitted
- page finished loading
- mouse moved
- keyboard key was pressed
- network-related browser activity occurred

JavaScript can listen for these events and run code in response.

---

## 1. Basic Event Listener

HTML:

```html
<button id="save">
  Save
</button>
```

JavaScript:

```js
const button =
  document.querySelector(
    "#save"
  );

button.addEventListener(
  "click",
  () => {
    console.log(
      "button clicked"
    );
  }
);
```

The callback runs when the click event occurs.

---

# 2. addEventListener()

Syntax:

```js
element.addEventListener(
  eventType,
  handler
);
```

Example:

```js
input.addEventListener(
  "input",
  handleInput
);
```

This registers a handler without immediately calling it.

---

# 3. Do Not Call the Handler During Registration

Wrong:

```js
button.addEventListener(
  "click",
  handleClick()
);
```

This executes `handleClick` immediately and passes its return value.

Correct:

```js
button.addEventListener(
  "click",
  handleClick
);
```

or:

```js
button.addEventListener(
  "click",
  () => {
    handleClick();
  }
);
```

---

# 4. The Event Object

Browsers pass an event object to the handler.

```js
button.addEventListener(
  "click",
  (event) => {
    console.log(event);
  }
);
```

The event object contains information about what happened.

Common properties:

```js
event.type
event.target
event.currentTarget
event.timeStamp
```

Different event types expose additional properties.

---

# 5. event.target

`event.target` is the element where the event originated.

Example:

```html
<button id="save">
  <span>Save</span>
</button>
```

If the user clicks the `span`:

```js
button.addEventListener(
  "click",
  (event) => {
    console.log(
      event.target
    );
  }
);
```

the target may be the `span`.

This becomes very important in event delegation.

---

# 6. event.currentTarget

`event.currentTarget` is the element whose listener is currently running.

Example:

```js
button.addEventListener(
  "click",
  (event) => {
    console.log(
      event.currentTarget
    );
  }
);
```

Inside this handler:

```text
currentTarget = button
```

even if a nested child was clicked.

---

# 7. target vs currentTarget

Remember:

```text
target
→ where event started

currentTarget
→ element whose listener
   is currently executing
```

This is one of the most important browser-event distinctions.

---

# 8. Common Mouse Events

Examples:

```text
click
dblclick
mousedown
mouseup
mousemove
mouseenter
mouseleave
mouseover
mouseout
```

Not all mouse events propagate in exactly the same way.

For example, `mouseenter` and `mouseleave` behave differently from `mouseover` and `mouseout`.

---

# 9. Keyboard Events

Common:

```js
document.addEventListener(
  "keydown",
  (event) => {
    console.log(
      event.key
    );
  }
);
```

Useful properties:

```js
event.key
event.code
event.ctrlKey
event.shiftKey
event.altKey
event.metaKey
```

---

# 10. key vs code

`event.key` represents the interpreted key value.

Example:

```text
"a"
"A"
"Enter"
```

`event.code` represents the physical keyboard key location.

Example:

```text
"KeyA"
"Enter"
```

Use the one that matches your intent.

---

# 11. Input Events

```js
input.addEventListener(
  "input",
  (event) => {
    console.log(
      event.target.value
    );
  }
);
```

The `input` event fires as the field value changes.

---

# 12. change Event

```js
select.addEventListener(
  "change",
  (event) => {
    console.log(
      event.target.value
    );
  }
);
```

The exact timing of `change` depends on the form control.

For text inputs, `input` is usually better for immediate updates.

---

# 13. Form Submit Event

```js
form.addEventListener(
  "submit",
  (event) => {
    console.log(
      "submitted"
    );
  }
);
```

A form submit has browser default behavior, which will be discussed in Lesson 82.

---

# 14. Focus Events

Common events:

```text
focus
blur
focusin
focusout
```

`focusin` and `focusout` bubble.

`focus` and `blur` traditionally do not bubble in the same way.

---

# 15. load and DOMContentLoaded

```js
window.addEventListener(
  "load",
  () => {
    console.log(
      "page resources loaded"
    );
  }
);
```

`load` generally waits for resources such as images.

```js
document.addEventListener(
  "DOMContentLoaded",
  () => {
    console.log(
      "DOM parsed"
    );
  }
);
```

These events represent different readiness stages.

---

# 16. Multiple Listeners

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

Both handlers can run.

This is one major advantage of `addEventListener` over assigning a single `onclick` property.

---

# 17. onclick Property

You may see:

```js
button.onclick =
  function () {
    console.log(
      "clicked"
    );
  };
```

This works, but assigning another handler:

```js
button.onclick =
  anotherHandler;
```

replaces the previous one.

`addEventListener` supports multiple listeners.

---

# 18. removeEventListener()

To remove a listener, you need the same function reference.

Correct:

```js
function handleClick() {
  console.log("clicked");
}

button.addEventListener(
  "click",
  handleClick
);

button.removeEventListener(
  "click",
  handleClick
);
```

---

# 19. Anonymous Function Removal Problem

This will not remove the original listener:

```js
button.addEventListener(
  "click",
  () => {
    console.log("clicked");
  }
);

button.removeEventListener(
  "click",
  () => {
    console.log("clicked");
  }
);
```

Those are two different function objects.

---

# 20. once Option

```js
button.addEventListener(
  "click",
  handleClick,
  {
    once: true,
  }
);
```

The browser automatically removes the listener after its first invocation.

---

# 21. passive Option

```js
element.addEventListener(
  "touchmove",
  handler,
  {
    passive: true,
  }
);
```

A passive listener promises not to call `preventDefault()`.

This can help browser scrolling performance for certain input events.

Do not set it blindly if you need to prevent the default action.

---

# 22. AbortSignal for Listener Cleanup

Modern browsers can remove listeners through an `AbortController`.

```js
const controller =
  new AbortController();

button.addEventListener(
  "click",
  handleClick,
  {
    signal:
      controller.signal,
  }
);
```

Later:

```js
controller.abort();
```

This removes listeners registered with that signal.

Useful for grouped cleanup.

---

# 23. this in Regular Event Handlers

With a normal function:

```js
button.addEventListener(
  "click",
  function (event) {
    console.log(
      this ===
        event.currentTarget
    );
  }
);
```

For a normal DOM event listener callback, `this` is generally the current target.

But arrow functions use lexical `this`.

So prefer `event.currentTarget` when you want explicit, readable code.

---

# 24. Event Listener Memory Considerations

Listeners can keep objects reachable through closures.

If a listener is no longer needed, remove it when appropriate.

This is especially important in:

- long-lived pages
- custom widgets
- manual SPA logic
- repeated mounting/unmounting

---

# 25. CustomEvent Preview

Browsers can also dispatch custom events.

```js
const event =
  new CustomEvent(
    "job:saved",
    {
      detail: {
        id: 42,
      },
    }
  );

document.dispatchEvent(
  event
);
```

Listen:

```js
document.addEventListener(
  "job:saved",
  (event) => {
    console.log(
      event.detail
    );
  }
);
```

Useful for browser-level event communication, though frameworks often provide their own state/event patterns.

---

# 26. React Connection

React uses a higher-level event system, but concepts such as:

- target
- currentTarget
- default behavior
- propagation
- keyboard/mouse events

still matter.

Example:

```jsx
<button
  onClick={(event) => {
    console.log(
      event.currentTarget
    );
  }}
>
  Save
</button>
```

Understanding native events makes React events easier to reason about.

---

# Interview Questions

### target vs currentTarget?

`target` is where the event originated; `currentTarget` is the element whose listener is currently executing.

### addEventListener vs onclick?

`addEventListener` supports multiple handlers and options; assigning `onclick` replaces the previous property handler.

### Why can removeEventListener fail?

Because the same function reference used during registration must generally be supplied.

---

# Key Takeaways

- Browser applications are event-driven.
- `addEventListener()` registers event handlers.
- Handlers receive an event object.
- `target` and `currentTarget` are different.
- Different event types expose different data.
- Multiple listeners can exist for one event.
- Listener removal requires matching function references.
- Options such as `once`, `passive`, and `signal` affect listener behavior.
