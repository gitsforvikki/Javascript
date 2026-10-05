# Lesson 115 — Efficient Data Processing

Efficient JavaScript is not about replacing every readable method with a low-level loop.

It is about:

- choosing the right algorithm
- choosing the right data structure
- avoiding unnecessary work
- avoiding repeated traversals when they matter
- measuring before optimizing

---

## 1. Algorithm First

Compare:

```js
array.find(
  (item) =>
    item.id === id
);
```

for repeated lookup versus:

```js
map.get(id);
```

If lookups happen constantly, a Map may be a better data structure.

---

## 2. Complexity Mental Model

Common rough costs:

```text
array index access
→ O(1)

Map/Set lookup
→ average O(1)

array find/includes
→ O(n)

nested full scans
→ often O(n²)

sorting
→ usually O(n log n)
```

Big-O is a growth model, not an exact runtime prediction.

---

## 3. Avoid Repeated Linear Search

Bad pattern:

```js
for (
  const order
  of orders
) {
  const user =
    users.find(
      (user) =>
        user.id ===
        order.userId
    );
}
```

If both arrays are large, this can approach:

```text
O(n × m)
```

---

## 4. Build an Index

```js
const usersById =
  new Map(
    users.map(
      (user) => [
        user.id,
        user,
      ]
    )
  );

for (
  const order
  of orders
) {
  const user =
    usersById.get(
      order.userId
    );
}
```

You pay once to build the index, then perform fast lookups.

---

## 5. Multiple Array Passes

Readable:

```js
const result =
  items
    .filter(isActive)
    .map(toViewModel)
    .filter(isVisible);
```

This performs multiple passes and creates intermediate arrays.

Often perfectly fine.

---

## 6. Single-Pass Alternative

If data is huge or performance is proven to matter:

```js
const result = [];

for (
  const item
  of items
) {
  if (!isActive(item)) {
    continue;
  }

  const view =
    toViewModel(
      item
    );

  if (!isVisible(view)) {
    continue;
  }

  result.push(view);
}
```

One pass, fewer intermediates.

Use only when the extra complexity is justified.

---

## 7. Do Not Prematurely Optimize

For 100 items:

```js
filter().map()
```

may be clearer and fast enough.

Optimize when:

- profiling shows a bottleneck
- dataset is large
- operation runs very frequently

---

## 8. Avoid Recomputing Expensive Values

Bad:

```js
for (
  const item
  of items
) {
  const config =
    expensiveConfig();

  process(
    item,
    config
  );
}
```

If config is identical each time:

```js
const config =
  expensiveConfig();

for (
  const item
  of items
) {
  process(
    item,
    config
  );
}
```

---

## 9. Memoization

If an expensive deterministic result repeats:

```text
same input
→ same output
```

memoization may help.

But remember cache overhead and invalidation.

---

## 10. Use Set for Membership

Instead of:

```js
blockedIds.includes(
  id
);
```

inside a large hot loop, consider:

```js
const blocked =
  new Set(
    blockedIds
  );

blocked.has(id);
```

This can significantly improve repeated membership checks.

---

## 11. Avoid Accidental Quadratic Work

Example:

```js
for (
  const a
  of items
) {
  for (
    const b
    of items
  ) {
    // ...
  }
}
```

Nested loops are not always bad, but understand the growth.

For 10 items:

```text
100 comparisons
```

For 10,000:

```text
100,000,000 comparisons
```

---

## 12. Early Exit

Use:

```js
find()
some()
every()
```

when you can stop early.

Example:

```js
const hasError =
  items.some(
    isInvalid
  );
```

This stops on the first match.

---

## 13. Do Not Sort Unless Needed

Sorting has non-trivial cost.

Bad:

```js
items
  .sort(compare)
  .find(predicate);
```

If sorting is irrelevant to the lookup, remove it.

---

## 14. Avoid Repeated Sorting

If the same dataset is sorted repeatedly with the same criteria, consider:

- sorting once
- memoizing derived data
- maintaining indexed structures

depending on update frequency.

---

## 15. Strings and Concatenation

Modern engines optimize string concatenation well.

Do not assume:

```js
result += piece;
```

is automatically bad.

For very large text construction, measure alternatives such as array accumulation + `join()`.

---

## 16. Object Allocation in Hot Loops

This:

```js
for (...) {
  result.push({
    a,
    b,
  });
}
```

may be exactly what you need.

But if objects are throwaway intermediates in a very hot path, reducing allocations may reduce GC pressure.

Measure first.

---

## 17. Chunk Large Work

Processing 1,000,000 items synchronously can block the main thread.

Instead, process in chunks:

```text
process 5,000 items
↓
yield
↓
process next chunk
```

This improves responsiveness.

---

## 18. Generator / Iterator Connection

Generators can lazily produce values:

```js
function* transform(
  items
) {
  for (
    const item
    of items
  ) {
    yield process(
      item
    );
  }
}
```

Useful when you do not need every result upfront.

---

## 19. Streaming Mental Model

Instead of:

```text
load everything
↓
process everything
↓
render everything
```

sometimes use:

```text
receive chunk
↓
process chunk
↓
render/use chunk
```

Streaming reduces peak memory and improves time-to-first-result.

---

## 20. DOM Batching

Bad:

```js
for (
  const item
  of items
) {
  document.body.append(
    createRow(item)
  );
}
```

Better structure may use:

```js
const fragment =
  document.createDocumentFragment();
```

build first, then append together.

The exact performance gain depends on browser behavior and what layout work occurs.

---

## 21. Read/Write Grouping

Repeated:

```text
read layout
write style
read layout
write style
```

can cause forced layout work.

Prefer:

```text
read all
↓
compute
↓
write all
```

when possible.

---

## 22. React Derived Data

Expensive derived data:

```js
const filtered =
  expensiveFilter(
    items,
    query
  );
```

may be memoized if:

- computation is actually expensive
- dependencies are stable
- recomputation is frequent

Use `useMemo` only when justified.

---

## 23. Pagination and Virtualization

Do not render/process thousands of records if the user can only see 20.

Strategies:

- pagination
- infinite scrolling
- virtualization/windowing

This reduces:

- DOM size
- memory
- render work
- processing

---

## 24. Server vs Client Processing

Sometimes the best client optimization is:

> Do less work on the client.

Examples:

- server-side filtering
- server-side pagination
- pre-aggregation
- indexed database query

Architecture often matters more than micro-optimization.

---

## 25. Measure with Performance Tools

Use:

- Performance panel
- console.time / console.timeEnd
- performance.now()
- profiler tooling

Example:

```js
const start =
  performance.now();

runTask();

const end =
  performance.now();

console.log(
  end - start
);
```

---

## 26. Microbenchmarks Can Mislead

A tiny benchmark may ignore:

- JIT optimization
- warm-up
- GC
- browser rendering
- network
- real dataset shape

Prefer measuring realistic application scenarios.

---

## Interview Questions

### What is usually more important than micro-optimizing loops?

Choosing the right algorithm and data structure.

### Why use Map instead of repeated find?

For repeated key lookups, Map can avoid repeated O(n) scans.

### When combine filter/map into one loop?

Only when profiling shows the extra passes/intermediate arrays matter.

---

## Key Takeaways

- Optimize algorithms before syntax.
- Use Maps/Sets for repeated lookup/membership.
- Avoid accidental O(n²) work.
- Early-exit when possible.
- Do not sort/process more than needed.
- Reduce repeated expensive computations.
- Chunk or offload large work.
- Measure real bottlenecks before optimizing.
