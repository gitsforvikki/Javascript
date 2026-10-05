# Lesson 99 — Retry, Timeout and Resilient Request Patterns

Real networks fail.

A production request layer must consider:

- timeout
- cancellation
- retries
- backoff
- jitter
- idempotency
- rate limits
- retry budgets
- user experience

The most important rule is:

> Do not retry every failure blindly.

---

## 1. Why Requests Need Timeouts

Without a timeout, a request may remain pending much longer than the user or application can tolerate.

A timeout is an application policy:

```text
If operation exceeds X time,
stop waiting / cancel it
```

---

## 2. Timeout with AbortController

Portable pattern:

```js
async function fetchWithTimeout(
  url,
  {
    timeoutMs = 5000,
    ...options
  } = {}
) {
  const controller =
    new AbortController();

  const timer =
    setTimeout(() => {
      controller.abort(
        new Error(
          "Request timed out"
        )
      );
    }, timeoutMs);

  try {
    return await fetch(
      url,
      {
        ...options,
        signal:
          controller.signal,
      }
    );
  } finally {
    clearTimeout(timer);
  }
}
```

This actually signals cancellation to Fetch.

---

## 3. External Signal + Timeout

A more complete helper may need both:

- caller cancellation
- timeout cancellation

Conceptually:

```text
user abort
OR
timeout
↓
request aborts
```

Modern runtimes may use `AbortSignal.any()`.

Otherwise, you can manually bridge signals.

---

## 4. What Is a Retry?

Retry means performing the operation again after a failure.

Example:

```text
attempt 1
↓
temporary failure
↓
wait
↓
attempt 2
```

Retries are useful only when the failure may be temporary and repeating the request is safe.

---

## 5. Retryable Failure Examples

Often reasonable candidates:

- temporary network failure
- 408 Request Timeout
- 429 Too Many Requests
- some 5xx responses
- connection reset
- transient gateway failure

But API-specific behavior matters.

---

## 6. Usually Non-Retryable Examples

Often do not retry automatically:

- 400 malformed request
- 401 without changing credentials
- 403 forbidden
- 404 for a resource that truly does not exist
- validation error
- business-rule rejection

Retrying the same invalid request usually creates load without helping.

---

## 7. The Biggest Risk — Retrying Mutations

Suppose:

```text
POST /payments
↓
server charges card
↓
response lost
↓
client thinks request failed
↓
client retries
↓
possible duplicate charge
```

This is why retry safety depends on **idempotency**.

---

## 8. What Is Idempotency?

An operation is idempotent when repeating the same operation has the same intended final effect as performing it once.

Examples conceptually:

```text
GET resource
→ normally idempotent

PUT resource to exact state
→ idempotent by design

DELETE resource
→ idempotent by design

POST create/payment
→ not inherently idempotent
```

Real server implementations still matter.

---

## 9. Idempotency Keys

For sensitive POST operations, APIs may support:

```http
Idempotency-Key: unique-request-id
```

The server remembers the operation and prevents duplicate processing.

This is common in payment APIs.

Client retries become much safer when the server supports idempotency.

---

## 10. Do Not Retry Immediately in a Tight Loop

Bad:

```js
while (true) {
  try {
    return await fetch(
      url
    );
  } catch {}
}
```

This can hammer the server/network.

Use delay between attempts.

---

## 11. Fixed Delay

Simple:

```text
attempt
↓ fail
wait 1 second
↓
attempt again
```

Better than immediate retry, but still not ideal under large-scale outages.

---

## 12. Exponential Backoff

A common strategy:

```text
attempt 1
↓ fail
wait 500ms

attempt 2
↓ fail
wait 1000ms

attempt 3
↓ fail
wait 2000ms

attempt 4
↓ fail
wait 4000ms
```

Formula conceptually:

```js
baseDelay *
2 ** attempt
```

Usually cap the maximum delay.

---

## 13. Why Backoff Helps

