---
title: What's the difference between a JavaScript variable that is `null`, `undefined` or undeclared?
subtitle: How would you go about checking for any of these states?
aliases:
  - null vs undefined vs undeclared
tags:
  - javascript
  - interview
  - variables
  - 
  - undefined
  - scope
solved: true
solvedDate: 2026-08-26
type: quiz
---

> [!info]
> 🟢 Difficulty: Medium
> 📂 Category: JavaScript Interview Question
> ⏱️ Review Time: ~5 minutes

---

## TL;DR

There are three distinct states:

| Trait | `null` | `undefined` | Undeclared |
|---|---|---|---|
| Meaning | Explicitly set by the developer to indicate that a variable has no value | Variable has been declared but not assigned a value | Variable has not been declared at all |
| Type via `typeof` | `'object'` | `'undefined'` | `'undefined'` |
| Equality comparison | `null == undefined` is `true` | `undefined == null` is `true` | Throws a `ReferenceError` |

The important distinction is:

```text
null
→ variable exists
→ explicitly assigned "no value"

undefined
→ variable exists
→ declared but has no assigned value

undeclared
→ variable does not exist
→ no declaration was made
```

---

## Interview Answer (30–60 sec)

> `null`, `undefined`, and undeclared are different states. `null` is an explicit value assigned by the developer to represent no value. `undefined` means the variable has been declared but hasn't been assigned a value, and it can also be returned when a function doesn't explicitly return anything. An undeclared variable has never been declared with `var`, `let`, or `const`; trying to access it normally causes a `ReferenceError`. For checking `undefined`, I would use strict equality or `typeof`. For `null`, I would use strict equality with `null`. For an undeclared identifier, `typeof` is useful because it safely returns `'undefined'` instead of throwing exception.

---

## Key Takeaways

### `null`

```js
const value = null;

value === null; // true
typeof value;   // "object"
```

`null` is an explicit value.

Think:

```text
"I know this variable exists,
but I intentionally want it to have no value."
```

---

### `undefined`

```js
let value;

value === undefined;       // true
typeof value === 'undefined'; // true
```

The variable exists, but no value has been assigned.

A function with no explicit return also produces `undefined`:

```js
function foo() {}

const result = foo();

result === undefined; // true
```

---

### Undeclared

```js
console.log(something);
```

If `something` has never been declared, directly accessing it throws:

```text
ReferenceError
```

However:

```js
typeof something === 'undefined'; // true
```

`typeof` can safely check an undeclared identifier.

---

## 🧠 Mental Model

```text
                 Does the variable exist?
                         │
              ┌──────────┴──────────┐
             NO                     YES
              │                      │
        UNDECLARED             Is it assigned?
              │                      │
        typeof → "undefined"    ┌────┴────┐
                                 NO       YES
                                 │          │
                           UNDEFINED     What value?
                                            │
                                      ┌─────┴─────┐
                                    null        other
```

The key distinction:

```text
undeclared ≠ undefined
```

An undeclared identifier does not have a variable binding.

An `undefined` variable is a valid declared variable whose value is `undefined`.

---

## Common Interview Traps

### "`typeof null` returns `'null'`."

❌ Incorrect.

```js
typeof null; // "object"
```

This is a long-standing JavaScript quirk.

---

### "`null` and `undefined` mean exactly the same thing."

❌ Not exactly.

```js
const a = null;

let b;

a; // null
b; // undefined
```

`null` is explicitly assigned.

`undefined` commonly represents an unassigned value.

---

### "An undeclared variable is just another undefined variable."

❌ Incorrect.

```js
let foo;

foo; // undefined
```

But:

```js
bar; // ReferenceError
```

if `bar` was never declared.

---

### "`typeof undeclaredVariable` throws a ReferenceError."

❌ Incorrect.

```js
typeof undeclaredVariable; // "undefined"
```

This is one of the special useful behaviors of `typeof`.

---

### "Use loose equality to check for undefined."

