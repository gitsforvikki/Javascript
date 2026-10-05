# Lesson 97 — HTTP Request Lifecycle and API Error Handling

Reliable networking code starts with understanding **where** a request can fail.

A useful mental model is:

```text
application code
↓
request construction
↓
browser/runtime
↓
DNS / connection / TLS
↓
HTTP request
↓
server
↓
HTTP response
↓
body parsing
↓
business validation
↓
UI/application handling
```

Different failures happen at different layers.

---

## 1. The HTTP Request Lifecycle

At a simplified level:

```text
1. Build request
2. Resolve destination
3. Establish connection
4. Send HTTP request
5. Server processes request
6. Server sends HTTP response
7. Client receives response
8. Client reads/parses body
9. Application interprets result
```

A failure can occur at any step.

---

## 2. Transport / Network Failure

Examples:

- DNS failure
- connection refused
- connection reset
- device offline
- TLS negotiation problem
- aborted request

In Fetch, these can cause the Promise to reject.

Example:

```js
try {
  await fetch(
    "https://invalid.example"
  );
} catch (error) {
  console.error(
    "Network/request failure",
    error
  );
}
```

---

## 3. HTTP Error Response

Suppose server responds:

```text
404 Not Found
```

or:

```text
500 Internal Server Error
```

Fetch usually still fulfills with a `Response`.

So this:

```js
const response =
  await fetch(url);
```

can succeed at the transport layer while still representing an HTTP failure.

---

## 4. Transport Success vs Application Success

Very important distinction:

```text
fetch fulfilled
≠
request succeeded for business purposes
```

Example:

```js
const response =
  await fetch(
    "/api/users/999"
  );

console.log(
  response.status
);
```

Could be:

```text
404
```

The network worked.

The resource did not exist.

---

## 5. Status Code Groups

High-level categories:

```text
1xx
→ informational

2xx
→ success

3xx
→ redirection

4xx
→ client/request-side problem

5xx
→ server-side problem
```

These categories guide error handling, but individual status semantics matter.

---

## 6. Common 2xx Statuses

Examples:

```text
200 OK
201 Created
202 Accepted
204 No Content
```

Important:

`202 Accepted` may mean work has only been accepted for later processing.

`204 No Content` normally has no response body to parse.

---

## 7. Common 4xx Statuses

Examples:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
422 Unprocessable Content
429 Too Many Requests
```

These are not all handled the same way.

---

## 8. 401 vs 403

Useful interview distinction:

```text
401
→ authentication is missing/invalid

403
→ identity may be known,
  but access is not allowed
```

Exact API usage can vary, but this is the standard conceptual distinction.

---

## 9. 400 vs 422

Common API pattern:

```text
400
→ malformed/invalid request in a broad sense

422
→ request syntax understood,
  but semantic validation failed
```

APIs differ in how they choose between them.

Your client should follow the documented contract rather than hard-coding assumptions from one backend.

---

## 10. 409 Conflict

Typical uses:

- duplicate resource
- version conflict
- state conflict

Example:

```text
email already exists
```

could be modeled as a 409 by some APIs.

---

## 11. 429 Too Many Requests

This means rate limiting.

A server may include:

```http
Retry-After: 30
```

A resilient client may respect that information before retrying.

This connects directly to Lesson 99.

---

## 12. Common 5xx Statuses

Examples:

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

These often represent transient server/infrastructure failures, but not always.

Retry decisions should still be intentional.

---

## 13. Parse Errors

Suppose:

```js
const response =
  await fetch(url);

const data =
  await response.json();
```

If the body is not valid JSON, `response.json()` rejects.

So:

```text
HTTP request succeeded
↓
response received
↓
body parsing failed
```

This is a different failure category from network failure or HTTP status failure.

---

## 14. Business / Domain Errors

A server may return:

```http
200 OK
```

with:

```json
{
  "success": false,
  "code": "INSUFFICIENT_STOCK"
}
```

Whether that design is ideal depends on the API.

But the important lesson is:

> HTTP status and business result are related but not identical concepts.

The client must understand the API contract.

---

## 15. Recommended Error Layers

Think in layers:

```text
NetworkError
→ request never completed normally

HttpError
→ response status not acceptable

ParseError
→ body could not be decoded

DomainError
→ response is technically valid,
  but business rule failed
