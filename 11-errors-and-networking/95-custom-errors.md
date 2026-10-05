# Lesson 95 — Throwing and Creating Custom Errors

Handling errors well starts with throwing meaningful errors.

JavaScript lets you throw any value, but best practice is usually to throw an `Error` object or an Error subclass.

---

## 1. throw

Basic:

```js
throw new Error(
  "Something failed"
);
```

Once thrown, normal execution stops until a handler is found.

---

## 2. Do Not Usually Throw Strings

Possible:

```js
throw "Failed";
```

But better:

```js
throw new Error(
  "Failed"
);
```

Why?

Error objects provide:

- name
- message
- stack
- subclassing
- cause support

---

## 3. Validation Example

```js
function createUser(
  name
) {
  if (!name) {
    throw new Error(
      "Name is required"
    );
  }

  return {
    name,
  };
}
```

The function enforces its contract.

---

## 4. Built-In Specialized Errors

Use an appropriate built-in error when it matches the problem.

Example:

```js
function divide(
  a,
  b
) {
  if (
    typeof a !==
      "number" ||
    typeof b !==
      "number"
  ) {
    throw new TypeError(
      "Arguments must be numbers"
    );
  }

  return a / b;
}
```

---

## 5. RangeError Example

```js
function setPercentage(
  value
) {
  if (
    value < 0 ||
    value > 100
  ) {
    throw new RangeError(
      "Percentage must be 0-100"
    );
  }
}
```

This communicates that the type may be valid but the value is outside the allowed range.

---

## 6. Custom Error Class

```js
class ValidationError
  extends Error {
  constructor(message) {
    super(message);

    this.name =
      "ValidationError";
  }
}
```

Use:

```js
throw new ValidationError(
  "Email is invalid"
);
```

---

## 7. Why super(message)?

`Error` is the parent class.

```js
super(message);
```

initializes the base Error behavior, including the message and stack behavior provided by the runtime.

---

## 8. instanceof

```js
try {
  throw new ValidationError(
    "Invalid data"
  );
} catch (error) {
  if (
    error instanceof
      ValidationError
  ) {
    console.log(
      "Validation issue"
    );
  }
}
```

Custom classes allow error-type-based handling.

---

## 9. Add Extra Context

```js
class HttpError
  extends Error {
  constructor(
    message,
    status
  ) {
    super(message);

    this.name =
      "HttpError";

    this.status =
      status;
  }
}
```

Use:

```js
throw new HttpError(
  "Not found",
  404
);
```

---

## 10. Domain-Specific Errors

Example:

```js
class InsufficientStockError
  extends Error {
  constructor(productId) {
    super(
      `Insufficient stock for product ${productId}`
    );

    this.name =
      "InsufficientStockError";

    this.productId =
      productId;
  }
}
```

This is more meaningful than:

```js
throw new Error(
  "Something went wrong"
);
```

---

## 11. Error cause

Modern JavaScript supports an error `cause`.

```js
try {
  await databaseQuery();
} catch (error) {
  throw new Error(
    "Unable to load user",
    {
      cause: error,
    }
  );
}
```

This preserves lower-level failure context.

---

## 12. Why Wrap Errors?

Suppose a database library throws:

```text
ECONNRESET
```

Your service may add business meaning:

```text
Unable to load profile
```

while preserving:

```js
error.cause
```

Layered error context is easier to debug.

---

## 13. Avoid Losing the Original Error

Bad:

```js
catch {
  throw new Error(
    "Failed"
  );
}
```

You lose the original details.

Better:

```js
catch (error) {
  throw new Error(
    "Failed to save order",
    {
      cause: error,
    }
  );
}
```

---

## 14. Error Codes

Sometimes a stable machine-readable code is useful.

```js
class AppError
  extends Error {
  constructor(
    message,
    code
  ) {
    super(message);

    this.name =
      "AppError";

    this.code =
      code;
  }
}
```

Example:

```js
throw new AppError(
  "Email already exists",
  "EMAIL_EXISTS"
);
```

Use message for humans, code for program logic.

---

## 15. Do Not Branch on Error Message Text

Fragile:

```js
if (
  error.message ===
  "User not found"
) {
}
```

Better:

```js
if (
  error instanceof
    UserNotFoundError
) {
}
```

or use a stable code.

Messages can change.

---

## 16. Operational vs Programmer Errors

Useful distinction:

### Operational error

Expected failure condition:

- network unavailable
- validation failed
- file missing
- request timed out

### Programmer error

Bug:

- reading property of `undefined`
- impossible state
- broken invariant

Do not blindly recover from programmer bugs as if they were normal business failures.

---

## 17. Fail Fast

If a function receives invalid required input:

```js
function processOrder(
  order
) {
  if (!order) {
    throw new TypeError(
      "order is required"
    );
  }

  // continue
}
```

Failing early prevents corrupted state later.

---

## 18. Custom Errors for API Layers

Example:

```js
class ApiError
  extends Error {
  constructor(
    message,
    {
      status,
      code,
      details,
      cause,
    } = {}
  ) {
    super(
      message,
      {
        cause,
      }
    );

    this.name =
      "ApiError";

    this.status =
      status;

    this.code =
      code;

    this.details =
      details;
  }
}
```

This gives your application a consistent error shape.

---

## 19. Error Serialization Caution

Native Error properties such as `message` and `stack` are not always serialized the way you expect with:

```js
JSON.stringify(error);
```

If sending an error response, create an explicit safe object:

```js
{
  code:
    error.code,
  message:
    error.message,
}
```

Do not expose internal stack traces to clients in production.

---

## 20. Security Consideration

Avoid leaking:

- database details
- stack traces
- filesystem paths
- secrets
- internal service names

User-facing errors should be useful but safe.

Internal logs can contain more diagnostic context.

---

## 21. React / Next.js Connection

An API/service layer might throw:

```js
throw new ApiError(
  "Unable to load jobs",
  {
    status: 503,
    code:
      "SERVICE_UNAVAILABLE",
  }
);
```

Then a UI or route boundary decides how to present it.

This keeps networking logic separate from presentation logic.

---

## Interview Questions

### Why throw Error objects instead of strings?

They provide structured metadata, stack traces, subclassing, and cause chaining.

### Why create custom errors?

To represent domain-specific failure types and attach structured context.

### What is error.cause?

A way to preserve the lower-level error when wrapping it with higher-level context.

### Why avoid matching on error.message?

Messages are unstable human-readable text; use error types or codes for program logic.

---

## Key Takeaways

- Prefer throwing Error objects.
- Use built-in error subclasses when appropriate.
- Create custom Error subclasses for domain-specific failures.
- Use stable codes/types for machine handling.
- Preserve original failures with `cause`.
- Do not leak sensitive internal error details.
- Throw errors at the layer where the failure becomes meaningful.
