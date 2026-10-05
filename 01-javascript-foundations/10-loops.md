# Lesson 10 — Loops and Iteration

Loops allow us to execute code repeatedly.

## for Loop

Use a `for` loop when you know how the iteration should progress.

```js
for (let i = 0; i < 5; i++) {
  console.log(i);
}
```

Output:

```text
0
1
2
3
4
```

## while Loop

A `while` loop continues while its condition is truthy.

```js
let count = 0;

while (count < 3) {
  console.log(count);
  count++;
}
```

## do...while

A `do...while` executes at least once before checking the condition.

```js
let count = 0;

do {
  console.log(count);
  count++;
} while (count < 3);
```

## for...of

Useful for iterating over iterable values such as arrays and strings.

```js
const skills = ["JavaScript", "React", "Next.js"];

for (const skill of skills) {
  console.log(skill);
}
```

## for...in

`for...in` iterates over enumerable property keys of an object.

```js
const user = {
  name: "Vikash",
  role: "Developer",
};

for (const key in user) {
  console.log(key, user[key]);
}
```

For arrays, prefer `for...of` or array methods rather than `for...in`.

## break

Stops the loop completely.

```js
for (let i = 1; i <= 10; i++) {
  if (i === 5) {
    break;
  }

  console.log(i);
}
```

## continue

Skips the current iteration and continues with the next one.

```js
for (let i = 1; i <= 5; i++) {
  if (i === 3) {
    continue;
  }

  console.log(i);
}
```

## Loops vs Array Methods

In modern JavaScript, arrays are also commonly processed using methods such as:

```js
map()
filter()
reduce()
forEach()
find()
```

We will cover these properly in the arrays section.

## Key Takeaways

- `for` is a general-purpose loop.
- `while` runs while a condition remains truthy.
- `do...while` always runs at least once.
- `for...of` is useful for arrays and other iterables.
- `for...in` iterates object property keys.
- `break` stops a loop and `continue` skips an iteration.

## Interview Quick Answer

**What is the difference between for...of and for...in?**

`for...of` iterates over values from an iterable such as an array, while `for...in` iterates over enumerable property keys of an object.
