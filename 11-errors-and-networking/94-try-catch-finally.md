# Lesson 94 — `try`, `catch` and `finally`

JavaScript errors interrupt normal execution unless they are handled.

The main language-level tools for handling runtime errors are:

- `try`
- `catch`
- `finally`

The key idea is:

> Put code that may fail inside `try`, handle the failure in `catch`, and put cleanup that must run either way inside `finally`.

---

## 1. Basic try/catch

```js
try {
  JSON.parse(
    "{ invalid json }"
  );
} catch (error) {
  console.log(
    "Parsing failed"
  );
}
```

Without the `catch`, the thrown error would interrupt the current execution path.

---

## 2. What Happens When an Error Is Thrown?

Consider:

```js
console.log("A");

try {
  throw new Error(
    "Something failed"
  );

  console.log("B");
} catch (error) {
  console.log("C");
}

console.log("D");
```

Output:

```text
A
C
D
```

Why?

Once the error is thrown, JavaScript stops executing the remaining statements in that `try` block and searches for a matching handler.

---

## 3. The Error Object

A typical Error object exposes:

```js
error.name
error.message
error.stack
```

Example:

```js
try {
  throw new Error(
    "Invalid operation"
  );
} catch (error) {
  console.log(
    error.name
  );

  console.log(
    error.message
  );
}
```

Typical output:

```text
Error
Invalid operation
```

---

## 4. Built-In Error Types

Common built-in errors include:

```text
Error
TypeError
ReferenceError
SyntaxError
RangeError
URIError
AggregateError
```

Examples:

```js
null.foo;
```

can cause a `TypeError`.

```js
unknownVariable;
```

can cause a `ReferenceError`.

---

## 5. catch Parameter

Modern JavaScript lets you omit the catch parameter if you do not need it:

```js
try {
  riskyOperation();
} catch {
  console.log(
    "Something failed"
  );
}
```

Use the error parameter when you need details.

---

## 6. finally

`finally` runs after `try` / `catch` regardless of success or failure.

```js
try {
  console.log(
    "working"
  );
} catch (error) {
  console.log(
    "failed"
  );
} finally {
  console.log(
    "cleanup"
  );
}
```

Typical output:

```text
working
cleanup
```

If an error occurs:

```text
failed
cleanup
```

---

## 7. Why finally Exists

Use `finally` for cleanup that must happen either way.

Examples:

- hide loading indicator
- close resource
- release lock
- reset temporary state
- stop timing/measurement

Example:

```js
setLoading(true);

try {
  saveData();
} catch (error) {
  showError(error);
} finally {
  setLoading(false);
}
```

---

## 8. finally with return

This is subtle.

```js
function test() {
  try {
    return "try";
  } finally {
    console.log(
      "finally"
    );
  }
}
```

Output:

```text
finally
```

Return value:

```text
try
```

The `finally` block runs before the function actually completes.

---

## 9. finally Can Override return

Avoid this:

```js
function test() {
  try {
    return "try";
  } finally {
    return "finally";
  }
}
```

Result:

```text
finally
```

The `finally` return overrides the earlier return.

This is usually confusing and should be avoided.

---

## 10. finally Can Override an Error Too

```js
function test() {
  try {
    throw new Error(
      "Original"
    );
  } finally {
    throw new Error(
      "Cleanup failed"
    );
  }
}
```

The final observed error is:

```text
Cleanup failed
```

So cleanup code should be written carefully.

---

## 11. Nested try/catch

```js
try {
  try {
    riskyTask();
  } catch (error) {
    console.log(
      "inner"
    );

    throw error;
  }
} catch (error) {
  console.log(
    "outer"
  );
}
```

An inner catch can:

- recover
- transform
- rethrow

---

## 12. Rethrowing

```js
try {
  riskyOperation();
} catch (error) {
  console.error(
    "Logging locally",
    error
  );

  throw error;
}
```

This lets a higher-level caller decide what to do next.

---

## 13. Catch Only What You Can Handle

Bad pattern:

```js
try {
  doEverything();
} catch {
  // ignore
}
```

This hides errors.

Better:

```js
try {
  parseConfig();
} catch (error) {
  return defaultConfig;
}
```

Here, a fallback is intentional.

---

## 14. Synchronous try/catch Only Catches Synchronous Throws in Its Execution Context

This will not catch the later timer error:

```js
try {
  setTimeout(() => {
    throw new Error(
      "Later"
    );
  }, 1000);
} catch (error) {
  console.log(
    "Will not run"
  );
}
```

Why?

The timer callback executes later in a different call-stack turn.

---

## 15. Promise Connection

Promise errors are handled with:

```js
promise.catch(...)
```

or:

```js
try {
  await promise;
} catch (error) {
  // ...
}
```

This connects directly to your async error-handling lessons.

---

## 16. try/catch with await

```js
async function load() {
  try {
    const data =
      await fetchData();

    return data;
  } catch (error) {
    console.error(
      error
    );

    throw error;
  }
}
```

An awaited rejection behaves like a thrown error at the `await` point.

---

## 17. Syntax Errors vs Runtime Errors

A parsing-level syntax error can prevent the script/module from running at all.

Example:

```js
const = 10;
```

You cannot rely on a surrounding runtime `try/catch` in ordinary source code to recover from code that failed to parse.

But errors thrown by runtime parsing APIs such as:

```js
JSON.parse(...)
```

can be caught.

---

## 18. Error Boundaries by Layer

A useful application pattern:

```text
low-level function
→ throw meaningful error

service layer
→ add context / rethrow

UI/API boundary
→ convert to user response
```

Do not handle every error at the deepest possible level.

Handle it where you can make a meaningful decision.

---

## 19. Example — Safe JSON Parsing

```js
function parseJson(
  value,
  fallback = null
) {
  try {
    return JSON.parse(
      value
    );
  } catch {
    return fallback;
  }
}
```

This is a valid recovery because fallback behavior is intentional.

---

## 20. Example — Form Submission

```js
async function submitForm() {
  setSubmitting(true);

  try {
    const result =
      await saveForm();

    showSuccess(result);
  } catch (error) {
    showError(
      error.message
    );
  } finally {
    setSubmitting(false);
  }
}
```

Clear separation:

```text
try
→ success path

catch
→ failure path

finally
→ cleanup
```

---

## Interview Questions

### What does finally do?

It runs after `try`/`catch` regardless of whether an error occurred, usually for cleanup.

### Can catch recover execution?

Yes. If it handles the error and does not rethrow, execution can continue.

### Can try/catch catch an error thrown later inside setTimeout?

Not with an outer synchronous try/catch around the timer registration.

### Why rethrow?

To add local logging/context while allowing a higher-level caller to handle the failure.

---

## Key Takeaways

- `try` contains risky code.
- `catch` handles thrown errors.
- `finally` runs for cleanup either way.
- A throw exits the remaining `try` block immediately.
- Catch only errors you can handle meaningfully.
- Rethrow when higher layers should decide.
- Avoid returning/throwing from `finally` unless intentional.
- Async failures require Promise/await-aware handling.
