---
title: Given a predicate function and an array, implement a function dropRightWhile(array, predicate). This function should create a slice of array excluding elements dropped from the end. Elements are dropped until predicate returns falsy. Your function should not modify the original array. The array may or may not be in sorted order.
aliases:
  - Drop Right While
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
> Drop elements from end while predicate true, return remaining slice

## Problem

Return slice of `array` excluding elements dropped from the end. Drop continues while `predicate(value, index, array)` returns truthy. Stop at first falsy. No mutation.

```javascript
dropRightWhile([1, 2, 3, 4, 5], (value) => value > 3);            // => [1, 2, 3]
dropRightWhile([1, 2, 3], (value) => value < 6);                  // => []
dropRightWhile([1, 2, 3, 4, 5], (value) => value > 6);            // => [1, 2, 3, 4, 5]
dropRightWhile([1, 2, 3, 4, 5], (_value, index) => index > 2);    // => [1, 2, 3]
dropRightWhile([10, 11, 12, 4, 5], (value, _i, arr) => value < arr[1]); // => [10, 11, 12]
```

## Companies

- [[Lodash]]

## Pattern

- [[Drop While]]

## 🤔 Thought Process

Aim to find the index where predicate failed. Traverse from end with `while`, check index valid and predicate true, then decrement. Return all elements up to that index via `slice`.

## 💻 Final Solution

```javascript
export default function dropRightWhile(array, predicate) {
  let index = array.length - 1;

  while (index >= 0 && predicate(array[index], index, array)) {
    index--;
  }
  return array.slice(0, index + 1);
}
```

## 🤔 Why This Works

- Traverses right-to-left, stops at first falsy — that's the boundary index
- `slice(0, index + 1)` returns everything up to boundary, inclusive of the stopping element
- `index >= 0` guard handles predicate-always-true case → `slice(0, 0)` = `[]`
- Predicate called with all 3 args: `(value, index, array)` per spec
- `slice` returns new array, original untouched

## 🐞 Bugs I Made

None.

## Production Considerations

Lodash `_.dropRightWhile` identical behavior. `slice` is O(n) but copies are inherent to "new array" requirement. Alternative: `findIndex` reversed, but manual `while` is clearer and single-pass.

## ⭐ Revision Notes

### Key Facts

- Drop continues only while predicate truthy — stops at first falsy
- Right-to-left traversal via `index--`
- Boundary index is where predicate first fails

### Common Interview Questions

- **What if predicate always true?** `index` reaches `-1`, `slice(0, 0)` → `[]`
- **What if always false?** `index` stays at `length - 1`, returns whole array
- **Why pass all 3 args?** Spec requires `(value, index, array)` — some predicates use `index` or `array` (Ex 4, 5)

### Interview Takeaways

- `slice(0, index + 1)` boundary math — off-by-one trap
- Think "find boundary index" not "filter elements"
- Always pass full predicate signature

### Related

- [[Drop While]]
- [[Array Traversal]]
- [[Array.prototype.slice]]
