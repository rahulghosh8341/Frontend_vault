---
title: Curry III
aliases:
  - Curry III
difficulty: Hard
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Currying]]"
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[Symbol.toPrimitive]]"
  - "[[Function.prototype.bind]]"
  - "[[Type Coercion]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-13
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 20 min
> Implement dynamic currying that allows infinite chained calls and evaluates when coerced to primitive (number/string).

## Problem

Implement `curry(func)` returning a function callable indefinitely with arbitrary arguments per call. When coerced to a primitive (e.g. `+curried(1)(2)` or `"" + curried(1, 2)`), invoke `func` with all accumulated arguments.

```js
function multiply(...numbers) {
  return numbers.reduce((a, b) => a * b, 1);
}
const curried = curry(multiply);
+curried(3);         // 3
+curried(3)(4);      // 12
+curried(1, 2, 3, 4); // 24
```

## Pattern

- [[Currying]]
- [[Closure]]

## 🤔 Thought Process

- Unlike Curry I & II, there is no fixed arity `func.length` to terminate on (`multiply` uses rest parameter `...numbers`, so `func.length === 0`).
- Every invocation must return a function so chaining never stops: `fn(1)(2)...(n)`.
- Evaluation trigger is JavaScript type coercion: when used in string/numeric context (`+fn`, `${fn}`, `fn == 10`).
- Use `Symbol.toPrimitive` on the returned function to intercept primitive conversion and execute `func.apply(this, args)`.
- Use `curried.bind(this, ...args)` to cleanly pre-pend accumulated arguments for the next function call.

## 💻 Final Solution

```js
export default function curry(func) {
  return function curried(...args) {
    // Each call returns another function that remembers all arguments seen so far.
    const fn = curried.bind(this, ...args);

    // Numeric/string coercion becomes the signal to evaluate the accumulated arguments.
    fn[Symbol.toPrimitive] = () => func.apply(this, args);
    return fn;
  };
}
```

## 🤔 Why This Works

- `curried.bind(this, ...args)` produces a new function with accumulated arguments pre-applied. When that returned function is invoked later with new arguments, `bind` automatically appends them after the preset `args`.
- In JS, functions are objects. Properties can be assigned to them directly.
- `Symbol.toPrimitive` is a well-known Symbol called by JS engine when converting an object to a primitive value. It takes precedence over `valueOf()` and `toString()`.
- Coercions like `+fn` trigger `Symbol.toPrimitive("number")`, which calls `func.apply(this, args)` and evaluates the result.

## 🐞 Bugs I Made

None.

## Production Considerations

- `Symbol.toPrimitive` is standard ES6+ behavior; for legacy ES5 environments, define both `valueOf` and `toString` methods on `fn`.
- Value cannot be accessed directly without coercion (e.g. `curried(1)(2)` is still a function, not a number).
- Primarily a brain teaser/interview problem testing deep language internals (`bind`, Symbol coercion protocol, function object nature).

## ⭐ Revision Notes

### Key Facts

- `Symbol.toPrimitive(hint)` handles coercion for `"number"`, `"string"`, and `"default"` hints.
- Functions are objects in JS; can hold custom properties and symbols.
- `fn.bind(this, ...args)` performs partial application natively.
- Solves infinite currying without explicit termination signal like empty call `()`.

### Common Interview Questions

- How does `Symbol.toPrimitive` work? Takes precedence over `valueOf()` and `toString()` when object undergoes primitive conversion.
- What if target environment does not support ES6 Symbols? Assign `fn.valueOf = fn.toString = () => func.apply(this, args)`.
- How does `bind` help with currying? Pre-fills initial arguments, returning a new function ready to accept remaining arguments.

### 🧠 Mental Model

```text
curried(1, 2)
     ↓
fn = curried.bind(this, 1, 2)
fn[Symbol.toPrimitive] = () => func(1, 2)
     ↓
+fn → triggers Symbol.toPrimitive → evaluates func(1, 2)
```

### Interview Takeaways

- Infinite chaining requires deferred evaluation signal.
- Two standard signals in JS:
  1. Empty call `fn()()`
  2. Coercion via `Symbol.toPrimitive` / `valueOf`

### Related

- [[Currying]]
- [[Curry]]
- [[Curry II]]
- [[Closure]]
- [[Function.prototype.bind]]
