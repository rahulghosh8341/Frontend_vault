---
title: Make Counter II
aliases:
  - Make Counter II
difficulty: Medium
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
pattern:
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-12
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 10 min
> Implement `makeCounter` returning an object with `get`, `increment`, `decrement`, `reset` methods.

## Problem

Return an object with four methods sharing closure state over a counter value. `reset()` restores to initial value.

```js
const counter = makeCounter(5);
counter.get();       // 5
counter.increment(); // 6
counter.decrement(); // 5
counter.reset();     // 5
```

## Companies

- [[Amazon]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

- Closure over `count` for shared mutable state.
- `initialValue` preserved for `reset`.
- Arrow functions fine here, no `this` dependency.
- `++count` and `--count` (pre-increment/decrement) mutate then return, matching expected behavior.
- `reset` uses assignment expression `(count = initialValue)` to assign and return in one step.

## 💻 Final Solution

```js
export default function makeCounter(initialValue = 0) {
  let count = initialValue;
  return {
    get: () => count,
    increment: () => ++count,
    decrement: () => --count,
    reset: () => (count = initialValue),
  };
}
```

## 🤔 Why This Works

- All four methods close over same `count` variable. Mutations visible across methods.
- `initialValue` param never mutated, always available for `reset`.
- Pre-increment `++count` mutates first, returns new value. Correct for increment/decrement.
- `(count = initialValue)` is assignment expression: assigns and evaluates to assigned value.

## 🐞 Bugs I Made

None.

## Production Considerations

- Could use `class` with `#private` fields for same encapsulation. Closure approach more functional.
- No validation on `initialValue` type. Production code might `Number()` coerce or throw.

## ⭐ Revision Notes

### Key Facts

- Closure = private state shared across returned object methods
- Pre-increment `++x` for mutate-and-return
- Assignment expression `(x = val)` returns assigned value
- `initialValue` param acts as immutable reset target

### Common Interview Questions

- Why pre-increment not post? `++count` returns new value. `count++` would return old value before mutation.
- Why parentheses in `(count = initialValue)`? Without parens, arrow function would treat `=` as function body assignment without return. Parens make it expression returning value.
- Difference from Make Counter I? Returns object with multiple methods vs single function. Demonstrates closure shared across multiple functions.

### Interview Takeaways

- Module pattern: closure + object literal = encapsulated API
- Same closure, multiple accessors = shared private state
- Assignment expressions useful in concise arrow functions

### Related

- [[Closure]]
- [[Make Counter]]
- [[Once]]
