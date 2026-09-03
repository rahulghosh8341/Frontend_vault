---
title: Explain the concept of "Hoisting" in JavaScript
aliases:
  - Hoisting
tags:
  - javascript
  - interview
  - hoisting
  - execution-context
solved: true
solvedDate: 2026-08-09
type: quiz
---

> [!info]
> 🟢 Difficulty: Easy
> 📂 Category: JavaScript Interview Question
> ⏱️ Review Time: ~3 minutes

---

## TL;DR

Hoisting is JavaScript's behavior during compilation where declarations become available before execution begins.

- `var` → Hoisted and initialized to `undefined`
- `let` / `const` → Hoisted but remain in the Temporal Dead Zone (TDZ)
- Function declarations → Fully hoisted
- Function expressions → Only the variable binding is hoisted
- Classes → Hoisted but remain in TDZ
- Imports → Hoisted and evaluated before module execution

---

## Interview Answer (30–60 sec)

> Hoisting refers to JavaScript creating bindings for declarations before executing the code. It doesn't literally move code. `var` variables are initialized as `undefined`, `let` and `const` remain in the Temporal Dead Zone until execution reaches their declaration, function declarations are completely hoisted, while function expressions only hoist the variable binding.

---

## Key Takeaways

| Declaration | Before Declaration |
|-------------|-------------------|
| `var` | `undefined` |
| `let` | `ReferenceError` |
| `const` | `ReferenceError` |
| `class` | `ReferenceError` |
| Function Declaration | Works |
| Function Expression (`var`) | `undefined` |
| Import | Works |

---

## Visual Model

```text
Compilation Phase
│
├── Create Scope
│
├── var
│      ↓
│   undefined
│
├── let / const
│      ↓
│     TDZ
│
├── function
│      ↓
│ Full Function Object
│
├── class
│      ↓
│     TDZ
│
└── import
       ↓
   Module Loaded

↓

Execution Phase

Assignments happen
```

---

## Common Interview Traps

❌ "`let` isn't hoisted."

✔ It is hoisted but remains inside the Temporal Dead Zone.

---

❌ "Only `var` is hoisted."

✔ Everything is hoisted; initialization differs.

---

❌ "Function expressions are hoisted."

✔ Only the variable binding is hoisted.

---

❌ "`typeof` never throws."

✔ It throws inside the TDZ.

---

## Common Follow-ups

- What is the Temporal Dead Zone?
- Difference between `var`, `let`, and `const`?
- Why are function declarations callable before definition?
- Difference between declaration and initialization?
- Why should `var` be avoided?

---

## My Notes

- 
-  Hoisting
- 

---

# Detailed Reference

## Hoisting

Hoisting is a term used to explain the behavior of declarations in JavaScript code.

Variables declared with the `var` keyword have their declaration "moved" up to the top of their containing scope during compilation.

Only the declaration is hoisted. Initialization remains where it is written.

### Hoisting of `var`

```js
console.log(foo); // undefined
var foo = 1;
console.log(foo); // 1
```

Equivalent to

```js
var foo;

console.log(foo);

foo = 1;

console.log(foo);
```

---

### Hoisting of `let`, `const`, and `class`

These declarations are hoisted but remain inside the **Temporal Dead Zone (TDZ)** until execution reaches the declaration.

```js
console.log(a);

let a = 10;
```

Throws

```
ReferenceError
```

Same applies to

```js
const a = 10;

class Foo {}
```

---

### Hoisting of Function Expressions

Only the variable declaration is hoisted.

```js
console.log(fn);

fn();

var fn = function () {};
```

Output

```
undefined

TypeError
```

Arrow functions follow the same rule.

---

### Hoisting of Function Declarations

Entire function is hoisted.

```js
sayHello();

function sayHello() {
    console.log("Hello");
}
```

Works normally.

---

### Hoisting of Imports

```js
foo();

import foo from "./foo.js";
```

Imports are hoisted and executed before the rest of the module.

---

## Under the Hood

JavaScript executes in two phases.

### Compilation

- Creates scopes
- Registers declarations
- Initializes bindings

### Execution

Runs code line-by-line.

Initialization during compilation:

| Declaration | Initial Value |
|-------------|---------------|
| `var` | `undefined` |
| `let` | TDZ |
| `const` | TDZ |
| `class` | TDZ |
| Function | Function Object |
| Import | Loaded |

MDN classifies hoisting into four observable behaviors:

1. Value hoisting
2. Declaration hoisting
3. Scope tainting (TDZ)
4. Import side effects

---

## Modern Practices

- Prefer `const`
- Use `let` when reassignment is required
- Avoid `var`
- Declare variables near the top of their scope
- Enable ESLint rules:
  - `no-use-before-define`
  - `no-undef`

---

## Additional Examples

### Function Declaration vs Function Expression

```js
console.log(declared());

console.log(expressed());

function declared() {
    return "Function";
}

var expressed = function () {
    return "Expression";
};
```

Function declaration works.

Function expression throws **TypeError**.

---

### `var` with `setTimeout`

```js
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 0);
}
```

Output

```
3
3
3
```

Fix:

- Use `let`
- Or capture `i` using an IIFE.

---

### `var` Escapes Block Scope

```js
if (true) {
    var a = 1;
    let b = 2;
}

console.log(a);

console.log(b);
```

Output

```
1

ReferenceError
```

---

### Redeclaration

```js
var x = 1;

var x = 2;
```

Allowed.

```js
let y = 1;

let y = 2;
```

Throws `SyntaxError`.

---

### Class Declarations

```js
console.log(typeof Foo);

class Foo {}
```

Throws

```
ReferenceError
```

Classes are hoisted but remain inside the TDZ.

---

### `typeof` and TDZ

```js
console.log(typeof undeclaredVariable);

console.log(typeof someLet);

let someLet = 1;
```

Output

```
"undefined"

ReferenceError
```

---

### Shared `var` and Function Name

```js
function outer() {
    console.log(inner);

    inner();

    function inner() {}

    var inner = "changed";
}
```

Function declaration takes precedence until assignment happens.

---

## Common Misconceptions

- Classes are not hoisted.
- `let` is not hoisted.
- `typeof` never throws.
- Function expressions are completely hoisted.

All of the above are incorrect.

---

## Related Concepts

- [[Execution Context]]
- [[Scope]]
- [[Temporal Dead Zone]]
- [[Closures]]
- [[Functions]]
- [[Modules]]
- [[var vs let vs const]]

---

## Related Interview Questions

- [[Explain the Temporal Dead Zone]]
- [[Difference between var, let and const]]
- [[Explain Execution Context]]
- [[Explain JavaScript Scope]]

---

## Further Reading

- MDN – Hoisting
- ECMAScript Specification
- GreatFrontend Explanation
