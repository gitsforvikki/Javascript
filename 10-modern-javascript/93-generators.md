# Lesson 93 — Generators

Generators provide a simpler way to create iterators.

A generator function can pause and resume its execution.

Core syntax:

```text
function*
yield
```

The main idea:

> Calling a generator function does not run it to completion immediately. It returns a generator object that controls execution step by step.

---

## 1. Basic Generator

```js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}
```

Call it:

```js
const iterator =
  numbers();
```

The generator body has not run to completion.

Instead, `iterator` is a generator object.

---

## 2. next()

```js
console.log(
  iterator.next()
);
```

Result:

```js
{
  value: 1,
  done: false,
}
```

Next:

```js
iterator.next();
```

returns:

```js
{
  value: 2,
  done: false,
}
```

Eventually:

```js
{
  value: undefined,
  done: true,
}
```

---

## 3. yield Pauses Execution

```js
function* demo() {
  console.log("A");

  yield 1;

  console.log("B");

  yield 2;

  console.log("C");
}
```

Create:

```js
const gen =
  demo();
```

Nothing printed yet.

First:

```js
gen.next();
```

Output:

```text
A
```

Execution pauses at:

```js
yield 1;
```

Second `next()` resumes from there.

---

## 4. Generator Execution Timeline

```text
demo()
↓
generator object created
↓
body paused before execution

next()
↓
run until yield 1
↓
pause

next()
↓
resume
↓
run until yield 2
↓
pause

next()
↓
resume
↓
finish
```

---

## 5. Generators Are Iterators

Generator objects provide:

```js
next()
```

So they are iterators.

They also implement `Symbol.iterator`.

That means generator objects are also iterable.

---

## 6. for...of with Generator

```js
function* numbers() {
  yield 1;
  yield 2;
  yield 3;
}

for (
  const value
  of numbers()
) {
  console.log(value);
}
```

Output:

```text
1
2
3
```

This is much easier than manually writing an iterator object.

---

## 7. Generator as Custom Iterable

Instead of this verbose iterator:

```js
const range = {
  [Symbol.iterator]() {
    // manual next()
  },
};
```

you can use:

```js
const range = {
  start: 1,
  end: 3,

  *[Symbol.iterator]() {
    for (
      let value =
        this.start;
      value <= this.end;
      value++
    ) {
      yield value;
    }
  },
};
```

Now:

```js
[...range];
```

produces:

```js
[1, 2, 3]
```

---

## 8. return in Generator

```js
function* demo() {
  yield 1;

  return 99;
}
```

Calls:

```js
const gen =
  demo();

gen.next();
// { value: 1, done: false }

gen.next();
// { value: 99, done: true }
```

Important:

`for...of` ignores the final return value because it stops when `done` becomes true.

---

## 9. yield vs return

```text
yield
→ produce value
→ pause
→ can resume

return
→ produce final result
→ finish generator
```

---

## 10. Passing Values into next()

Generators support two-way communication.

```js
function* conversation() {
  const answer =
    yield "What is your name?";

  yield `Hello ${answer}`;
}
```

Use:

```js
const gen =
  conversation();

gen.next();
```

returns the question.

Then:

```js
gen.next(
  "Vikash"
);
```

passes `"Vikash"` back into the paused generator.

The value becomes the result of the previous `yield` expression.

---

## 11. Important First next() Rule

A value passed to the very first:

```js
gen.next(value)
```

is generally ignored because the generator has not paused at a `yield` expression yet.

The communication pattern starts after the first suspension.

---

## 12. yield*

`yield*` delegates to another iterable.

```js
function* first() {
  yield 1;
  yield 2;
}

function* all() {
  yield* first();

  yield 3;
}
```

Then:

```js
[...all()];
```

Result:

```js
[1, 2, 3]
```

---

## 13. yield* with Arrays

```js
function* values() {
  yield* [
    "React",
    "Next.js",
  ];
}
```

Because arrays are iterable.

---

## 14. Infinite Generator

Generators can represent sequences without creating every value in memory first.

```js
function* ids() {
  let id = 1;

  while (true) {
    yield id++;
  }
}
```

Use:

```js
const generator =
  ids();

generator.next().value;
// 1

generator.next().value;
// 2
```

The infinite sequence is generated lazily.

---

## 15. Lazy Evaluation

Normal array creation:

```js
const values = [
  1,
  2,
  3,
  // ...
];
```

stores values upfront.

Generator:

```js
function* values() {
  yield computeValue();
}
```

