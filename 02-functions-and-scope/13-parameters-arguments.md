# Lesson 13 — Parameters, Arguments and Default Parameters

Functions often need input. JavaScript uses **parameters** and **arguments** to pass that input.

## Parameters vs Arguments

Parameters are variables written when defining a function.

```js
function greet(name) {
  console.log(`Hello ${name}`);
}
```

Here, `name` is a parameter.

Arguments are actual values passed when calling the function.

```js
greet("Vikash");
```

Here, `"Vikash"` is an argument.

Conceptually:

```text
function definition
        ↓
function greet(name)
               ↑
           parameter

function call
        ↓
greet("Vikash")
       ↑
    argument
```

## Multiple Parameters

```js
function add(a, b) {
  return a + b;
}

add(10, 20);
```

Mapping:

```text
a ← 10
b ← 20
```

## Missing Arguments

JavaScript does not require every parameter to receive an argument.

```js
function showUser(name, age) {
  console.log(name);
  console.log(age);
}

showUser("Vikash");
```

Output:

```text
Vikash
undefined
```

The missing argument results in `undefined`.

## Extra Arguments

JavaScript also allows more arguments than declared parameters.

```js
function greet(name) {
  console.log(name);
}

greet("Vikash", 25, "India");
```

The named parameter receives only the corresponding first value.

## Default Parameters

Default parameters provide fallback values.

```js
function greet(name = "Guest") {
  return `Hello ${name}`;
}

console.log(greet());         // Hello Guest
console.log(greet("Vikash")); // Hello Vikash
```

Defaults are used when the argument is missing or explicitly `undefined`.

```js
function calculatePrice(price, tax = 0.18) {
  return price + price * tax;
}

calculatePrice(1000);
calculatePrice(1000, 0.1);
```

## Parameters Can Use Earlier Parameters

```js
function calculateTotal(price, quantity = 1, total = price * quantity) {
  return total;
}

console.log(calculateTotal(100, 3)); // 300
```

## Objects as Arguments

Real application functions often receive objects.

```js
function printUser(user) {
  console.log(user.name);
}

const user = {
  name: "Vikash",
  role: "Developer",
};

printUser(user);
```

Remember from Lesson 5: JavaScript is pass-by-value. When an object is supplied, the value being passed is a reference to that object.

Therefore:

```js
function changeName(user) {
  user.name = "Kumar";
}

const user = {
  name: "Vikash",
};

changeName(user);

console.log(user.name); // Kumar
```

Both the caller and function can access the same object.

## The arguments Object

Traditional functions have an array-like `arguments` object containing supplied arguments.

```js
function show() {
  console.log(arguments[0]);
  console.log(arguments[1]);
}

show("JavaScript", "React");
```

However, modern JavaScript generally prefers **rest parameters** when a function needs a variable number of arguments.

Arrow functions do not have their own `arguments` object.

## Common Mistake: Assuming Default Handles null

```js
function greet(name = "Guest") {
  console.log(name);
}

greet(undefined); // Guest
greet(null);      // null
```

A default parameter is applied for `undefined`, not for every falsy value.

## Interview Perspective

**What is the difference between parameters and arguments?**

Parameters are variables declared in the function definition. Arguments are the actual values supplied when the function is called.

**When are default parameters used?**

A default parameter is used when an argument is omitted or its value is `undefined`.

## Key Takeaways

- Parameters belong to the function definition.
- Arguments are values supplied during invocation.
- Missing arguments normally produce `undefined`.
- JavaScript allows extra arguments.
- Default parameters provide fallback values.
- Traditional functions provide `arguments`, while rest parameters are generally preferred in modern code.
