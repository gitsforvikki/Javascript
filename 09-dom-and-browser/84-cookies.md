# Lesson 84 — Cookies Basics

Cookies are small pieces of data associated with websites.

Unlike `localStorage`, cookies can participate directly in HTTP communication because the browser can automatically include matching cookies with requests.

This makes cookies especially important for:

- sessions
- authentication
- preferences
- server/client state coordination

---

## 1. Basic Cookie Mental Model

Server response:

```http
Set-Cookie: session=abc123
```

Browser stores the cookie.

Later matching request:

```http
Cookie: session=abc123
```

The browser can attach the cookie automatically according to cookie rules.

This is fundamentally different from Web Storage.

---

# 2. Creating a Cookie from JavaScript

Basic:

```js
document.cookie =
  "theme=dark";
```

This does not replace all cookies.

It sets/updates one cookie.

Read:

```js
console.log(
  document.cookie
);
```

The result is a semicolon-separated string containing JavaScript-visible cookies.

---

# 3. document.cookie Is Unusual

Unlike a normal property:

```js
document.cookie =
  "theme=dark";
```

does not simply overwrite one big cookie string.

The browser interprets the assignment as a cookie-setting instruction.

Reading returns available cookies.

---

# 4. Cookie Name and Value

Basic format:

```text
name=value
```

Example:

```js
document.cookie =
  "language=en";
```

Cookie values should be encoded when they may contain special characters.

Example:

```js
document.cookie =
  `name=${encodeURIComponent(
    "Vikash Kumar"
  )}`;
```

---

# 5. Cookie Attributes

Cookies can include attributes controlling:

- lifetime
- path
- domain
- transport security
- cross-site behavior
- JavaScript accessibility

Important attributes include:

```text
Expires
Max-Age
Path
Domain
Secure
HttpOnly
SameSite
```

---

# 6. Session Cookie

A cookie without an explicit persistent expiry is often treated as a session cookie.

Example:

```js
document.cookie =
  "theme=dark; Path=/";
```

Its lifetime depends on browser session behavior.

Do not confuse this with `sessionStorage`; they are completely different systems.

---

# 7. Max-Age

Example:

```js
document.cookie =
  "theme=dark; Max-Age=3600; Path=/";
```

This asks for a lifetime of 3600 seconds.

---

# 8. Expires

Example conceptually:

```js
document.cookie =
  "theme=dark; Expires=Wed, 21 Oct 2026 07:28:00 GMT; Path=/";
```

`Expires` uses an absolute date/time.

`Max-Age` uses a relative number of seconds.

---

# 9. Path

```text
Path=/
```

controls which request paths can receive the cookie.

Example:

```text
Path=/admin
```

limits normal cookie inclusion to matching path scope.

For site-wide cookies, `Path=/` is common.

---

# 10. Domain

The `Domain` attribute controls host scope.

Modern host-only cookies can be created by omitting `Domain`.

Setting a domain can allow applicable subdomains to receive the cookie.

Cookie domain rules are security-sensitive, so avoid broad scopes without a reason.

---

# 11. Secure

```text
Secure
```

means the cookie should be sent only over secure HTTPS connections, subject to browser rules.

Example server header:

```http
Set-Cookie: session=abc123; Secure
```

Sensitive production cookies should normally use HTTPS and `Secure`.

---

# 12. HttpOnly

```text
HttpOnly
```

prevents client-side JavaScript from reading the cookie through:

```js
document.cookie
```

This is especially valuable for sensitive session/authentication cookies.

Important:

> JavaScript cannot create an HttpOnly cookie.

It must be set through an HTTP response/server mechanism.

---

# 13. Why HttpOnly Matters

If an authentication cookie is JavaScript-readable, successful XSS code may steal it.

With `HttpOnly`:

```text
browser can send cookie
↓
JavaScript cannot directly read it
```

This does not solve XSS completely, but it reduces token-exfiltration risk.

---

# 14. SameSite

`SameSite` controls how cookies behave in cross-site request contexts.

Common values:

```text
Strict
Lax
None
```

At a high level:

```text
Strict
→ strongest cross-site restriction

Lax
→ allows some top-level navigation cases

None
→ allows cross-site sending
  and requires Secure in modern browsers
```

Exact browser rules are nuanced, but this mental model is enough for fundamentals.

---

# 15. Cookies and CSRF

Because browsers can automatically attach cookies to requests, cookie-based authentication can be exposed to CSRF risks.

Mitigations can include:

- SameSite settings
- CSRF tokens
- origin checks
- correct request design

Cookie security should be designed intentionally.

---

# 16. Cookies and XSS

Two different security concerns:

```text
XSS
→ attacker runs JS in your origin

CSRF
→ attacker causes browser to send authenticated request
```

`HttpOnly` helps protect sensitive cookie values from direct JavaScript access.

`SameSite` can help reduce certain CSRF scenarios.

They solve different problems.

---

# 17. Reading document.cookie

```js
console.log(
  document.cookie
);
```

Example:

```text
theme=dark; language=en
```

It is not automatically parsed into an object.

You must parse it yourself or use a library/helper.

---

# 18. Simple Cookie Parser

