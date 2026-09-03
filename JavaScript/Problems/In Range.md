---
title: Implement a function that checks if a number is within a given range
aliases:
  - In Range
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Range Checking]]"
concepts:
  - "[[Function Arguments]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Check if a number `value` is `>= start` and `< end`, with implicit `start=0` for two-argument calls and reversed-range swap support.

## Problem

Implement a function `inRange(value, [start=0], end)` to check if a number `value` is greater than or equal to `start` and less than `end`. If only two arguments are specified, the second argument becomes `end` and `start` is set to `0`. If `start` is greater than `end`, the parameters are swapped to support reversed ranges.

```js
inRange(3, 2, 4);    // => true
inRange(4, 8);       // => true
inRange(4, 2);       // => false
inRange(2, 2);       // => false
inRange(1.2, 2);     // => true
inRange(5.2, 4);     // => false
inRange(-3, -2, -6); // => true
```

## Companies

- None

## Pattern

- [[Range Checking]]

## 🤔 Thought Process

Function supports two calling conventions: `inRange(value, start, end)` and `inRange(value, end)` (implicit `start = 0`). Use `arguments.length === 2` to detect two-argument case. If `start > end`, swap the two via destructuring. Range is inclusive on `start`, exclusive on `end` (`value >= start && value < end`).

## 💻 Final Solution

```js
export default function inRange(value, start, end) {
  if (arguments.length === 2) {
    end = start;
    start = 0;
  }

  if (start > end) {
    [start, end] = [end, start];
  }

  return value >= start && value < end;
}
```

## 🤔 Why This Works

JavaScript assigns arguments positionally — `inRange(4, 8)` gives `value=4, start=8, end=undefined`. The `arguments.length === 2` check detects this and shifts `start` into `end` with `start` defaulted to `0`. The `start > end` swap handles reversed ranges like `inRange(-3, -2, -6) → start=-6, end=-2`. The final comparison `value >= start && value < end` is inclusive on the lower bound and exclusive on the upper bound.

## 🐞 Bugs I Made

None.

## Production Considerations

Native alternative: use comparison operators directly. No built-in JS range function exists; lodash `_.inRange` provides the same semantics. O(1) time, O(1) space.

## ⭐ Revision Notes

### Key Facts

- `arguments.length` distinguishes two-arg vs three-arg calls — `arguments` is not array but array-like with `.length`.
- Destructuring swap `[a, b] = [b, a]` works on declared variables in modern JS.
- Range bounds: `start` inclusive, `end` exclusive.

### Common Interview Questions

- Why `arguments` instead of default parameter `[start=0]`? The default param can't detect two-arg case — `inRange(4, 8)` with `start=0` default would lose the `8`. `arguments.length` is the reliable check.
- Can you use rest parameters? `function inRange(value, ...args)` then `if (args.length === 1)` — works but `arguments` is more idiomatic here.

### Interview Takeaways

- Normalize variable-arity arguments first, then apply domain logic uniformly.
- Reversed-range swap must happen **after** arity normalization.

### Related

- [[Range Checking]]
- [[Function Arguments]]
