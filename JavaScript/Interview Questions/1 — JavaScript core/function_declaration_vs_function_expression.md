---
title: Explain the differences in usage between `function foo() {}` and `var foo = function() {}`
aliases: function declaration vs function expression
tags:
  - functions
  - hoisting
  - scope
section: "1 — JavaScript core"
solved: true
type: quiz
solvedDate: 2026-09-07
---

> [!info] 🟢 Difficulty: Medium 📂 Category: JavaScript Interview Question ⏱️ Review Time: ~5 minutes

## TL;DR

`function foo() {}` is a **function declaration**, while `var foo = function() {}` is a **function expression** assigned to a variable.

The most important difference is **hoisting**:

- Function declarations can be called before their declaration in the enclosing scope.
- With `var foo = function() {}`, the `var` binding is hoisted and initialized to `undefined`; the function value is assigned only when execution reaches the assignment, so an earlier call throws `TypeError: foo is not a function`.
- Named function expressions have an additional name-scope rule: the function's internal name is accessible inside the function itself, but not from the surrounding scope.

## Interview Answer (30–60 sec)

A function declaration and a function expression both create functions, but they differ mainly in hoisting and naming scope. A function declaration like `function foo() {}` can be called before the declaration because the function definition is available when the scope is initialized. With `var foo = function() {}`, only the `var` binding is hoisted and initialized to `undefined`, so calling `foo()` before the assignment throws a `TypeError`. Function expressions can also be named, such as `const myFunc = function namedFunc() {}`, but `namedFunc` is only accessible inside the function itself. Function declarations are commonly used for normal reusable functions; function expressions are useful when a function is being assigned or passed as a value.

## Key Takeaways

### Function declaration

```js
foo(); // works

function foo() {
  console.log('FOOOOO');
}
```

The declaration is available before its textual position in the enclosing scope.

### Function expression with `var`

```js
foo(); // TypeError: foo is not a function

var foo = function () {
  console.log('FOOOOO');
};
```

Conceptually:

```js
var foo; // undefined
foo();   // TypeError
```

The function is assigned later.

### `let` and `const` function expressions

Function expressions can also use `let` and `const`:

```js
const foo = function () {
  console.log('FOOOOO');
};
```

`let` and `const` have their own hoisting and temporal-dead-zone behavior.

### Named function expression

```js
const myFunc = function namedFunc() {
  console.log(namedFunc); // works
};

myFunc();
console.log(namedFunc); // ReferenceError
```

`namedFunc` is available inside the function itself, but not as a name in the surrounding scope.

## 🧠 Mental Model

```text
function foo() {}
        ↓
function declaration
        ↓
callable before declaration

var foo = function () {}
        ↓
var binding exists first
        ↓
foo is undefined until assignment
```

For a named function expression:

```text
const myFunc = function namedFunc() {
                              ↑
                       function-local name
};
```

Think:

```text
function declaration
→ name belongs to the enclosing scope

named function expression
→ expression variable belongs to enclosing scope
→ function name is available inside the function
```

## Common Interview Traps

### "Function expressions are not hoisted at all."

Too broad.

For:

```js
var foo = function () {};
```

the `var foo` binding is hoisted, but the function value is assigned only when execution reaches the assignment.

For `let`/`const`, the binding exists but remains in the temporal dead zone until initialization.

### "Function expressions and function declarations behave the same."

They do not.

This works:

```js
foo();
function foo() {}
```

But this fails:

```js
foo();
var foo = function () {};
```

### "The name of a named function expression is available everywhere."

It is not:

```js
const myFunc = function namedFunc() {};

myFunc();       // works
namedFunc();    // ReferenceError
```

`namedFunc` is scoped to the function itself.

## Common Follow-ups

