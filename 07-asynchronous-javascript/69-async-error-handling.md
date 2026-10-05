# Lesson 69 — Error Handling with `async/await`

Async/await makes Promise code look synchronous.

That also means Promise rejections can often be handled using familiar:

- `try`
- `catch`
- `finally`

The key idea is:

> An awaited rejected Promise behaves like a thrown error at the await point.

---

## 1. Basic try/catch

```js
async function loadData() {
  try {
    const data =
      await getData();

    console.log(data);
  } catch (error) {
    console.error(
      error
    );
  }
}
```

If `getData()` rejects, execution jumps to `catch`.

---

# 2. Rejected Promise Becomes Throw

```js
async function test() {
  try {
    await Promise.reject(
      new Error("Failed")
    );
  } catch (error) {
    console.log(
      error.message
    );
  }
}
```

Output:

```text
Failed
```

Mental model:

```text
await rejected Promise
↓
throw at await point
↓
catch handles error
```

---

# 3. try/catch Can Cover Multiple awaits

```js
async function load() {
  try {
    const user =
      await getUser();

    const orders =
      await getOrders(
        user.id
      );

    const details =
      await getDetails(
        orders[0].id
      );

    return details;
  } catch (error) {
    console.error(
      error
    );
  }
}
```

If any awaited operation rejects, control moves to the catch block.

---

# 4. Catching Too Broadly

A large `try` block is convenient:

```js
try {
  await step1();
  await step2();
  await step3();
}
```

But sometimes you need to know exactly which operation failed.

In those cases:

- use smaller try/catch blocks
- add contextual errors
- wrap lower-level errors with `cause`

---

# 5. Rethrowing

```js
async function loadUser() {
  try {
    return await fetchUser();
  } catch (error) {
    console.error(
      "loadUser failed"
    );

    throw error;
  }
}
```

The function still returns a rejected Promise.

You handled/logged locally, then propagated the failure upward.

---

# 6. Error Transformation

```js
async function loadProfile() {
  try {
    return await fetchProfile();
  } catch (error) {
    throw new Error(
      "Unable to load profile",
      {
        cause: error,
      }
    );
  }
}
```

Now higher-level code receives a more meaningful domain error.

---

# 7. Catch and Recover

```js
async function loadSettings() {
  try {
    return await fetchSettings();
  } catch {
    return {
      theme: "light",
    };
  }
}
```

If the request fails, the async function fulfills with a fallback object.

Important:

> Returning from catch recovers the operation.

---

# 8. finally with async/await

```js
async function submit() {
  setLoading(true);

  try {
    await saveData();

    showSuccess();
  } catch (error) {
    showError(error);
  } finally {
    setLoading(false);
  }
}
```

`finally` runs whether the operation:

- succeeds
- throws
- rejects

This is perfect for cleanup.

---

# 9. finally Runs Before Function Completion

```js
async function test() {
  try {
    return "success";
  } finally {
    console.log(
      "cleanup"
    );
  }
}
```

The cleanup runs before the returned Promise settles with `"success"`.

---

# 10. finally Can Override Outcome

```js
async function test() {
  try {
    return "success";
  } finally {
    throw new Error(
      "cleanup failed"
    );
  }
}
```

The async function rejects with:

```text
cleanup failed
```

So do not place risky logic casually inside `finally`.

---

# 11. Fetch HTTP Errors Still Need Checking

```js
async function loadUsers() {
  const response =
    await fetch(
      "/api/users"
    );

  if (!response.ok) {
    throw new Error(
      `HTTP ${response.status}`
    );
  }

  return response.json();
}
```

Why?

`fetch()` normally does not reject just because the server returns HTTP 404 or 500.

You must decide which HTTP statuses count as application failures.

---

# 12. Correct Fetch Pattern

```js
async function fetchJson(url) {
  const response =
    await fetch(url);

  if (!response.ok) {
    throw new Error(
      `Request failed: ${response.status}`
    );
  }

  return await response.json();
}
```

Then:

```js
try {
  const users =
    await fetchJson(
      "/api/users"
    );
} catch (error) {
  console.error(error);
}
```

---

