# Lesson 96 — Fetch API

The Fetch API is the modern browser API for making HTTP requests.

Basic form:

```js
const response =
  await fetch(url);
```

Important:

> `fetch()` returns a Promise of a `Response` object.

It does **not** directly return parsed JSON.

---

## 1. Basic GET Request

```js
const response =
  await fetch(
    "/api/users"
  );
```

Then parse JSON:

```js
const users =
  await response.json();
```

Two async steps:

```text
fetch()
↓
Response

response.json()
↓
parsed JavaScript value
```

---

## 2. Why response.json() Is Async

The response body may arrive as a stream.

Parsing the body is a separate asynchronous operation.

So:

```js
response.json()
```

returns a Promise.

---

## 3. Response Object

Useful properties:

```js
response.ok
response.status
response.statusText
response.headers
response.url
response.redirected
```

---

## 4. response.ok

`response.ok` is true for successful 2xx status codes.

Example:

```js
if (!response.ok) {
  throw new Error(
    `HTTP ${response.status}`
  );
}
```

This is essential because Fetch does not reject merely because the server returned 404 or 500.

---

## 5. Important Fetch Error Rule

`fetch()` generally rejects for request/network-level failures, such as:

- network failure
- invalid request setup
- aborted request

It usually fulfills normally for HTTP responses like:

- 400
- 404
- 500

So:

```text
HTTP error response
≠
Promise rejection automatically
```

You must inspect the response.

---

## 6. Safe Basic Helper

```js
async function fetchJson(
  url
) {
  const response =
    await fetch(url);

  if (!response.ok) {
    throw new Error(
      `Request failed: ${response.status}`
    );
  }

  return response.json();
}
```

Usage:

```js
const users =
  await fetchJson(
    "/api/users"
  );
```

---

## 7. Request Options

```js
fetch(
  "/api/users",
  {
    method: "POST",
    headers: {
      "Content-Type":
        "application/json",
    },
    body:
      JSON.stringify({
        name: "Vikash",
      }),
  }
);
```

---

## 8. HTTP Methods

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
```

Typical intent:

```text
GET
→ read

POST
→ create/action

PUT
→ replace

PATCH
→ partial update

DELETE
→ remove
```

The server defines actual behavior.

---

## 9. Sending JSON

```js
const response =
  await fetch(
    "/api/jobs",
    {
      method: "POST",

      headers: {
        "Content-Type":
          "application/json",
      },

      body:
        JSON.stringify({
          title:
            "React Developer",
        }),
    }
  );
```

`body` must be a supported request-body type.

For JSON, stringify the object.

---

## 10. Reading JSON Safely

Do not assume every response has JSON.

If server returns 204 No Content, calling:

```js
response.json()
```

may fail.

Your client should know the API contract.

Example:

```js
if (
  response.status ===
  204
) {
  return null;
}
```

---

## 11. Other Body Readers

Response provides methods such as:

```js
response.text()
response.blob()
response.arrayBuffer()
response.formData()
```

Choose based on content type.

---

## 12. Body Can Usually Be Consumed Once

A response body is stream-based.

After:

```js
await response.json();
```

you normally cannot consume the same body again.

If you truly need a duplicate, `response.clone()` can clone the response before body consumption.

---

## 13. Headers

Read:

```js
response.headers.get(
  "content-type"
);
```

Send custom header:

```js
fetch(url, {
  headers: {
    Authorization:
      `Bearer ${token}`,
  },
});
```

Be careful not to expose sensitive tokens where JavaScript should not have access to them.

---

## 14. FormData

```js
const formData =
  new FormData();

formData.append(
  "name",
  "Vikash"
);

formData.append(
  "avatar",
  file
);
```

Then:

```js
await fetch(
  "/api/profile",
  {
    method: "POST",
    body: formData,
  }
);
```

Important:

Do not manually set the multipart Content-Type boundary when sending a native `FormData`; the browser handles it.

---

## 15. URLSearchParams

For URL-encoded form-style data:

```js
const params =
  new URLSearchParams({
    q: "react",
    page: "1",
  });
```

Can be useful in query strings or compatible request bodies.

---

## 16. Query Parameters

```js
const url =
  new URL(
    "/api/jobs",
    window.location.origin
  );

url.searchParams.set(
  "page",
  "2"
);

url.searchParams.set(
  "role",
  "react"
);

const response =
  await fetch(url);