- What is hoisting in JavaScript?
- What is the difference between function declarations and function expressions?
- What exactly gets hoisted with `var`?
- What happens when a `var` function expression is called before assignment?
- How do `let` and `const` change the behavior?
- What is the temporal dead zone?
- Why would you use a named function expression?
- Where is the name of a named function expression accessible?
- Are arrow functions function declarations or function expressions?
- What is the difference between a function declaration and a function expression in terms of scope?

## My Notes

- **Remember:** declaration vs expression.
- **Most important interview point:** function declaration can be called before its declaration; `var` function expression cannot.
- **Named function expression:** internal function name is available inside the function, not outside.
- Don't describe function expressions as simply "not hoisted"; distinguish the variable binding from the function value.

---

# Detailed Reference

## TL;DR

`function foo() {}` is a function declaration while `var foo = function() {}` is a function expression. The key difference is that function declarations have their bodies hoisted but the bodies of function expressions are not (they have the same hoisting behavior as `var`-declared variables).

If you try to invoke a function expression before it is declared, you will get an `Uncaught TypeError: XXX is not a function` error. Function declarations can be called in the enclosing scope even before they are declared.

```js
foo(); // 'FOOOOO'
function foo() {
  console.log('FOOOOO');
}
```

Function expressions if called before they are declared will result in an error.

```js
foo(); // Uncaught TypeError: foo is not a function
var foo = function () {
  console.log('FOOOOO');
};
```

Another key difference is in the scope of the function name. Function expressions can be named by defining a name after the `function` keyword and before the parentheses. However, when using named function expressions, the function name is only accessible within the function itself. Trying to access it outside will result in a `ReferenceError`.

```js
const myFunc = function namedFunc() {
  console.log(namedFunc); // Works
};

myFunc(); // Runs the function and logs the function reference
console.log(namedFunc); // ReferenceError: namedFunc is not defined
```

Note: The examples use `var` due to legacy reasons. Function expressions can be defined using `let` and `const`, and the key difference is in the hoisting behavior of those keywords.

---

## Function declarations

A function declaration is a statement that defines a function with a name. It is typically used to declare a function that can be called multiple times throughout the enclosing scope.

```js
function foo() {
  console.log('FOOOOO');
}
foo(); // 'FOOOOO'
```

## Function expressions

A function expression is an expression that defines a function and assigns it to a variable. It is often used when a function is needed only once or in a specific context.

```js
var foo = function () {
  console.log('FOOOOO');
};
foo(); // 'FOOOOO'
```

Note: The examples use `var` due to legacy reasons. Function expressions can be defined using `let` and `const`, and the key difference is in the hoisting behavior of those keywords.

## Key differences

### Hoisting

The key difference is that function declarations have their bodies hoisted but the bodies of function expressions are not (they have the same hoisting behavior as `var`-declared variables). If you try to invoke a function expression before it is defined, you will get an `Uncaught TypeError: XXX is not a function` error.

Function declarations:

```js
foo(); // 'FOOOOO'
function foo() {
  console.log('FOOOOO');
}
```

Function expressions:

```js
foo(); // Uncaught TypeError: foo is not a function
var foo = function () {
  console.log('FOOOOO');
};
```

### Name scope

Function expressions can be named by defining a name after the `function` keyword and before the parentheses. However, when using named function expressions, the function name is only accessible within the function itself. Trying to access it outside will result in a `ReferenceError`.

```js
const myFunc = function namedFunc() {
  console.log(namedFunc); // Works
};

myFunc(); // Runs the function and logs the function reference
console.log(namedFunc); // ReferenceError: namedFunc is not defined
```

## When to use each

- Function declarations:
  - When you want to create a function on the global scope and make it available throughout the enclosing scope.
  - If a function is reusable and needs to be called multiple times.
- Function expressions:
  - If a function is only needed once or in a specific context.
  - Use to limit the function's availability to subsequent code and keep the enclosing scope clean.

In general, it's preferable to use function declarations to avoid the mental overhead of determining if a function can be called. The practical usages of function expressions are quite rare.

## Further reading

- Function declaration | MDN
- Function expression | MDN
