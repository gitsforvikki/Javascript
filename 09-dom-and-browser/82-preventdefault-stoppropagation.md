# Lesson 82 — `preventDefault()` vs `stopPropagation()`

These two event methods solve different problems:

```text
preventDefault()
→ stop browser default action

stopPropagation()
→ stop event propagation
```

They are not interchangeable.

---

## 1. What Is Default Browser Behavior?

Many HTML elements have built-in actions.

Examples:

- link navigates
- form submits
- checkbox toggles
- context menu opens
- browser drag behavior occurs

JavaScript can sometimes prevent these actions.

---

# 2. preventDefault()

Example:

```html
<a
  id="link"
  href="/dashboard"
>
  Dashboard
</a>
```

JavaScript:

```js
link.addEventListener(
  "click",
  (event) => {
    event.preventDefault();

    console.log(
      "navigation prevented"
    );
  }
);
```

The click event still exists and can still propagate.

Only the default navigation is prevented.

---

# 3. Form Example

```js
form.addEventListener(
  "submit",
  (event) => {
    event.preventDefault();

    const formData =
      new FormData(form);

    console.log(
      formData
    );
  }
);
```

This prevents the browser's normal form submission/navigation behavior.

Very common in JavaScript applications.

---

# 4. stopPropagation()

Suppose:

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
    console.log("card");
  }
);

save.addEventListener(
  "click",
  (event) => {
    event.stopPropagation();

    console.log("save");
  }
);
```

Clicking Save:

```text
save
```

The event is stopped from continuing to ancestor listeners.

---

# 5. stopPropagation Does Not Prevent Default

Suppose a link handler does:

```js
link.addEventListener(
  "click",
  (event) => {
    event.stopPropagation();
  }
);
```

The link may still navigate.

Propagation and default action are separate systems.

---

# 6. preventDefault Does Not Stop Bubbling

```js
parent.addEventListener(
  "click",
  () => {
    console.log("parent");
  }
);

link.addEventListener(
  "click",
  (event) => {
    event.preventDefault();

    console.log("link");
  }
);
```

Click link.

Possible output:

```text
link
parent
```

Navigation is prevented, but bubbling continues.

---

# 7. Main Comparison

```text
preventDefault()
│
└── browser action
    prevented

stopPropagation()
│
└── event travel
    stopped
```

Think:

```text
default behavior
≠
event propagation
```

---

# 8. stopImmediatePropagation()

Another method:

```js
event.stopImmediatePropagation();
```

This:

- stops propagation to ancestors
- prevents later listeners on the same element from running

---

# 9. Same Element Example

```js
button.addEventListener(
  "click",
  (event) => {
    event
      .stopImmediatePropagation();

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

Click:

```text
A
```

The second listener is prevented.

---

# 10. stopPropagation vs stopImmediatePropagation

```text
stopPropagation
→ stop moving through propagation path
→ other listeners on same element may still run

stopImmediatePropagation
→ also stop later listeners on same element
```

Use the stronger method sparingly.

---

# 11. event.defaultPrevented

You can check:

```js
event.defaultPrevented
```

This is true if `preventDefault()` successfully marked the event's default action as prevented.

Useful when multiple handlers coordinate behavior.

---

# 12. cancelable

Not every event's default behavior is cancelable.

Check:

```js
event.cancelable
```

If false, calling `preventDefault()` has no meaningful default-action cancellation effect.

---

# 13. Passive Listeners

If a listener is registered with:

```js
{
  passive: true
}
```

you are promising not to call `preventDefault()`.

So this combination is invalid in intent:

```js
element.addEventListener(
  "touchmove",
  (event) => {
    event.preventDefault();
  },
  {
    passive: true,
  }
);
```

The browser may ignore the prevention and warn.

---

# 14. Avoid stopPropagation by Default

A common mistake is to use `stopPropagation()` whenever parent and child handlers conflict.

That can hide design issues and break:

- analytics
- delegated handlers
- outer UI behavior
- third-party integrations

Use it only when the propagation itself is genuinely unwanted.

---

# 15. Better Conditional Handling

Sometimes the parent can ignore certain targets instead of the child stopping propagation.

Example:

```js
card.addEventListener(
  "click",
  (event) => {
    if (
      event.target.closest(
        "button"
      )
    ) {
      return;
    }

    openCard();
  }
);
```

This can preserve normal propagation for other listeners.

---

# 16. Form Validation Example

```js
form.addEventListener(
  "submit",
  (event) => {
    if (
      !form.checkValidity()
    ) {
      event.preventDefault();
    }
  }
);
```

The intent is specifically to control default submission.

No propagation stop is necessarily required.

---

# 17. Checkbox Example

```js
checkbox.addEventListener(
  "click",
  (event) => {
    event.preventDefault();
  }
);
```

The browser's normal checkbox toggling can be prevented.

This illustrates that `preventDefault()` applies to more than links/forms.

---

# 18. React Connection

React event objects expose:

```js
event.preventDefault();
event.stopPropagation();
```

The conceptual difference is the same.

Example:

```jsx
<form
  onSubmit={(event) => {
    event.preventDefault();

    submitData();
  }}
>
```

---

# 19. Common Mistakes

### Mistake 1

Using `stopPropagation()` to prevent a link from navigating.

### Mistake 2

Using `preventDefault()` expecting parent listeners not to run.

### Mistake 3

Using `stopImmediatePropagation()` without understanding same-element listener effects.

### Mistake 4

Calling `preventDefault()` inside a passive listener.

---

# Interview Questions

### preventDefault vs stopPropagation?

`preventDefault()` prevents the browser's default action. `stopPropagation()` stops the event from continuing through the propagation path.

### Does preventDefault stop bubbling?

No.

### Does stopPropagation prevent link navigation?

No, not by itself.

### What does stopImmediatePropagation do?

It stops propagation and prevents later listeners on the same element from running.

---

# Key Takeaways

- Default behavior and propagation are separate.
- `preventDefault()` controls browser default actions.
- `stopPropagation()` controls event travel.
- `stopImmediatePropagation()` is stronger and also affects same-element listeners.
- Not all events are cancelable.
- Passive listeners should not call `preventDefault()`.
- Avoid stopping propagation unless it is truly required.