If a service is overloaded, immediate repeated retries can make the outage worse.

This can create a:

```text
retry storm
```

Backoff gives the system time to recover.

---

## 14. Jitter

If thousands of clients all retry after exactly:

```text
1s
2s
4s
```

they may all hit the server simultaneously again.

Jitter adds randomness.

Example:

```js
const delay =
  Math.random() *
  maxDelay;
```

or randomize around an exponential-backoff value.

This spreads retry traffic.

---

## 15. Full Jitter Concept

One common pattern:

```js
const cap =
  Math.min(
    maxDelay,
    baseDelay *
      2 ** attempt
  );

const delay =
  Math.random() *
  cap;
```

This is often called full jitter.

Exact retry algorithms depend on system needs.

---

## 16. Delay Helper

```js
function delay(
  ms,
  signal
) {
  return new Promise(
    (
      resolve,
      reject
    ) => {
      if (
        signal?.aborted
      ) {
        reject(
          signal.reason
        );

        return;
      }

      const id =
        setTimeout(
          resolve,
          ms
        );

      signal?.addEventListener(
        "abort",
        () => {
          clearTimeout(id);

          reject(
            signal.reason
          );
        },
        {
          once: true,
        }
      );
    }
  );
}
```

Now retry waiting can also be cancelled.

---

## 17. Basic Retry Helper

```js
async function retry(
  operation,
  {
    retries = 3,
    baseDelay = 500,
  } = {}
) {
  let lastError;

  for (
    let attempt = 0;
    attempt <= retries;
    attempt++
  ) {
    try {
      return await operation();
    } catch (error) {
      lastError =
        error;

      if (
        attempt === retries
      ) {
        break;
      }

      const wait =
        baseDelay *
        2 ** attempt;

      await delay(wait);
    }
  }

  throw lastError;
}
```

This is a foundation, not yet production-ready.

---

## 18. Add Retry Policy

Do not retry everything.

```js
function isRetryable(
  error
) {
  if (
    error.code ===
    "NETWORK_ERROR"
  ) {
    return true;
  }

  if (
    error.status ===
    429
  ) {
    return true;
  }

  if (
    error.status >= 500 &&
    error.status < 600
  ) {
    return true;
  }

  return false;
}
```

Then stop retrying non-retryable failures immediately.

---

## 19. Respect Retry-After

For 429 or some service responses:

```http
Retry-After: 30
```

The client should consider honoring server guidance.

The header can represent seconds or an HTTP date depending on format.

A resilient client parses it carefully.

---

## 20. Retry Count vs Total Time Budget

Three retries can mean very different total durations.

Example:

```text
request timeout: 10s
retries: 3
backoff included

worst case
→ potentially very long user wait
```

Think in terms of a **total operation budget**, not just retry count.

---

## 21. Per-Attempt Timeout vs Overall Timeout

Two different policies:

```text
per-attempt timeout
→ each request gets max 5s

overall timeout
→ entire operation including retries
  must finish within 12s
```

Production systems often need both.

---

## 22. Retry + Timeout Timeline

Example:

```text
Attempt 1
max 3s
↓ fail

Backoff
500ms

Attempt 2
max 3s
↓ fail

Backoff
1000ms

Attempt 3
max 3s
```

Total user wait can exceed individual request timeout significantly.

---

## 23. Retry with AbortController

A caller should be able to cancel the whole retry loop.

Conceptually:

```js
retry(
  () =>
    fetchWithTimeout(
      url,
      {
        signal,
      }
    ),
  {
    signal,
  }
);
```

Cancellation should stop:

- active request
- waiting backoff
- future attempts

---

## 24. Avoid Retrying Abort Errors

If the user intentionally cancelled:

```text
abort
↓
do not retry
```

Otherwise cancellation becomes pointless.

Retry logic must distinguish:

- transient failure
- explicit cancellation

---

## 25. Circuit Breaker Preview

If a dependency is repeatedly failing, continuing to send requests may be harmful.