```js
function getCookies() {
  return document.cookie
    .split("; ")
    .filter(Boolean)
    .reduce(
      (
        result,
        item
      ) => {
        const [
          rawKey,
          ...rest
        ] =
          item.split("=");

        result[rawKey] =
          decodeURIComponent(
            rest.join("=")
          );

        return result;
      },
      {}
    );
}
```

This is a learning example, not a full RFC-complete cookie library.

Cookie parsing has edge cases.

---

# 19. Deleting a Cookie

There is no direct `deleteCookie()` browser API.

You typically expire it:

```js
document.cookie =
  "theme=; Max-Age=0; Path=/";
```

Important:

The path/domain scope should match the cookie you intend to delete.

Otherwise a similarly named cookie may remain.

---

# 20. Cookies Are Sent Per Matching Request

Cookies are not simply a global browser bag.

Whether a cookie is included depends on attributes such as:

- domain
- path
- secure context
- SameSite rules
- expiry

The browser applies these rules for each request.

---

# 21. Cookie Size Is Small

Cookies are intended for small pieces of data.

Browsers impose per-cookie and total limits.

Do not store large application data in cookies.

Every matching cookie can also add bytes to HTTP requests.

---

# 22. Web Storage vs Cookies

```text
localStorage
→ client-side storage
→ JS readable
→ not automatically sent to server
→ larger than typical cookie capacity

sessionStorage
→ tab/session-scoped client storage
→ JS readable
→ not automatically sent

cookies
→ small request-aware storage
→ can be automatically sent
→ can use HttpOnly/Secure/SameSite
```

---

# 23. Authentication Example

Server might set:

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/
```

Then browser requests can automatically include the session cookie.

Client JavaScript does not need to manually read and attach it.

---

# 24. fetch and Credentials

Cross-origin cookie behavior with `fetch()` depends on origin/CORS/credentials rules.

Example:

```js
fetch(
  "https://api.example.com/user",
  {
    credentials:
      "include",
  }
);
```

For cross-origin requests, server CORS configuration must also permit credentials.

Same-origin fetch behavior differs.

This is an important practical area, but full CORS is beyond this cookie-basics lesson.

---

# 25. Your JWT Cookie Connection

A common authentication design:

```text
server authenticates user
↓
server sets JWT/session cookie
↓
cookie uses HttpOnly
↓
browser sends cookie automatically
↓
server validates request
```

This is why backend code often configures cookie flags such as:

```text
httpOnly
secure
sameSite
```

These flags directly affect security and request behavior.

---

# 26. Next.js Connection

Server-side code can set cookies through framework/server APIs.

In modern Next.js App Router, cookies are commonly handled server-side rather than manually through `document.cookie` for sensitive authentication flows.

The exact API is framework-version dependent, but the underlying browser cookie rules remain the same.

---

# 27. Cookie Preferences Example

For a non-sensitive preference:

```js
document.cookie =
  "language=en; Max-Age=2592000; Path=/; SameSite=Lax";
```

This can be reasonable when the server also needs the preference.

If only client JavaScript needs it, Web Storage may be simpler.

---

# 28. When to Choose Which Storage

Ask:

```text
Does server need value
automatically with requests?
↓
cookie may be appropriate

Client-only preference?
↓
localStorage may be simpler

Only this tab/session?
↓
sessionStorage may fit
```

Then evaluate security requirements.

---

# 29. Common Mistakes

### Mistake 1

Storing large JSON blobs in cookies.

### Mistake 2

Thinking `HttpOnly` can be set from JavaScript.

### Mistake 3

Assuming `stopPropagation()` or other DOM behavior affects cookies.

### Mistake 4

Forgetting cookie path/domain when deleting.

### Mistake 5

Using `SameSite=None` without `Secure` in modern browsers.

### Mistake 6

Treating cookies and localStorage as interchangeable.

---

# 30. Interview Questions

### Cookies vs localStorage?

Cookies can be automatically included with matching HTTP requests and support security attributes. `localStorage` is JavaScript-controlled client storage and is not automatically sent.

### What does HttpOnly do?

It prevents client-side JavaScript from reading the cookie.

### What does Secure do?

It restricts cookie transmission to secure HTTPS contexts according to browser rules.

### What does SameSite control?

It controls cookie inclusion in cross-site request contexts.

### Can JavaScript set an HttpOnly cookie?

No.

---

# Section 9 Complete Mental Model

```text
HTML
↓
DOM tree
↓
select/manipulate nodes
↓
create/update/remove nodes
↓
browser events
↓
capture + target + bubble
↓
event delegation
↓
default action vs propagation
↓
client storage
├── localStorage
├── sessionStorage
└── cookies
```

---

# Key Takeaways

- Cookies are small pieces of browser-managed data.
- Matching cookies can be sent automatically with HTTP requests.
- Cookie attributes control lifetime and scope.
- `HttpOnly` prevents JavaScript access.
- `Secure` protects transport usage.
- `SameSite` affects cross-site behavior.
- Cookies can introduce CSRF considerations because they are automatically attached.
- Cookies are much smaller than Web Storage and should not hold large data.
- Sensitive authentication cookies are usually set server-side.
- Cookies and Web Storage serve different purposes.