# 13. Local Error Handling

Suppose notifications are optional:

```js
async function loadPage() {
  const user =
    await getUser();

  let notifications = [];

  try {
    notifications =
      await getNotifications();
  } catch {
    notifications = [];
  }

  return {
    user,
    notifications,
  };
}
```

Only the optional operation gets a fallback.

---

# 14. Do Not Swallow Critical Errors

Bad:

```js
try {
  await processPayment();
} catch (error) {
  console.log(error);
}

return {
  success: true,
};
```

This may report success after a critical failure.

Error recovery should reflect business requirements.

---

# 15. Promise catch Still Works with async Functions

Since async functions return Promises:

```js
async function load() {
  throw new Error(
    "Failed"
  );
}
```

You can consume with:

```js
load().catch(
  (error) => {
    console.error(error);
  }
);
```

So both styles are valid:

```text
inside async function
→ try/catch

outside returned Promise
→ .catch()
```

---

# 16. Forgetting await Changes Error Handling

Consider:

```js
async function test() {
  try {
    return failingAsyncTask();
  } catch (error) {
    console.log(
      "caught"
    );
  }
}
```

If `failingAsyncTask()` returns a Promise that rejects later, the local `catch` may not catch it because the Promise was returned without being awaited inside the try block.

If you want this try/catch to handle the rejection:

```js
async function test() {
  try {
    return await failingAsyncTask();
  } catch (error) {
    console.log(
      "caught"
    );

    throw error;
  }
}
```

This is an important nuance.

---

# 17. return await Is Usually Redundant — Except Sometimes

In simple code:

```js
async function load() {
  return await getData();
}
```

can often be:

```js
async function load() {
  return getData();
}
```

But inside a local `try/catch`, `return await` can be necessary when you want that local catch to observe the rejection.

---

# 18. Parallel Error Handling with Promise.all

```js
async function loadDashboard() {
  try {
    const [
      profile,
      jobs,
      stats,
    ] =
      await Promise.all([
        getProfile(),
        getJobs(),
        getStats(),
      ]);

    return {
      profile,
      jobs,
      stats,
    };
  } catch (error) {
    console.error(
      "Dashboard failed",
      error
    );

    throw error;
  }
}
```

A rejection from any required operation causes the await to throw.

---

# 19. Partial Failure with allSettled

```js
async function loadWidgets() {
  const results =
    await Promise.allSettled([
      getProfile(),
      getStats(),
      getRecommendations(),
    ]);

  return results;
}
```

No catch is required merely because one input rejected, because `allSettled()` itself fulfills with all outcomes.

---

# 20. Cleanup Example

```js
async function uploadFile(file) {
  showSpinner();

  try {
    const result =
      await upload(file);

    return result;
  } catch (error) {
    showError(
      error.message
    );

    throw error;
  } finally {
    hideSpinner();
  }
}
```

This is a clean success/error/cleanup structure.

---

# 21. React / Next.js Connection

In server actions or handlers:

```js
export async function saveJob(data) {
  try {
    const result =
      await db.insert(data);

    return {
      success: true,
      result,
    };
  } catch (error) {
    console.error(error);

    return {
      success: false,
      message:
        "Unable to save job",
    };
  }
}
```

Important: decide carefully whether to:

- return a failure object
- throw
- log and rethrow

based on the application boundary.

---

# Interview Questions

### How do you handle rejected Promises with async/await?

Use `try/catch` around the awaited operation.

### What happens if an async function throws?

Its returned Promise becomes rejected.

### Can catch recover an async function?

Yes. Returning a normal value from catch fulfills the async function's Promise with that value.

### Why can return await matter inside try/catch?

Because awaiting causes the rejection to occur inside that try block, allowing the local catch to handle it.

---

# Key Takeaways

- Awaited rejections behave like thrown errors.
- `try/catch` is the standard async/await error pattern.
- `finally` is useful for cleanup.
- Returning from catch recovers the async function.
- Throwing from catch propagates failure.
- Fetch HTTP failures often require manual `response.ok` checks.
- Avoid swallowing critical errors.
- `return await` is especially relevant when local try/catch must observe a rejection.
