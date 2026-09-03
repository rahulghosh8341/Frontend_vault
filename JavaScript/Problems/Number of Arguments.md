---
title: Implement a function `numberOfArguments` that returns the number of arguments it was called with
aliases:
  - Number of Arguments
difficulty: Easy
time: 5 min
languages:
  - JavaScript
concepts:
  - "[[Function Properties]]"
solved: true
solvedDate: 2026-08-19
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 5 min
> Returns how many arguments were actually passed at call time — the call-site arity.

## Problem

Implement `numberOfArguments()` that returns the count of arguments it was invoked with. This value can be more or less than the declared parameter count.

```js
numberOfArguments();          // 0
numberOfArguments(1);         // 1
numberOfArguments(2, 3);      // 2
numberOfArguments('a','b','c'); // 3
```

---

## 🤔 Thought Process

1. Need the **runtime** argument count, not the declared parameter count.
2. `arguments.length` gives exactly that — available inside any regular function regardless of how many parameters are declared.
3. Alternative: rest parameter `...args` captures all arguments into a real array; `args.length` is equivalent and works in arrow functions too.

---

## 💻 Final Solution

```js
// Approach 1 — arguments object
export default function numberOfArguments() {
  return arguments.length;
}

// Approach 2 — rest parameters (modern, works in arrow functions)
export default function numberOfArguments(...args) {
  return args.length;
}
```

---

## 🤔 Why This Works

`arguments.length` reflects **what the caller supplied**, not what the function declared. The value of each argument is irrelevant — `null`, `undefined`, `NaN`, spread values all count once per position.

| Call | Count |
|---|---|
| `numberOfArguments()` | 0 |
| `numberOfArguments(undefined)` | 1 |
| `numberOfArguments(...[1, 2, 3])` | 3 |

**Key trap — default parameters don't affect `arguments.length`:**

```js
function foo(a = 1, b = 2) {
  return arguments.length;
}
foo();          // 0  — nothing was passed
foo(undefined); // 1  — undefined was explicitly passed
```

**`arguments` vs rest params:**

| | `arguments` | `...args` |
|---|---|---|
| Works in arrow functions | ❌ No | ✅ Yes |
| Real array | ❌ Array-like | ✅ Yes |
| Modern preference | ❌ | ✅ |

---

## 🐞 Bugs I Made

None.

---

## ⭐ Revision Notes

### Mental Model

```
fn.length        → "how was the function written?"   (static, at parse time)
arguments.length → "how was the function called?"    (dynamic, at call time)
```

### Key Facts

- `arguments` is not available in arrow functions — use rest params there
- Passing `undefined` **does** count as one argument
- Rest params `...args` is the modern, preferred approach
- This pattern is foundational for variadic utilities like **[[Curry II]]** and **[[Classnames]]**

### Common Interview Questions

- What's the difference between `fn.length` and `arguments.length`? → Static vs runtime arity
- Does passing `undefined` count as an argument? → Yes, always
- Can you use `arguments` in an arrow function? → No, use rest params

### Related

- [[Function Length]]
- [[Function Properties]]
- [[Curry II]]
- [[Classnames]]