❌ Avoid this when specifically checking for `undefined`.

```js
value == null;
```

is also true when `value` is `null`.

Prefer:

```js
value === undefined;
```

or:

```js
typeof value === 'undefined';
```

---

### "Use loose equality to check for null."

❌ Avoid this when specifically checking for `null`.

```js
value == undefined;
```

is also true when `value` is `null`.

Prefer:

```js
value === null;
```

---

## Common Follow-ups

- What is the difference between `null` and `undefined`?
- Why does `typeof null` return `"object"`?
- How do you check whether a variable is `undefined`?
- How do you safely check whether an identifier is undeclared?
- Why does `typeof` not throw for undeclared variables?
- What happens when you access an undeclared variable?
- What is the difference between undeclared and uninitialized?
- What happens when a function doesn't return a value?
- Why should you avoid undeclared variables?
- What is the difference between `undefined` and the Temporal Dead Zone?
- How does strict mode affect undeclared variables?

---

## My Notes

- 

---

# Detailed Reference

## TL;DR

| Trait | `null` | `undefined` | Undeclared |
| --- | --- | --- | --- |
| Meaning | Explicitly set by the developer to indicate that a variable has no value | Variable has been declared but not assigned a value | Variable has not been declared at all |
| Type (via `typeof` operator) | `'object'` | `'undefined'` | `'undefined'` |
| Equality Comparison | `null == undefined` is `true` | `undefined == null` is `true` | Throws a `ReferenceError` |

* * *

## Undeclared

Undeclared variables are created when you assign a value to an identifier that is not previously created using `var`, `let` or `const`. Undeclared variables will be defined globally, outside of the current scope. In strict mode, a `ReferenceError` will be thrown when you try to assign to an undeclared variable. Undeclared variables are bad in the same way that global variables are bad. Avoid them at all costs! To check for them, wrap its usage in a `try`/`catch` block.

```js
function foo() {
  x = 1; // Throws a ReferenceError in strict mode
}

foo();
console.log(x); // 1 (if not in strict mode)
```

Using the `typeof` operator on undeclared variables will give `'undefined'`.

```js
console.log(typeof y === 'undefined'); // true
```

## `undefined`

A variable that is `undefined` is a variable that has been declared, but not assigned a value. It is of type `undefined`. If a function does not return a value, and its result is assigned to a variable, that variable will also have the value `undefined`. To check for it, compare using the strict equality ` _=== ` operator or `typeof` which will give the `'undefined'` string. Note that you should not be using the loose equality operator (` _== `) to check, as it will also return `true` if the value is `null`.
```js
let foo;
console.log(foo); // undefined
console.log(foo === undefined); // true
console.log(typeof foo === 'undefined'); // true

console.log(foo == null); // true. Wrong, don't use this to check if a value is undefined!

function bar() {} // Returns undefined if there is nothing returned.
let baz = bar();
console.log(baz); // undefined
```

## `null`

A variable that is `null` will have been explicitly assigned to the `null` value. It represents no value and is different from `undefined` in the sense that it has been explicitly assigned. To check for `null`, simply compare using the strict equality operator. Note that like the above, you should not be using the loose equality operator (`_==`) to check, as it will also return `true` if the value is `undefined`.

```js
const foo = null;
console.log(foo === null); // true
console.log(typeof foo === 'object'); // true

console.log(foo == undefined); // true. Wrong, don't use this to check if a value is null!
```

## Notes

- As a good habit, never leave your variables undeclared or unassigned. Explicitly assign `null` to them after declaring if you don't intend to use them yet.
- Always explicitly declare variables before using them to prevent errors.
- Using some static analysis tooling in your workflow (e.g. ESLint, TypeScript Compiler), will enable checks that you are not referencing undeclared variables.

## Practice

Practice implementing type utilities that check for null and undefined on GreatFrontEnd.

## Further Reading

- MDN Web Docs: null
- MDN Web Docs: undefined
- MDN Web Docs: ReferenceError