computes values only when iteration requests them.

This is called **lazy evaluation**.

---

## 16. Generator return()

Generator objects support:

```js
gen.return(
  "finished"
);
```

This ends the generator early.

Result:

```js
{
  value: "finished",
  done: true,
}
```

---

## 17. finally Cleanup

```js
function* resource() {
  try {
    yield 1;
    yield 2;
  } finally {
    console.log(
      "cleanup"
    );
  }
}
```

If the generator is closed early, the `finally` block can run.

This supports cleanup semantics.

---

## 18. generator.throw()

You can inject an error into a paused generator:

```js
gen.throw(
  new Error("Failed")
);
```

If the generator has matching error handling, it can catch that error.

This is advanced control-flow behavior.

---

## 19. Error Handling Inside Generator

```js
function* task() {
  try {
    yield 1;
  } catch (error) {
    console.log(
      error.message
    );
  }
}
```

Then:

```js
const gen =
  task();

gen.next();

gen.throw(
  new Error("Boom")
);
```

Output:

```text
Boom
```

---

## 20. Generator vs Normal Function

Normal function:

```text
call
↓
run
↓
return
↓
finished
```

Generator:

```text
call
↓
generator object

next
↓
run
↓
yield
↓
pause

next
↓
resume
```

---

## 21. Generator vs Iterator

An iterator is the protocol:

```text
next()
→ { value, done }
```

A generator is convenient syntax for implementing that protocol.

So:

> Every generator object is an iterator, but not every iterator was created by a generator.

---

## 22. Practical Range Generator

```js
function* range(
  start,
  end
) {
  for (
    let value = start;
    value <= end;
    value++
  ) {
    yield value;
  }
}
```

Use:

```js
for (
  const number
  of range(
    1,
    5
  )
) {
  console.log(number);
}
```

---

## 23. Pagination Concept

Generators can model paginated/lazy sequences.

Conceptually:

```js
function* pages() {
  yield page1;
  yield page2;
  yield page3;
}
```

In real async pagination, **async generators** are often more appropriate.

---

## 24. Async Generator Preview

Syntax:

```js
async function* stream() {
  yield await getValue();
}
```

Consume:

```js
for await (
  const value
  of stream()
) {
  console.log(value);
}
```

Async generators combine:

- async/await
- iteration
- generators

This is an advanced extension of the concepts you now understand.

---

## 25. React/Application Connection

Generators are less common in everyday React components than Promises or array methods.

But they appear in:

- state-management libraries
- workflow engines
- parsers
- lazy data processing
- custom iterables
- streaming pipelines

Understanding generators also makes advanced libraries easier to read.

---

## 26. Redux-Saga Example Concept

Redux-Saga famously uses generator functions:

```js
function* loadUser() {
  // yield effects
}
```

The library controls the generator and interprets yielded values.

You do not need Redux-Saga to understand generators, but it is a well-known real-world example.

---

## 27. Common Mistakes

### Mistake 1

Thinking calling a generator runs the whole body.

It does not.

### Mistake 2

Confusing `yield` with `return`.

### Mistake 3

Forgetting generators are stateful.

### Mistake 4

Thinking generator laziness makes CPU work automatically asynchronous.

It does not.

Generators control execution, not threads.

---

## Interview Questions

### What is a generator?

A special function declared with `function*` that can pause with `yield` and resume with `next()`.

### What does calling a generator return?

A generator object that is both an iterator and iterable.

### What does yield do?

It produces a value and pauses generator execution.

### yield vs return?

`yield` pauses and can resume; `return` finishes the generator.

### Why are generators useful?

They simplify custom iterators and enable lazy/stateful sequence generation.

---

## Section 10 Complete Mental Model

```text
Modern syntax
↓
cleaner expressions and object patterns

Modules
↓
explicit file dependencies

Map / Set
↓
modern collection types

WeakMap / WeakSet
↓
GC-sensitive object associations

Symbol
↓
unique keys + language protocols

Symbol.iterator
↓
iterable protocol

iterator.next()
↓
values one at a time

generator
↓
easy stateful iterator creation
```

---

## Key Takeaways

- Generator functions use `function*`.
- `yield` pauses execution.
- `next()` resumes execution.
- Generator objects follow the iterator protocol.
- Generator objects are also iterable.
- `yield*` delegates to another iterable.
- Generators support lazy sequences.
- Values can be passed back into a paused generator.
- Generators are execution-control tools, not asynchronous threads.
