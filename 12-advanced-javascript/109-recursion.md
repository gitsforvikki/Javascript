# Lesson 109 — Recursion

Recursion is when a function solves a problem by calling itself with a smaller or simpler version of that problem.

Every useful recursive solution needs:

- a base case
- a recursive case
- progress toward the base case

---

## 1. Basic Example

```js
function countdown(
  n
) {
  if (n <= 0) {
    console.log(
      "Done"
    );

    return;
  }

  console.log(n);

  countdown(
    n - 1
  );
}
```

Call:

```js
countdown(3);
```

Output:

```text
3
2
1
Done
```

---

## 2. Base Case

The base case stops recursion.

```js
if (n <= 0) {
  return;
}
```

Without it:

```js
function recurse() {
  recurse();
}
```

the function keeps calling itself until the call stack overflows.

---

## 3. Recursive Case

```js
countdown(
  n - 1
);
```

The recursive case reduces the problem.

Each call moves toward the base case.

---

## 4. Call Stack Mental Model

For:

```js
countdown(3);
```

conceptually:

```text
countdown(3)
↓
countdown(2)
↓
countdown(1)
↓
countdown(0)
↓
return
↑
return
↑
return
↑
return
```

Each active call has its own execution context.

---

## 5. Factorial

Definition:

```text
5!
=
5 × 4 × 3 × 2 × 1
```

Recursive:

```js
function factorial(
  n
) {
  if (n <= 1) {
    return 1;
  }

  return (
    n *
    factorial(
      n - 1
    )
  );
}
```

---

## 6. Trace Factorial

```js
factorial(4);
```

Expands:

```text
4 * factorial(3)
4 * 3 * factorial(2)
4 * 3 * 2 * factorial(1)
4 * 3 * 2 * 1
24
```

---

## 7. Recursion Has Two Phases

Many recursive functions have:

```text
descending phase
↓
build stack

base case

unwinding phase
↑
return values
```

Understanding stack unwinding is essential.

---

## 8. Recursive Sum

```js
function sum(
  numbers,
  index = 0
) {
  if (
    index >=
    numbers.length
  ) {
    return 0;
  }

  return (
    numbers[index] +
    sum(
      numbers,
      index + 1
    )
  );
}
```

---

## 9. Tree-Like Data

Recursion is especially natural for nested structures.

Example:

```js
const node = {
  value: "root",
  children: [
    {
      value: "A",
      children: [],
    },
    {
      value: "B",
      children: [],
    },
  ],
};
```

Traversal:

```js
function visit(
  node
) {
  console.log(
    node.value
  );

  for (
    const child
    of node.children
  ) {
    visit(child);
  }
}
```

---

## 10. DOM Traversal Connection

The DOM is a tree.

Recursive traversal:

```js
function walk(
  element
) {
  console.log(
    element
  );

  for (
    const child
    of element.children
  ) {
    walk(child);
  }
}
```

Recursion maps naturally to tree structures.

---

## 11. Nested Object Traversal

```js
function printKeys(
  object
) {
  for (
    const [
      key,
      value,
    ]
    of Object.entries(
      object
    )
  ) {
    console.log(key);

    if (
      value &&
      typeof value ===
        "object"
    ) {
      printKeys(
        value
      );
    }
  }
}
```

Be careful with circular references.

---

## 12. Circular Reference Problem

```js
const a = {};
a.self = a;
```

Naive recursion:

```js
printKeys(a);
```

may recurse forever.

Use a `WeakSet` to track visited objects.

---

## 13. Safe Recursive Traversal

```js
function visit(
  value,
  seen =
    new WeakSet()
) {
  if (
    !value ||
    typeof value !==
      "object"
  ) {
    return;
  }

  if (
    seen.has(value)
  ) {
    return;
  }

  seen.add(value);

  for (
    const child
    of Object.values(
      value
    )
  ) {
    visit(
      child,
      seen
    );
  }
}
```

This avoids infinite cycles.

