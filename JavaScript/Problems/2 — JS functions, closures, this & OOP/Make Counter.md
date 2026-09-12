---
title: Make Counter
aliases:
  - Make Counter
difficulty: Easy
time: 5 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
  - "[[Robinhood]]"
  - "[[Roblox]]"
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
> **Difficulty:** 🟢 Easy | **Time:** 5 min
> Implement `makeCounter` that returns a function incrementing from an optional initial value.

## Problem

Return a function that starts at `initialValue` (default 0) and returns one more each call.

```js
const counter = makeCounter(5);
counter(); // 5
counter(); // 6
counter(); // 7
```

## Companies

- [[Amazon]]
- [[Robinhood]]
- [[Roblox]]

## Pattern

- [[Closure]]

## 🤔 Thought Process

- Default param `initialValue = 0` handles optional argument.
- Closure over mutable `result` variable.
- Post-increment `result++` returns current value then increments for next call.

## 💻 Final Solution

```js
export default function makeCounter(initialValue = 0) {
  let result = initialValue;
  return function(){
    return result++;
  }
}
```

## 🤔 Why This Works

- `result++` (post-increment) returns value before incrementing. First call returns `initialValue`, next returns `initialValue + 1`, etc.
- Each `makeCounter()` call creates new closure scope with independent `result`. Counters do not share state.

## 🐞 Bugs I Made

None.

## Production Considerations

- Pre-increment `++result` would start at `initialValue + 1` on first call. Post-increment is correct here.

## ⭐ Revision Notes

### Key Facts

- Post-increment `x++` returns value then increments
- Pre-increment `++x` increments then returns value
- Each closure gets independent copy of enclosed variables

### Common Interview Questions

- Why `result++` not `++result`? Post-increment returns current value before mutation, matching expected first-call behavior.
- Are counters independent? Yes, each `makeCounter()` creates new closure scope.

### Interview Takeaways

- Classic closure-as-private-state pattern
- Post vs pre increment matters for return value

### Related

- [[Closure]]
- [[Make Counter II]]
- [[Once]]