```

This separation makes debugging much easier.

---

## 16. Structured ApiError

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

---

## 17. Parse Error Body Carefully

A failed response may return:

- JSON
- text
- HTML
- empty body

Do not assume all error responses are JSON.

Example helper:

```js
async function readBody(
  response
) {
  const contentType =
    response.headers.get(
      "content-type"
    ) ?? "";

  if (
    contentType.includes(
      "application/json"
    )
  ) {
    return response.json();
  }

  return response.text();
}
```

---

## 18. Build a Better Request Helper

```js
async function request(
  url,
  options
) {
  let response;

  try {
    response =
      await fetch(
        url,
        options
      );
  } catch (error) {
    throw new ApiError(
      "Network request failed",
      {
        code:
          "NETWORK_ERROR",
        cause:
          error,
      }
    );
  }

  let body;

  try {
    body =
      await readBody(
        response
      );
  } catch (error) {
    throw new ApiError(
      "Unable to parse response",
      {
        status:
          response.status,
        code:
          "INVALID_RESPONSE",
        cause:
          error,
      }
    );
  }

  if (!response.ok) {
    throw new ApiError(
      "API request failed",
      {
        status:
          response.status,
        code:
          body?.code,
        details:
          body,
      }
    );
  }

  return body;
}
```

This separates:

- network failure
- parse failure
- HTTP failure

---

## 19. Do Not Show Raw Server Error Directly to Users

Server may return technical details.

Instead:

```js
if (
  error.code ===
  "EMAIL_EXISTS"
) {
  showMessage(
    "This email is already registered."
  );
}
```

Map internal failures to safe user-facing messages.

---

## 20. Logging vs User Messaging

Different audiences:

```text
Logs
→ technical context

User message
→ actionable, safe explanation
```

Example:

Internal:

```text
POST /orders failed with 503
requestId=abc123
```

User:

```text
We couldn't place your order right now. Please try again.
```

---

## 21. Correlation / Request IDs

Servers often send an identifier:

```text
request-id
trace-id
correlation-id
```

If available, include it in diagnostics.

Example:

```js
const requestId =
  response.headers.get(
    "x-request-id"
  );
```

This helps backend/frontend teams trace one failed request across systems.

---

## 22. Client Validation Is Not Enough

You may validate before sending:

```js
if (!email.includes("@")) {
  // ...
}
```

But the server must still validate.

Why?

Client code can be bypassed.

Server validation remains authoritative.

---

## 23. Error Handling by Status

Example:

```js
if (
  error.status === 401
) {
  redirectToLogin();
}

if (
  error.status === 403
) {
  showForbidden();
}

if (
  error.status === 404
) {
  showNotFound();
}
```

Do this when the status has meaningful product behavior.

Do not build giant status switches when a simpler generic boundary is enough.

---

## 24. 500 Does Not Reveal the Root Cause

A client seeing:

```text
500
```

cannot know whether the server failure was:

- database
- bug
- dependency outage
- configuration
- internal timeout

The client should treat the server's public contract as authoritative and avoid guessing.

---

## 25. HTTP Idempotency Preview

Retry behavior depends heavily on whether repeating the request is safe.

Examples:

```text
GET
→ normally idempotent

PUT
→ designed to be idempotent

DELETE
→ designed to be idempotent semantically

POST
→ not inherently idempotent
```

But real API behavior matters.

This becomes critical in Lesson 99.

---

## 26. React / Next.js Connection

In application code:

```js
try {
  const jobs =
    await request(
      "/api/jobs"
    );

  setJobs(jobs);
} catch (error) {
  if (
    error.status === 401
  ) {
    // auth handling
  }

  setError(
    "Unable to load jobs"
  );
}
```

Keep raw networking details out of UI components when possible.

---

## Interview Questions

### Does a fulfilled fetch Promise mean HTTP success?

No. It means an HTTP response was received; status may still be 4xx/5xx.

### Network error vs HTTP error?

A network error prevents normal response completion. An HTTP error is still a valid HTTP response with an undesirable status.

### Why separate parse errors?

Because a valid HTTP response can still contain malformed/unexpected body content.

### 401 vs 403?

401 generally means authentication is required/invalid; 403 means access is forbidden despite the request being understood.

---

## Key Takeaways

- Network success and application success are different.
- Classify errors by layer.
- Fetch generally fulfills for 4xx/5xx responses.
- Parse errors are separate from HTTP status errors.
- Business/domain failures need explicit modeling.
- Use structured error types/codes.
- Keep user messages safe and actionable.
- Server validation is authoritative.
- Retry decisions depend on status and idempotency.
