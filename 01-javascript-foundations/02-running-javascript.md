# Lesson 02 — Running JavaScript: Browser, Console and Node.js

JavaScript can run in different environments.

## 1. Browser

JavaScript can be added to an HTML page using the `script` tag.

```html
<script>
  console.log("Hello JavaScript");
</script>
```

Or from an external file:

```html
<script src="app.js"></script>
```

## 2. Browser Console

Open browser developer tools and use the **Console** tab.

```js
console.log("Hello from browser");
```

This is useful for quick testing and debugging.

## 3. Node.js

Node.js allows JavaScript to run outside the browser.

Example file:

```js
// app.js
const message = "Hello from Node.js";

console.log(message);
```

Run:

```bash
node app.js
```

## Browser vs Node.js

| Browser | Node.js |
| --- | --- |
| Runs JavaScript in the browser | Runs JavaScript outside the browser |
| Provides DOM APIs | Does not provide the browser DOM |
| Used mainly for frontend code | Commonly used for servers, scripts and tools |

The JavaScript language is the same, but the available APIs depend on the runtime environment.

## Key Takeaways

- JavaScript needs a runtime environment.
- Browsers execute frontend JavaScript.
- Node.js executes JavaScript outside the browser.
- Browser and Node.js provide different runtime APIs.

## Interview Quick Answer

**Can JavaScript run outside a browser?**

Yes. Runtimes such as Node.js allow JavaScript to execute outside the browser.
