---
title: Implement a function `functionLength` to return the number of parameters a function expects
aliases:
  - Function Length
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
> Returns the number of parameters a function was declared with — its static arity.

## Problem

Implement `functionLength(fn)` that returns the number of parameters `fn` was declared with — its **static arity**.

```js
functionLength(foo);    // 0  — function foo() {}
functionLength(bar);    // 1  — function bar(a) {}
functionLength(baz);    // 2  — function baz(a, b) {}
```

> Note: this is a **static value** from the function signature, not the number of arguments passed at call time.

---

## 🤔 Thought Process

1. Every function object in JavaScript has a built-in `length` property.
2. Understand exactly what it measures — it is **not** runtime arguments.
3. Read it directly. No invocation, wrapping, or source-parsing needed.

---

## 💻 Final Solution

```js
export default function functionLength(fn) {
  return fn.length;
}
```

---

## 🤔 Why This Works

`fn.length` is **static metadata** attached to the function object by the JS engine at parse time. It measures **declared arity** — specifically:

| Rule | Effect |
|---|---|
| Counts required parameters | `(a, b)` → `2` |
| Stops before the first default | `(a, b = 1)` → `1` |
| Excludes rest parameters | `(...args)` → `0` |

The critical distinction the interviewer is testing:

```
fn.length         → declared arity (known when function is created)
arguments.length  → call-site arity (only exists inside a running invocation)
```

The helper must never **call** `fn`, wrap it, or inspect `arguments` — those are runtime concepts. `fn.length` is purely observational.

---

## 🐞 Bugs I Made

None.

---

## ⭐ Revision Notes

### Key Facts

- `fn.length` = parameters **before** the first default value
- `fn.length` = 0 if the first parameter has a default, or if only rest params exist
- `arguments.length` ≠ `fn.length` — completely different timelines

### Mental Model

```
fn.length     = "how was this function written?"   (static, at parse time)
arguments.length = "how was this function called?"  (dynamic, at call time)
```

### Common Interview Questions

- What does `fn.length` count? → Required params before the first default
- Does `fn.length` include rest parameters? → No
- What's the difference between `fn.length` and `arguments.length`? → Static vs runtime

### Interview Takeaways

- This is foundational for implementing **[[Curry II]]** — currying relies on `fn.length` to know when enough arguments have been collected.
- Transpilers (Babel) can rewrite default parameters, which is why tests focus on ordinary functions.

### Related

- [[Curry II]]
- [[Function Properties]]
