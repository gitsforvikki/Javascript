# Lesson 113 — Memory Leaks and Common Causes

A memory leak happens when memory is no longer useful to the application but remains reachable, so the garbage collector cannot reclaim it.

The core idea is:

> A leak is usually a reachability problem.

---

## 1. Common Leak Sources

Frequent causes include:

- global references
- forgotten timers
- event listeners
- detached DOM nodes
- growing caches
- subscriptions
- closures
- unresolved application registries
- long-lived Maps

---

## 2. Accidental Global State

```js
const cache = [];
```

If this array keeps receiving data forever:

```js
cache.push(
  largeObject
);
```

memory keeps growing because the global reference never disappears.

---

## 3. Unbounded Cache

```js
const cache =
  new Map();

function remember(
  key,
  value
) {
  cache.set(
    key,
    value
  );
}
```

If entries are never removed, a cache becomes a memory leak.

A cache needs a lifecycle policy:

- size limit
- TTL
- LRU
- explicit invalidation

---

## 4. Timers

```js
setInterval(() => {
  doSomething();
}, 1000);
```

If the interval is no longer needed but never cleared, it keeps its callback and captured references alive.

---

## 5. Event Listener Leaks

```js
window.addEventListener(
  "resize",
  handleResize
);
```

If a component/widget is destroyed but the listener remains, the callback may retain associated state.

---

## 6. Detached DOM Nodes

```js
const panel =
  document.querySelector(
    "#panel"
  );

panel.remove();
```

If some global array still stores:

```js
savedNodes.push(
  panel
);
```

the node remains reachable even though it is no longer in the DOM.

---

## 7. Closures Retaining Large Data

```js
function setup() {
  const hugeData =
    loadHugeData();

  return () => {
    console.log(
      hugeData.length
    );
  };
}
```

If the returned function is kept forever, so is `hugeData`.

---

## 8. Subscription Leak

```js
store.subscribe(
  listener
);
```

If the library returns:

```js
const unsubscribe =
  store.subscribe(
    listener
  );
```

then you should usually call:

```js
unsubscribe();
```

when the consumer no longer needs updates.

---

## 9. WebSocket Leak

```js
const socket =
  new WebSocket(url);
```

If a page/widget stops using it but leaves the connection open, you may retain:

- callbacks
- buffers
- state
- network resources

Close when appropriate:

```js
socket.close();
```

---

## 10. Observer APIs

APIs such as:

- MutationObserver
- ResizeObserver
- IntersectionObserver

can keep callbacks/references active.

Example:

```js
const observer =
  new ResizeObserver(
    callback
  );

observer.observe(
  element
);
```

Cleanup:

```js
observer.disconnect();
```

---

## 11. AbortController Helps Cleanup

A shared AbortController can simplify cleanup:

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
```

Later:

```js
controller.abort();
```

---

## 12. React Effect Leak

Bad:

```js
useEffect(() => {
  window.addEventListener(
    "resize",
    handleResize
  );
}, []);
```

Better:

```js
useEffect(() => {
  window.addEventListener(
    "resize",
    handleResize
  );

  return () => {
    window.removeEventListener(
      "resize",
      handleResize
    );
  };
}, []);
```

---

## 13. Timer Cleanup in React

```js
useEffect(() => {
  const id =
    setInterval(
      refresh,
      5000
    );

  return () => {
    clearInterval(id);
  };
}, []);
```

---

## 14. Async Request Lifetime

An old request may finish after UI state is no longer relevant.

Use:

- AbortController
- stale-request guards
- framework-supported cancellation

to avoid unnecessary retained work and stale updates.

---

## 15. Memory Leak vs High Memory Usage

High memory usage is not automatically a leak.

A legitimate application may hold:

- images
- cached data
- large datasets

A leak means memory remains retained unnecessarily and often grows over time.

---

## 16. Leak Pattern

Typical signal:

```text
perform action
↓
memory increases

return to idle
↓
memory does not fall enough

repeat action
↓
memory keeps climbing
```

This suggests retained objects.

---

## 17. WeakMap Can Help Sometimes

If metadata is keyed by object lifetime:

```js
const metadata =
  new WeakMap();
```

This avoids keeping the key object alive.

But WeakMap does not solve unrelated references elsewhere.

---

## 18. Common Anti-Pattern — "Set to null Everything"

Manually setting every variable to `null` is not normally required.

Use it only when:

- a long-lived object holds a large no-longer-needed reference
- clearing the reference meaningfully changes reachability

Garbage collection already handles local temporary values.

---

## 19. DevTools Workflow

To investigate:

```text
1. Reproduce growth
2. Take heap snapshot
3. Repeat action
4. Take another snapshot
5. Compare retained objects
6. Inspect retaining paths
```

This helps identify the actual reference chain.

---

## 20. Detached Elements in DevTools

A detached DOM node is:

```text
removed from document
but still referenced
```

These are classic leak candidates.

---

## 21. React Strict Mode Note

Development Strict Mode may intentionally run certain setup/cleanup flows more than once to surface unsafe effects.

This can expose missing cleanup.

Do not treat every duplicate dev-only effect as a memory leak.

---

## Interview Questions

### What is a memory leak?

Memory that is no longer useful but remains reachable and therefore cannot be reclaimed.

### Common causes?

Forgotten timers, listeners, subscriptions, caches, detached DOM references, and closures.

### Does high memory usage always mean leak?

No.

---

## Key Takeaways

- Leaks are reachability problems.
- Long-lived references are the main cause.
- Clean up timers, listeners, subscriptions, observers, and sockets.
- Bound your caches.
- Removed DOM nodes can still remain in memory.
- Use DevTools retaining paths to find why objects stay alive.
- Do not confuse legitimate memory usage with a leak.