---

## 14. Recursion vs Iteration

Recursive:

```js
function countdown(
  n
) {
  if (n === 0) {
    return;
  }

  countdown(
    n - 1
  );
}
```

Iterative:

```js
for (
  let n = 10;
  n > 0;
  n--
) {
}
```

Both can solve similar problems.

---

## 15. When Recursion Is Better

Recursion is natural for:

- trees
- graphs
- nested menus
- filesystem traversal
- divide-and-conquer algorithms
- parser structures

---

## 16. When Iteration Is Better

Iteration may be simpler for:

- straightforward counting
- large linear loops
- stack-sensitive workloads

Use the clearest safe solution.

---

## 17. Stack Overflow

Each recursive call uses stack space.

Too-deep recursion can cause:

```text
RangeError:
Maximum call stack size exceeded
```

Exact limits depend on the runtime.

---

## 18. Tail Recursion

A tail-recursive function ends by directly returning the recursive call.

Example:

```js
function factorial(
  n,
  acc = 1
) {
  if (n <= 1) {
    return acc;
  }

  return factorial(
    n - 1,
    acc * n
  );
}
```

However, do not assume JavaScript engines universally optimize tail calls in practical environments.

Stack overflow can still be a concern.

---

## 19. Recursion and Memoization

Recursive algorithms often repeat work.

Example Fibonacci:

```js
fib(n - 1) +
fib(n - 2)
```

Memoization stores repeated subproblem results.

This connects directly to Lesson 105.

---

## 20. Divide and Conquer

Recursive algorithms often split problems.

Examples:

- merge sort
- quicksort
- binary search
- tree traversal

Conceptually:

```text
large problem
↓
smaller problems
↓
solve recursively
↓
combine results
```

---

## 21. Binary Search Recursive Example

```js
function binarySearch(
  array,
  target,
  left = 0,
  right =
    array.length - 1
) {
  if (
    left > right
  ) {
    return -1;
  }

  const middle =
    Math.floor(
      (
        left +
        right
      ) / 2
    );

  if (
    array[middle] ===
    target
  ) {
    return middle;
  }

  if (
    target <
    array[middle]
  ) {
    return binarySearch(
      array,
      target,
      left,
      middle - 1
    );
  }

  return binarySearch(
    array,
    target,
    middle + 1,
    right
  );
}
```

---

## 22. React Component Recursion

Nested UI:

```jsx
function MenuItem({
  item,
}) {
  return (
    <li>
      {item.label}

      {item.children?.length >
        0 && (
        <ul>
          {item.children.map(
            (child) => (
              <MenuItem
                key={
                  child.id
                }
                item={
                  child
                }
              />
            )
          )}
        </ul>
      )}
    </li>
  );
}
```

A component can recursively render tree data.

---

## 23. Recursion Checklist

Before writing recursion, identify:

```text
1. Base case
2. Smaller subproblem
3. Progress toward base
4. Return/combination logic
5. Maximum depth risk
6. Circular data risk
```

---

## 24. Common Mistakes

### Mistake 1

Missing base case.

### Mistake 2

Recursive call does not move toward base case.

### Mistake 3

Forgetting return during recursive accumulation.

### Mistake 4

Ignoring circular references.

### Mistake 5

Using recursion for huge linear depth when iteration is safer.

---

## Interview Questions

### What is recursion?

A function solving a problem by calling itself with a smaller version of that problem.

### What is a base case?

The condition that stops recursive calls.

### Why can recursion overflow?

Each active recursive call consumes call-stack space.

### When is recursion especially useful?

Tree/nested structures and divide-and-conquer problems.

---

## Key Takeaways

- Recursion needs a base case.
- Each recursive call must progress toward it.
- Recursive calls build stack frames.
- Returning values unwinds the stack.
- Trees and nested structures are natural recursion use cases.
- Deep recursion can overflow the stack.
- Circular structures require visited tracking.
- Memoization can optimize repeated recursive subproblems.
