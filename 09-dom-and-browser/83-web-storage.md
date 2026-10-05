# Lesson 83 — `localStorage` and `sessionStorage`

Browsers provide simple key-value storage through the Web Storage API.

The two main APIs are:

- `localStorage`
- `sessionStorage`

They look very similar, but their lifetimes are different.

The most important rule is:

> Web Storage stores string key-value pairs in the browser.

---

## 1. localStorage

`localStorage` persists data across page reloads and browser restarts until that data is explicitly removed or cleared by the browser/user.

Example:

```js
localStorage.setItem(
  "theme",
  "dark"
);
```

Read it:

```js
const theme =
  localStorage.getItem(
    "theme"
  );

console.log(theme);
```

Output:

```text
dark
```

---

# 2. sessionStorage

`sessionStorage` uses almost the same API:

```js
sessionStorage.setItem(
  "step",
  "2"
);
```

Read:

```js
sessionStorage.getItem(
  "step"
);
```

The main difference is lifetime and browsing context.

`sessionStorage` is scoped to the page's session/tab context and is normally cleared when that tab/window session ends.

---

# 3. Basic Comparison

```text
localStorage
→ persists until removed/cleared

sessionStorage
→ lives for the current tab/session
```

Both are scoped by origin.

---

# 4. Origin Scope

Storage is separated by origin.

Origin is based on:

```text
scheme
+
host
+
port
```

So these can have separate storage:

```text
https://example.com
http://example.com
https://example.com:3000
```

This prevents unrelated origins from reading each other's storage.

---

# 5. setItem()

Syntax:

```js
storage.setItem(
  key,
  value
);
```

Example:

```js
localStorage.setItem(
  "name",
  "Vikash"
);
```

If the key already exists, its value is replaced.

---

# 6. getItem()

```js
localStorage.getItem(
  "name"
);
```

If the key does not exist:

```text
null
```

This is different from:

```text
undefined
```

---

# 7. removeItem()

```js
localStorage.removeItem(
  "name"
);
```

This removes only that key.

---

# 8. clear()

```js
localStorage.clear();
```

This removes all keys for that storage area and origin.

Use carefully.

---

# 9. key() and length

Count:

```js
localStorage.length;
```

Access a key by index:

```js
localStorage.key(0);
```

Useful for iteration/debugging.

---

# 10. Values Are Strings

This is one of the most important Web Storage rules.

```js
localStorage.setItem(
  "count",
  10
);
```

Read:

```js
localStorage.getItem(
  "count"
);
```

Result:

```text
"10"
```

not numeric `10`.

---

# 11. Storing Objects

This does not work as intended:

```js
const user = {
  name: "Vikash",
};

localStorage.setItem(
  "user",
  user
);
```

The object is converted to a string such as:

```text
[object Object]
```

Instead use JSON.

---

# 12. JSON.stringify()

```js
const user = {
  name: "Vikash",
  role: "Developer",
};

localStorage.setItem(
  "user",
  JSON.stringify(user)
);
```

Read:

```js
const raw =
  localStorage.getItem(
    "user"
  );

const parsed =
  JSON.parse(raw);

console.log(
  parsed.name
);
```

---

# 13. Safe JSON Parsing

Stored data may be malformed, stale, or manually changed.

Safer:

```js
function readJson(
  key,
  fallback
) {
  const value =
    localStorage.getItem(
      key
    );

  if (value === null) {
    return fallback;
  }

  try {
    return JSON.parse(
      value
    );
  } catch {
    return fallback;
  }
}
```

Do not assume storage always contains valid JSON.

---

# 14. localStorage Is Synchronous

Web Storage methods are synchronous.

Example:

```js
localStorage.getItem(
  "theme"
);
```

runs on the current JavaScript thread.

For small settings this is normally fine.

But large or excessive storage operations can block the main thread.

Web Storage is not ideal for large structured datasets.

---

# 15. Storage Size Is Limited

Browser storage limits vary.

Do not treat `localStorage` as an unlimited database.

If quota is exceeded, writes can fail.

Use browser storage for appropriately small data such as:

- preferences
- lightweight drafts
- UI settings

For larger structured client-side data, IndexedDB may be more suitable.

---

# 16. Do Not Store Sensitive Secrets

Avoid storing highly sensitive values such as authentication secrets in Web Storage when exposure to JavaScript is a security concern.

Why?

Any JavaScript running in the page's origin can potentially access:

```js
localStorage
```

A successful XSS attack could therefore read stored values.

Security design should not rely on Web Storage being secret.

---

# 17. Authentication Token Discussion

You may see access tokens stored in:

```text
localStorage
```

But this has security tradeoffs because JavaScript can read them.

For many authentication designs, secure `HttpOnly` cookies are preferred for long-lived sensitive credentials because client JavaScript cannot read them.

