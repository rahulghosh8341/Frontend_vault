---
title: Given a predicate function and an array, implement a function dropWhile(array, predicate). This function should create a slice of array excluding elements dropped from the start. Elements are dropped until predicate returns falsy. Your function should not modify the original array. The array may or may not be in sorted order.
aliases:
  - Drop While
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Drop While]]"
concepts:
  - "[[Array.prototype.slice]]"
  - "[[Predicate]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Drop elements from start while predicate true, return remaining slice

## Problem

Return slice of `array` excluding elements dropped from the start. Drop continues while `predicate(value, index, array)` returns truthy. Stop at first falsy. No mutation.

```javascript
dropWhile([1, 2, 3, 4, 5], (value) => value < 3);            // => [3, 4, 5]
dropWhile([1, 2, 3], (value) => value < 6);                  // => []
dropWhile([1, 2, 3, 4, 5], (value) => value > 6);            // => [1, 2, 3, 4, 5]
dropWhile([1, 2, 3, 4, 5], (_value, index) => index < 3);    // => [4, 5]
dropWhile([4, 5, 12, 10, 11], (value, _i, arr) => value < arr[2]); // => [12, 10, 11]
```

## Companies

- [[Lodash]]

## Pattern

- [[Drop While]]

## 🤔 Thought Process

Find boundary index where predicate first returns falsy. Traverse from start with `for` loop, stop on falsy. Return `slice(i)` — everything from boundary onward.

## 💻 Final Solution

```javascript
export default function dropWhile(array, predicate) {
  let i;
  for (i = 0; i < array.length && predicate(array[i], i, array); i++) {
    // empty — just advancing i until predicate fails
  }
  return array.slice(i);
}
```

## 🤔 Why This Works

- `for` loop runs while predicate truthy; at first falsy, loop body skipped, `i` stays at boundary index
- `slice(i)` returns all elements from boundary to end
- `i` reaches `array.length` if predicate always true → `slice(array.length)` = `[]`
- `i` stays `0` if predicate immediately false → `slice(0)` = whole array
- Predicate receives all 3 args per spec: `(value, index, array)`

## 🐞 Bugs I Made

Left unused `let result = []` in skeleton — dead code, harmless.

## Production Considerations

Lodash `_.dropWhile` identical. `slice` is O(n). For huge arrays, find index with `findIndex` then slice: `array.slice(array.findIndex((v, i, a) => !predicate(v, i, a)))` — but `for` loop is clearer and single-pass.

## ⭐ Revision Notes

### Key Facts

- Drop-while is contiguous prefix removal, not filter
- Boundary index = first element where predicate falsy
- `slice(i)` with `i` at `array.length` returns `[]`

### Common Interview Questions

- **What if predicate always true?** `i = array.length`, `slice(array.length)` → `[]`
- **What if always false?** `i = 0`, `slice(0)` → full array
- **Why empty loop body?** Side-effect driven — loop advances `i` only, no work per iteration

### Interview Takeaways

- Mirror of `dropRightWhile` — reverse direction, `slice` from start instead of end
- Empty `for` body is intentional and idiomatic here
- Always pass full predicate signature

### Related

- [[Drop While]]
- [[Drop Right While]]
- [[Array.prototype.slice]]