# Lesson 01 — What is JavaScript and How Does It Work?

## What is JavaScript?

JavaScript is a **high-level programming language** mainly used to make web applications interactive and dynamic.

HTML provides structure, CSS provides styling, and JavaScript provides behavior.

Common uses include:
- Handling button clicks and user interactions
- Updating content without reloading the page
- Validating forms
- Calling APIs
- Building frontend applications with React/Next.js
- Building backend applications with Node.js

## How JavaScript Works

A JavaScript engine reads and executes JavaScript code.

Common engines:
- **V8** — Chrome and Node.js
- **SpiderMonkey** — Firefox
- **JavaScriptCore** — Safari

Simple flow:

```text
JavaScript Code
      ↓
JavaScript Engine
      ↓
Code is parsed and executed
      ↓
Output / Application Behavior
```

JavaScript itself is **single-threaded**, meaning one main JavaScript task executes at a time.

Asynchronous behavior such as timers, API requests, promises, and the event loop will be covered deeply later.

## Example

```js
const name = "Vikash";

console.log("Hello", name);
```

Output:

```text
Hello Vikash
```

## Key Takeaways

- JavaScript adds behavior and logic to applications.
- It runs using a JavaScript engine.
- V8 is used by Chrome and Node.js.
- JavaScript is single-threaded at its core.
- JavaScript can be used in both browsers and servers.

## Interview Quick Answer

**What is JavaScript?**

JavaScript is a high-level, dynamically typed programming language commonly used for building interactive web applications. It can run in browsers and outside browsers through runtimes such as Node.js.