A circuit breaker can temporarily stop requests after too many failures.

Conceptually:

```text
CLOSED
→ requests allowed

too many failures
↓
OPEN
→ fail fast temporarily

after cooldown
↓
HALF-OPEN
→ test recovery
```

This is more common in backend/distributed systems, but the resilience concept is useful.

---

## 26. Hedged Requests Preview

For latency-sensitive systems, some architectures start a backup request if the first is unusually slow.

This can reduce tail latency but increases load.

It is an advanced technique and should not be used casually.

---

## 27. React / UI Considerations

Retries affect user experience.

Possible behavior:

```text
attempting request
↓
show loading

temporary failure
↓
silent retry once

still failing
↓
show useful error + Retry button
```

Do not trap the user in endless automatic retry loops.

---

## 28. Manual Retry Button

A simple and often excellent UX:

```jsx
<button
  onClick={() => {
    loadData();
  }}
>
  Retry
</button>
```

Automatic retry is not always better than explicit user control.

---

## 29. Next.js / Server Considerations

Server-side retries can amplify load across many incoming requests.

Be careful retrying downstream services from server code.

A single user request can fan out into multiple backend attempts.

Always consider:

- idempotency
- latency budget
- dependency capacity
- observability

---

## 30. Observability

When retries happen, log enough context to understand them:

```text
attempt number
status/error code
elapsed time
request id
retry delay
final outcome
```

But avoid logging secrets or sensitive request bodies.

---

## 31. A Resilient Request Mental Model

```text
make request
↓
success?
├── yes
│   ↓
│ return result
│
└── no
    ↓
cancelled?
├── yes → stop
│
└── no
    ↓
retryable?
├── no → throw
│
└── yes
    ↓
within retry/time budget?
├── no → throw
│
└── yes
    ↓
backoff + jitter
    ↓
retry
```

---

## 32. What Not to Retry Blindly

Do not automatically retry:

- validation failures
- forbidden access
- incorrect credentials without refresh/change
- duplicate-sensitive POSTs without idempotency
- cancelled requests
- deterministic client bugs

---

## 33. Complete Example Policy

Conceptually:

```js
async function loadWithRetry(
  operation,
  options
) {
  // 1. run attempt
  // 2. enforce timeout
  // 3. stop if aborted
  // 4. classify error
  // 5. stop if non-retryable
  // 6. enforce max attempts/budget
  // 7. respect Retry-After
  // 8. exponential backoff + jitter
  // 9. retry
}
```

The important part is the policy, not memorizing one helper implementation.

---

## Interview Questions

### Why use exponential backoff?

To avoid hammering a failing or overloaded service with immediate repeated requests.

### What is jitter?

Randomness added to retry delays so many clients do not retry at the same time.

### Why is idempotency important for retries?

Because repeating non-idempotent operations can create duplicate side effects.

### Should every 5xx be retried?

No. Retry policy should consider API semantics, attempt budget, idempotency, and whether the failure is likely transient.

### Promise.race timeout vs AbortController timeout?

A race may only stop waiting; AbortController can signal cancellation to the underlying Fetch.

---

## Section 11 Complete Mental Model

```text
JavaScript error
↓
try / catch / finally
↓
throw meaningful Error objects
↓
custom domain/API errors
↓
fetch request
↓
network layer
↓
HTTP response
↓
parse layer
↓
business layer
↓
cancellation
↓
timeout
↓
retry classification
↓
idempotency
↓
backoff + jitter
↓
resilient request policy
```

---

## Key Takeaways

- Timeouts are explicit application policy.
- Retry only potentially transient failures.
- Do not blindly retry mutations.
- Idempotency is central to safe retries.
- Use exponential backoff.
- Add jitter to prevent synchronized retry storms.
- Respect server guidance such as `Retry-After`.
- Cancellation should stop retry loops.
- Consider total latency budget, not only retry count.
- Resilience requires correctness, not just more retries.