```

This is safer than manual string concatenation.

---

## 17. Cookies and Credentials

Same-origin cookies are handled according to Fetch/browser credential rules.

For certain cross-origin credentialed requests:

```js
fetch(
  "https://api.example.com/user",
  {
    credentials:
      "include",
  }
);
```

The server must also allow credentialed CORS requests correctly.

---

## 18. credentials Values

Common:

```text
omit
same-origin
include
```

Use the setting that matches your authentication architecture.

Do not assume `include` fixes incorrect cookie/CORS configuration.

---

## 19. CORS Is Browser Security Policy

A request can reach a server but still be blocked from being exposed to JavaScript because of CORS policy.

CORS is enforced by browsers.

It is not an authentication system.

It controls whether JavaScript from one origin can access responses from another origin.

---

## 20. mode: "no-cors" Is Not a General CORS Fix

Avoid advice like:

```js
fetch(url, {
  mode: "no-cors",
});
```

as a way to bypass CORS.

That produces an opaque response with severe restrictions and does not grant normal access to the response body/status.

Correct CORS configuration belongs on the server.

---

## 21. Redirects

Fetch typically follows redirects automatically for many requests.

You can inspect:

```js
response.redirected
response.url
```

Redirect handling can be configured, but normal application code often uses defaults.

---

## 22. Cache Option

Fetch supports request cache modes:

```js
fetch(url, {
  cache: "no-store",
});
```

Browser Fetch cache options are not identical to framework-level caching such as Next.js server caching.

Do not mix the two mental models.

---

## 23. Referrer and Other Advanced Options

Fetch supports many options:

- referrer
- referrerPolicy
- integrity
- keepalive
- signal

You do not need every option for daily use.

The most important practical ones are:

- method
- headers
- body
- credentials
- signal

---

## 24. A More Robust JSON Helper

```js
async function fetchJson(
  url,
  options
) {
  const response =
    await fetch(
      url,
      options
    );

  const contentType =
    response.headers.get(
      "content-type"
    );

  const isJson =
    contentType?.includes(
      "application/json"
    );

  const body =
    isJson
      ? await response.json()
      : await response.text();

  if (!response.ok) {
    throw new Error(
      `HTTP ${response.status}`
    );
  }

  return body;
}
```

In Lesson 97 we will make this error handling much more structured.

---

## 25. Fetch and Promise Chaining

```js
fetch("/api/users")
  .then((response) => {
    if (!response.ok) {
      throw new Error(
        "Request failed"
      );
    }

    return response.json();
  })
  .then((users) => {
    console.log(users);
  })
  .catch((error) => {
    console.error(error);
  });
```

Async/await is often easier to read, but both use the same Promise semantics.

---

## 26. React Connection

Client-side:

```js
useEffect(() => {
  async function load() {
    const response =
      await fetch(
        "/api/jobs"
      );

    // ...
  }

  load();
}, []);
```

But modern React/Next.js often provides better server-side or data-library patterns than manually fetching everything in effects.

Still, Fetch fundamentals remain essential.

---

## 27. Next.js Connection

Next.js extends server-side `fetch` behavior with framework caching/revalidation semantics depending on version/runtime.

But the underlying concepts still matter:

- HTTP method
- status
- headers
- body
- cancellation
- network failure

Keep browser Fetch semantics separate from Next.js framework caching behavior.

---

## 28. Common Mistakes

### Mistake 1

Assuming 404/500 automatically rejects Fetch.

### Mistake 2

Forgetting `response.json()` is async.

### Mistake 3

Calling `response.json()` twice.

### Mistake 4

Setting multipart Content-Type manually for FormData.

### Mistake 5

Using `no-cors` as a CORS workaround.

---

## Interview Questions

### What does fetch return?

A Promise that fulfills with a `Response` object when an HTTP response is available, or rejects for certain request/network failures.

### Does fetch reject on HTTP 404?

Usually no.

### Why do we check response.ok?

To treat unwanted HTTP status codes as application errors.

### Why is response.json async?

Because reading/parsing the response body is a separate asynchronous operation.

---

## Key Takeaways

- `fetch()` returns a Promise of `Response`.
- HTTP error statuses do not automatically reject.
- Check `response.ok` or status explicitly.
- Response body parsing is asynchronous.
- Use correct method, headers, and body.
- Use FormData for multipart uploads.
- CORS cannot be bypassed from frontend JavaScript.
- Fetch is Promise-based and integrates naturally with async/await.
