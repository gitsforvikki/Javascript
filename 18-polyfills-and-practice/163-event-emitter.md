# Lesson 163 — Implement an Event Emitter

An Event Emitter is a small publish/subscribe system.

It allows code to:

- subscribe to an event
- emit an event
- unsubscribe
- listen once

This is a very useful interview exercise because it combines:

- Maps
- Sets
- callbacks
- closures
- API design
- mutation during iteration

---

## 1. Desired API

```js
const emitter =
  new EventEmitter();

emitter.on(
  "message",
  handler
);

emitter.emit(
  "message",
  "Hello"
);

emitter.off(
  "message",
  handler
);
```

---

## 2. Mental Model

```text
event name
↓
collection of listeners
↓
emit
↓
call every listener
```

Example structure:

```text
"message"
→ listenerA
→ listenerB

"error"
→ listenerC
```

---

## 3. Why Map?

Use:

```js
new Map()
```

because event names map naturally to listener collections.

---

## 4. Why Set?

Use:

```js
new Set()
```

for listeners because:

- duplicate function references can be prevented
- deletion is easy
- iteration is straightforward

---

## 5. Basic Implementation

```js
class EventEmitter {
  constructor() {
    this.events =
      new Map();
  }

  on(
    eventName,
    listener
  ) {
    if (
      typeof listener !==
      "function"
    ) {
      throw new TypeError(
        "listener must be a function"
      );
    }

    if (
      !this.events.has(
        eventName
      )
    ) {
      this.events.set(
        eventName,
        new Set()
      );
    }

    this.events
      .get(
        eventName
      )
      .add(
        listener
      );

    return this;
  }

  emit(
    eventName,
    ...args
  ) {
    const listeners =
      this.events.get(
        eventName
      );

    if (!listeners) {
      return false;
    }

    for (
      const listener
      of listeners
    ) {
      listener(
        ...args
      );
    }

    return true;
  }

  off(
    eventName,
    listener
  ) {
    const listeners =
      this.events.get(
        eventName
      );

    if (!listeners) {
      return this;
    }

    listeners.delete(
      listener
    );

    if (
      listeners.size ===
      0
    ) {
      this.events.delete(
        eventName
      );
    }

    return this;
  }
}
```

---

## 6. Basic Usage

```js
const emitter =
  new EventEmitter();

function handleMessage(
  message
) {
  console.log(
    message
  );
}

emitter.on(
  "message",
  handleMessage
);

emitter.emit(
  "message",
  "Hello"
);
```

---

## 7. Unsubscribe

```js
emitter.off(
  "message",
  handleMessage
);
```

Future emits no longer call the listener.

---

## 8. once()

A once-listener should remove itself after the first execution.

```js
once(
  eventName,
  listener
) {
  const wrapper =
    (...args) => {
      this.off(
        eventName,
        wrapper
      );

      listener(
        ...args
      );
    };

  this.on(
    eventName,
    wrapper
  );

  return this;
}
```

---

## 9. Full once Example

```js
emitter.once(
  "ready",
  () => {
    console.log(
      "runs once"
    );
  }
);

emitter.emit(
  "ready"
);

emitter.emit(
  "ready"
);
```

Only the first emit triggers the listener.

---

## 10. Mutation During emit

This is subtle.

Suppose a listener removes another listener while emission is in progress.

Iterating the live Set can create surprising behavior.

Safer:

```js
const snapshot =
  [
    ...listeners,
  ];

for (
  const listener
  of snapshot
) {
  listener(
    ...args
  );
}
```

Now the current emission uses a stable snapshot.

---

## 11. Safer emit()

```js
emit(
  eventName,
  ...args
) {
  const listeners =
    this.events.get(
      eventName
    );

  if (
    !listeners ||
    listeners.size ===
      0
  ) {
    return false;
  }

  const snapshot =
    [
      ...listeners,
    ];

  for (
    const listener
    of snapshot
  ) {
    listener.apply(
      this,
      args
    );
  }

  return true;
}
```

---

## 12. Why listener.apply(this, args)?

Some event-emitter APIs invoke listeners with:

```js
this
```

bound to the emitter instance.

This is an API-design decision.

You can also choose:

```js
listener(
  ...args
);
```

if you want no special receiver semantics.

Be consistent.

---

## 13. Return an Unsubscribe Function

A very ergonomic design:

```js
on(
  eventName,
  listener
) {
  // register...

  return () => {
    this.off(
      eventName,
      listener
    );
  };
}
```

Usage:

```js
const unsubscribe =
  emitter.on(
    "message",
    handler
  );

unsubscribe();
```

---

## 14. Complete Interview-Friendly Version

```js
class EventEmitter {
  constructor() {
    this.events =
      new Map();
  }

  on(
    eventName,
    listener
  ) {
    if (
      typeof listener !==
      "function"
    ) {
      throw new TypeError(
        "listener must be a function"
      );
    }

    let listeners =
      this.events.get(
        eventName
      );

    if (!listeners) {
      listeners =
        new Set();

      this.events.set(
        eventName,
        listeners
      );
    }

    listeners.add(
      listener
    );

    return () => {
      this.off(
        eventName,
        listener
      );
    };
  }

  off(
    eventName,
    listener
  ) {
    const listeners =
      this.events.get(
        eventName
      );

    if (!listeners) {
      return false;
    }

    const removed =
      listeners.delete(
        listener
      );

    if (
      listeners.size ===
      0
    ) {
      this.events.delete(
        eventName
      );
    }

    return removed;
  }

  once(
    eventName,
    listener
  ) {
    let unsubscribe;

    const wrapper =
      (...args) => {
        unsubscribe();

        listener.apply(
          this,
          args
        );
      };

    unsubscribe =
      this.on(
        eventName,
        wrapper
      );

    return unsubscribe;
  }

  emit(
    eventName,
    ...args
  ) {
    const listeners =
      this.events.get(
        eventName
      );

    if (
      !listeners ||
      listeners.size ===
        0
    ) {
      return false;
    }

    const snapshot =
      [
        ...listeners,
      ];

    for (
      const listener
      of snapshot
    ) {
      listener.apply(
        this,
        args
      );
    }

    return true;
  }

  removeAllListeners(
    eventName
  ) {
    if (
      eventName ===
      undefined
    ) {
      this.events.clear();
    } else {
      this.events.delete(
        eventName
      );
    }
  }
}
```

---

## 15. Error Handling Policy

What happens if one listener throws?

Simple implementation:

```text
listener throws
↓
emit stops
↓
error propagates
```

Alternative design:

```text
catch each listener error
↓
continue others
```

Neither is universally correct.

The emitter API should define its policy.

---

## 16. Async Listeners

```js
emitter.on(
  "save",
  async () => {
    await save();
  }
);
```

A normal `emit()` does not automatically await returned Promises.

If you need that behavior, create a separate async API.

---

## 17. emitAsync Example

```js
async emitAsync(
  eventName,
  ...args
) {
  const listeners =
    [
      ...(
        this.events.get(
          eventName
        ) ?? []
      ),
    ];

  await Promise.all(
    listeners.map(
      (listener) =>
        listener.apply(
          this,
          args
        )
    )
  );
}
```

Now async listeners are coordinated.

---

## 18. EventEmitter vs DOM Events

DOM events include:

- bubbling
- capturing
- Event objects
- preventDefault
- stopPropagation

A simple custom EventEmitter usually has none of those.

It is just a publish/subscribe abstraction.

---

## 19. Real-World Uses

Event emitters appear in:

- Node.js
- WebSocket wrappers
- state systems
- SDKs
- plugins
- logging
- application event buses

---

## 20. Memory Leak Risk

If listeners are never removed:

```text
emitter
↓
listener
↓
captured data
```

all captured data may remain reachable.

Always design cleanup.

---

## 21. Complexity

For an event with `n` listeners:

```text
on:
average O(1)

off:
average O(1)

emit:
O(n)
```

Snapshot creation also uses:

```text
O(n)
```

temporary space.

---

## Interview Explanation

> I store event names in a Map and listeners in Sets. `on` registers, `off` removes, `once` wraps a listener and unsubscribes before invocation, and `emit` iterates over a snapshot so listeners can safely add/remove subscriptions during emission. I also return unsubscribe functions to make cleanup easy.

---

## Section 18 Complete Mental Model

```text
Array polyfills
→ iteration semantics

call / apply / bind
→ this binding

Promise.all
→ async coordination

debounce / throttle
→ event frequency control

memoize
→ caching

deep clone
→ graph copying

flatten
→ recursion / stack

currying
→ closures + argument accumulation

EventEmitter
→ publish / subscribe design
```

---

## Key Takeaways

- EventEmitter maps event names to listener collections.
- Map + Set is a clean design.
- `once` should remove itself.
- Snapshot listeners during emit for predictable mutation behavior.
- Returning unsubscribe functions improves cleanup.
- Async listeners require an explicit async emission policy.
- Listener cleanup matters for memory safety.
