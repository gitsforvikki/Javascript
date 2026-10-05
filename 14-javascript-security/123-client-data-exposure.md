# Lesson 123 — Client-Side Data Exposure

Anything delivered to the browser should be treated as visible to the user.

The core rule is:

> If data reaches client-side JavaScript, HTML, network responses, source maps, or browser storage, a determined user can inspect it.

Frontend code cannot safely hide secrets from the browser owner.

---

## 1. Client Code Is Observable

Users can inspect:

- JavaScript bundles
- HTML
- API responses
- request headers
- browser storage
- network traffic visible to DevTools
- source maps when published
- runtime variables

Obfuscation does not create a real security boundary.

---

## 2. Never Put Secrets in Frontend Bundles

Do not expose:

- database passwords
- private API keys
- signing secrets
- webhook secrets
- service-account credentials
- private encryption keys

If the browser needs the value to run, the user can ultimately inspect it.

---

## 3. Public vs Secret API Keys

Some API keys are intentionally public identifiers.

Examples may include:

- publishable payment keys
- analytics IDs
- public project identifiers

A public key should have limited permissions.

Do not assume the word "key" automatically means secret.

The provider's security model determines this.

---

## 4. Environment Variables Do Not Automatically Stay Secret

A common misconception:

```text
.env
=
secret forever
```

If a build system injects a value into client-side code, it becomes visible in the bundle.

Frameworks usually distinguish:

- server-only environment variables
- explicitly public/client-exposed variables

Follow the framework's rules.

---

## 5. Next.js Environment Variables

In Next.js, variables intentionally exposed to browser code typically use a public prefix such as:

```text
NEXT_PUBLIC_
```

Those values should be considered public.

Server secrets must remain in server-only execution paths.

---

## 6. Do Not Import Server Secrets into Client Components

A Client Component should never depend on secret server-only values.

Security boundary:

```text
server
→ may hold secrets

browser
→ must not receive secrets
```

If the browser needs an action, call a server endpoint/action that uses the secret internally.

---

## 7. API Responses

Do not return sensitive fields just because the UI does not display them.

Bad response:

```json
{
  "id": 10,
  "name": "User",
  "passwordHash": "...",
  "internalNotes": "..."
}
```

Even hidden fields are visible in DevTools.

Return only what the client needs.

---

## 8. Overfetching Is a Security Concern

Overfetching increases exposure.

Use explicit response DTOs/view models:

```js
return {
  id:
    user.id,
  name:
    user.name,
  avatar:
    user.avatar,
};
```

Do not serialize entire database records by default.

---

## 9. Hidden UI Is Not Authorization

Bad assumption:

```jsx
{isAdmin && (
  <button>
    Delete User
  </button>
)}
```

Hiding the button is useful UX.

But users can still call APIs manually.

Real authorization must happen on the server.

---

## 10. Client-Side Role Values Are Not Trustworthy

A browser value such as:

```js
localStorage.setItem(
  "role",
  "admin"
);
```

cannot prove authorization.

Users control their browser.

Server must verify identity and permissions independently.

---

## 11. JWT Contents Are Often Readable

Many JWTs are:

```text
base64url encoded
not encrypted
```

So payload claims can often be decoded by anyone holding the token.

Do not put secrets inside JWT payloads unless the token format/encryption specifically guarantees confidentiality.

---

## 12. Sensitive Data in URLs

Avoid placing sensitive values in query strings:

```text
https://example.com/reset?secret=...
```

URLs may appear in:

- browser history
- logs
- analytics
- referrer information
- screenshots

Some token-based flows necessarily use URLs, but tokens should be short-lived, scoped, and carefully handled.

---

## 13. Console Logs

Avoid logging secrets:

```js
console.log(
  token
);
```

Production logs can leak:

- tokens
- user data
- internal IDs
- sensitive payloads

Log intentionally.

---

## 14. Error Messages

Do not expose internal details to the browser.

Bad:

```json
{
  "error":
    "Database password invalid at postgres://..."
}
```

Better:

```json
{
  "error":
    "Unable to process request"
}
```

Keep detailed diagnostics server-side.

---

## 15. Source Maps

Source maps improve debugging but may reveal:

- original source structure
- comments
- internal filenames
- implementation details

They are not normally secrets by themselves, but decide intentionally whether production source maps should be publicly downloadable.

Never place secrets in source code regardless.

---

## 16. Browser Storage

Values in:

- localStorage
- sessionStorage
- IndexedDB
- JavaScript-readable cookies

are visible to code running in that origin and to the browser user.

Treat them as client data, not secret vaults.

---

## 17. Cached Responses

Sensitive API responses may be cached by:

- browser
- CDN
- service worker
- framework

Use correct caching headers/policies for private data.

Do not accidentally cache one user's private response for another user.

---

## 18. Third-Party Scripts

Any third-party JavaScript running with your page's origin privileges may access significant client-side data.

Limit:

- unnecessary scripts
- broad data exposure
- untrusted dependencies

Use CSP and integrity controls where appropriate.

---

## 19. postMessage Exposure

When sending data across windows/iframes:

```js
otherWindow.postMessage(
  data,
  targetOrigin
);
```

use a specific `targetOrigin`.

Avoid:

```text
*
```

for sensitive messages unless truly necessary.

Receivers should validate `event.origin`.

---

## 20. Feature Flags

Frontend feature flags are not secrets.

If a feature flag is shipped to the browser, users can inspect it.

Do not use client flags as authorization controls.

---

## 21. Payment Integration Connection

Public payment SDK keys may be exposed intentionally.

But secrets used for:

- payment verification
- webhook signature validation
- server API authentication

must remain server-side.

The browser initiates payment; the server verifies trusted results.

---

## 22. CORS Does Not Hide Data from Your Own Frontend User

CORS controls cross-origin browser access.

It does not prevent the current user from inspecting responses their own browser receives.

CORS is not a secrecy mechanism.

---

## 23. HTTPS Does Not Hide Data from the Browser User

HTTPS protects data in transit between browser and server.

It does not make client-delivered data invisible to the user running that browser.

---

## 24. Server Boundary Mental Model

```text
SECRET
↓
server only

browser needs capability
↓
request server action/API
↓
server uses secret
↓
return minimum safe result
```

This is the correct architecture for private credentials.

---

## 25. Common Mistakes

### Mistake 1

Putting a secret in .env and importing it into client code.

### Mistake 2

Returning database objects directly.

### Mistake 3

Using hidden UI as authorization.

### Mistake 4

Putting sensitive data in JWT payloads.

### Mistake 5

Assuming minification/obfuscation protects secrets.

---

## Interview Questions

### Can you hide a secret in frontend JavaScript?

No. Anything delivered to the browser must be considered observable.

### Is hiding an admin button authorization?

No. Authorization must be enforced server-side.

### Are environment variables always secret?

No. Values bundled into client code become public.

---

## Key Takeaways

- Browser-delivered data is observable.
- Keep secrets on the server.
- Return minimum necessary API data.
- Client roles/flags are not trustworthy authorization.
- Public frontend keys must be intentionally limited.
- Avoid sensitive data in URLs, logs, errors, and bundles.
- HTTPS protects transit, not secrecy from the browser user.
