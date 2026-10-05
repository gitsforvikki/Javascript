# Lesson 114 — Event Listener and Timer Cleanup

Cleanup means releasing resources when they are no longer needed.

This lesson focuses on two common sources of leaks and stale behavior:

- event listeners
- timers

---

## 1. Event Listener Lifecycle

Setup:

```js
button.addEventListener(
  "click",
  handleClick
);
```

Cleanup:

```js
button.removeEventListener(
  "click",
  handleClick
);
```

The same function reference must be used.

---

## 2. Anonymous Listener Problem

Bad:

```js
button.addEventListener(
  "click",
  () => {
    save();
  }
);
```

Later this does not remove it:

```js
button.removeEventListener(
  "click",
  () => {
    save();
  }
);
```

Those are different function objects.

---

## 3. Correct Named Handler

```js
function handleClick() {
  save();
}

button.addEventListener(
  "click",
  handleClick
);

// later

button.removeEventListener(
  "click",
  handleClick
);
```

---

## 4. Listener Options Matter

If capture mode differs, removal may fail.

```js
element.addEventListener(
  "click",
  handler,
  {
    capture: true,
  }
);
```

Cleanup must match the relevant capture setting.

---

## 5. AbortSignal-Based Cleanup

Modern pattern:

```js
const controller =
  new AbortController();

window.addEventListener(
  "resize",
  handleResize,
  {
    signal:
      controller.signal,
  }
);

document.addEventListener(
  "visibilitychange",
  handleVisibility,
  {
    signal:
      controller.signal,
  }
);
```

Then:

```js
controller.abort();
```

removes all signal-linked listeners.

---

## 6. setTimeout Cleanup

```js
const id =
  setTimeout(
    task,
    1000
  );
```

Cancel:

```js
clearTimeout(id);
```

Useful when pending work becomes irrelevant.

---

## 7. setInterval Cleanup

```js
const id =
  setInterval(
    refresh,
    5000
  );
```

Stop:

```js
clearInterval(id);
```

Intervals should almost always have a clear owner/lifecycle.

---

## 8. Recursive setTimeout Pattern

Instead of:

```js
setInterval(
  asyncTask,
  1000
);
```

you may use:

```js
let stopped =
  false;

async function loop() {
  if (stopped) {
    return;
  }

  await asyncTask();

  setTimeout(
    loop,
    1000
  );
}
```

This avoids overlapping async interval executions.

---

## 9. requestAnimationFrame Cleanup

```js
const id =
  requestAnimationFrame(
    update
  );
```

Cancel:

```js
cancelAnimationFrame(
  id
);
```

Useful when animation work should stop.

---

## 10. requestIdleCallback Cleanup

Where supported:

```js
const id =
  requestIdleCallback(
    work
  );
```

Cancel:

```js
cancelIdleCallback(
  id
);
```

Do not assume universal browser support without checking.

---

## 11. Observer Cleanup

```js
const observer =
  new IntersectionObserver(
    callback
  );

observer.observe(
  element
);
```

Stop observing one target:

```js
observer.unobserve(
  element
);
```

Or all:

```js
observer.disconnect();
```

---

## 12. React useEffect Cleanup

```js
useEffect(() => {
  window.addEventListener(
    "scroll",
    handleScroll
  );

  return () => {
    window.removeEventListener(
      "scroll",
      handleScroll
    );
  };
}, []);
```

Cleanup runs before the effect is disposed/re-run according to React lifecycle behavior.

---

## 13. Timer Cleanup in Effect

```js
useEffect(() => {
  const id =
    setTimeout(
      load,
      1000
    );

  return () => {
    clearTimeout(id);
  };
}, []);
```

---

## 14. Debounce Cleanup

If your debounced function exposes:

```js
debounced.cancel();
```

call it when the owner is destroyed.

Otherwise a trailing callback may execute after the UI is gone.

---

## 15. Throttle Cleanup

Similarly:

```js
throttled.cancel?.();
```

is useful if the implementation has pending trailing work.

---

## 16. Cleanup Ownership

A useful rule:

> The code that creates a long-lived resource should define how and when it is cleaned up.

Examples:

```text
add listener
→ remove listener

start timer
→ clear timer

subscribe
→ unsubscribe

open socket
→ close socket

start observer
→ disconnect
```

---

## 17. Idempotent Cleanup

Cleanup should ideally be safe to call more than once.

Example:

```js
let id =
  setInterval(
    task,
    1000
  );

function cleanup() {
  if (
    id !== null
  ) {
    clearInterval(id);
    id = null;
  }
}
```

This makes lifecycle management more robust.

---

## 18. Avoid Shared Unclear Ownership

Bad:

```text
module A starts timer
module B may clear it
module C stores id
```

Better:

```text
same module owns
setup + cleanup
```

Clear ownership prevents leaks.

---

## 19. Cleanup and Stale Closures

Even if memory is not huge, stale timers/listeners can execute old logic.

Example:

```text
old component state
captured in callback
↓
callback runs later
↓
incorrect behavior
```

Cleanup protects correctness, not only memory.

---

## 20. Interview Questions

### Why can removeEventListener fail?

Because the function reference or capture option does not match the original registration.

### Why clear timers?

To avoid unnecessary callbacks, overlapping work, stale state, and retained references.

### What is a good cleanup rule?

Every resource-creation step should have an explicit teardown path.

---

## Key Takeaways

- Clean up listeners and timers when their owner is done.
- Use the same handler reference for removal.
- AbortController can clean up groups of listeners.
- Clear timeout/interval/rAF work when irrelevant.
- Cleanup prevents stale behavior as well as leaks.
- Pair every setup operation with teardown.
- Keep ownership of setup and cleanup together.