The correct architecture depends on application requirements.

Cookies are covered in Lesson 84.

---

# 18. localStorage Use Case — Theme

```js
const savedTheme =
  localStorage.getItem(
    "theme"
  );

if (savedTheme) {
  document.documentElement
    .dataset.theme =
      savedTheme;
}
```

Update:

```js
localStorage.setItem(
  "theme",
  "dark"
);
```

This is a good lightweight persistence use case.

---

# 19. sessionStorage Use Case — Multi-Step Form

Suppose a user is completing a multi-step form.

```js
sessionStorage.setItem(
  "currentStep",
  "3"
);
```

Reloading the tab can preserve the step during that session.

When the tab/session ends, the data is normally discarded.

---

# 20. sessionStorage Is Tab-Scoped

A useful simplified mental model:

```text
Tab A
→ own sessionStorage

Tab B
→ separate sessionStorage
```

Even for the same origin, session storage is associated with the browsing context/session.

There are nuances around duplicated/opened tabs, but for normal application reasoning, treat each tab session separately.

---

# 21. localStorage Across Tabs

For the same origin, `localStorage` is shared across tabs/windows.

So one tab can update data that another tab also has access to.

This enables cross-tab coordination.

---

# 22. storage Event

Other same-origin documents can receive a `storage` event when storage changes.

Example:

```js
window.addEventListener(
  "storage",
  (event) => {
    console.log(
      event.key,
      event.oldValue,
      event.newValue
    );
  }
);
```

Important:

The page that performs the `localStorage` update does not normally receive its own `storage` event for that change.

Other relevant same-origin documents do.

---

# 23. Useful storage Event Fields

```js
event.key
event.oldValue
event.newValue
event.url
event.storageArea
```

This can support:

- logout synchronization
- preference synchronization
- cross-tab state hints

---

# 24. Example — Cross-Tab Logout Hint

Tab A:

```js
localStorage.setItem(
  "logout",
  String(Date.now())
);
```

Tab B:

```js
window.addEventListener(
  "storage",
  (event) => {
    if (
      event.key ===
      "logout"
    ) {
      redirectToLogin();
    }
  }
);
```

The storage value acts as a notification signal.

---

# 25. Property Syntax vs Storage Methods

You may see:

```js
localStorage.theme =
  "dark";
```

But the explicit methods are generally clearer:

```js
localStorage.setItem(
  "theme",
  "dark"
);
```

They communicate storage intent and avoid awkward key/property collisions.

---

# 26. Handling Missing Browser Environment

In server-side rendering environments, this code may fail:

```js
localStorage.getItem(
  "theme"
);
```

because `localStorage` is a browser API.

In Next.js, access it only in client/browser execution contexts.

---

# 27. Next.js Connection

In a Client Component:

```jsx
"use client";

import {
  useEffect,
  useState,
} from "react";

export default function Theme() {
  const [
    theme,
    setTheme,
  ] =
    useState("light");

  useEffect(() => {
    const stored =
      localStorage.getItem(
        "theme"
      );

    if (stored) {
      setTheme(stored);
    }
  }, []);

  return (
    <div>{theme}</div>
  );
}
```

The browser API is accessed after client mounting.

Libraries such as `next-themes` abstract many theme-storage details.

---

# 28. Web Storage vs Cookies Preview

```text
Web Storage
→ JavaScript-controlled client storage
→ not automatically sent with HTTP requests

Cookies
→ can be automatically sent with matching HTTP requests
→ can use security attributes
```

This is the fundamental distinction for the next lesson.

---

# 29. Common Mistakes

### Mistake 1

Expecting objects to be stored automatically.

### Mistake 2

Forgetting all values are strings.

### Mistake 3

Using `localStorage` in server-side code.

### Mistake 4

Treating `localStorage` as secure secret storage.

### Mistake 5

Using Web Storage as a large database.

---

# Interview Questions

### localStorage vs sessionStorage?

Both store string key-value pairs. `localStorage` persists until removed, while `sessionStorage` is tied to the current browsing session/tab.

### Is localStorage asynchronous?

No. The Web Storage API is synchronous.

### Are localStorage values automatically sent to the server?

No.

### How do you store an object?

Use `JSON.stringify()` when writing and `JSON.parse()` when reading.

---

# Key Takeaways

- Web Storage provides string key-value storage.
- `localStorage` persists across browser sessions.
- `sessionStorage` is session/tab scoped.
- Both are scoped by origin.
- Web Storage is synchronous.
- Objects require JSON serialization.
- Storage quota is limited.
- JavaScript can read Web Storage, so do not treat it as secret storage.
- `storage` events can coordinate same-origin tabs.
- Web Storage is not automatically included in HTTP requests.
