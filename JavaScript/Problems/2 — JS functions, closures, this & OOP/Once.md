---
title: Once
aliases:
  - Once
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
pattern:
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[Function.prototype.apply]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-12
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Implement a function that accepts a callback and restricts its invocation to at most once.

## Problem

Implement `once(func)` that returns a new function. First call executes `func` with correct `this` and arguments, caches result. All subsequent calls return cached result without calling `func` again.

```js
const fn = once(x => x * 2);
fn(5); // 10
fn(9); // 10 (cached)
```

## Companies

- [[Amazon]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

- Need boolean flag to track if function already called.
- Need variable to store first invocation result.
- Return wrapper using rest params `...value`.
- On first call: run `func.apply(this, value)`, store result, flip flag.
- On subsequent calls: skip execution, return stored result.
- `apply` preserves `this` binding from call site.

## 💻 Final Solution

```js
export default function once(func) {
  let counter = false;
  let result;

  return function(...value){
    if(!counter){
      result = func.apply(this,value);
      counter = true;
    }
    return result;
  }
}
```

## 🤔 Why This Works

- Closure captures `counter` and `result` across calls.
- `counter` acts as guard: only first call passes `if` check.
- `func.apply(this, value)` forwards both `this` context and all arguments.
- Regular `function` (not arrow) needed so `this` reflects caller's context.
- After first call, `result` holds cached return value permanently.

## 🐞 Bugs I Made

None.

## Production Considerations

- Lodash `_.once` works same way.
- Could use `func` itself as flag: set `func = null` after first call, check `if (func)`. Saves one variable.
- No memory cleanup: `result` held forever. If result is large object, consider WeakRef in long-lived apps.

## ⭐ Revision Notes

### Key Facts

- Closure + boolean flag = run-once pattern
- `func.apply(this, args)` preserves `this` binding
- Must use regular `function`, not arrow, for dynamic `this`
- Cached result returned on all subsequent calls

### Common Interview Questions

- Why `apply` instead of direct call? Preserves `this` binding from call site.
- Why regular function not arrow? Arrow captures lexical `this`, cannot reflect caller's `this`.
- What if callback returns `undefined`? Still works, flag controls execution not return value.

### Interview Takeaways

- Classic closure + state pattern
- `this` forwarding with `apply`/`call` is critical for higher-order function wrappers
- Same pattern extends to `memoize`, `throttle`, `debounce`

### Related

- [[Closure]]
- [[Function Chaining]]
- [[Debouncing]]
