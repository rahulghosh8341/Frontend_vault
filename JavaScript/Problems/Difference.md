---
title: Implement difference(array, values) so it returns a new array containing the values from array that do not appear in values. Preserve the order from array and do not modify the original arrays. For this question, treat NaN values as equal and skip empty slots in sparse arrays.
aliases:
  - Difference
difficulty: Easy
time: 10 min
languages:
  - JavaScript
companies:
  - "[[Lodash]]"
pattern:
  - "[[Set Lookup]]"
concepts:
  - "[[Set]]"
  - "[[NaN Equality]]"
  - "[[Sparse Arrays]]"
solved: true
solvedDate: 2026-08-30
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Return array elements not present in second array, preserve order

## Problem

Return new array containing values from `array` that do NOT appear in `values`. Preserve order, no mutation. Treat `NaN` as equal. Skip empty slots.

```javascript
difference([1, 2, 3], [2, 3]);         // => [1]
difference([1, 2, 3, 4], [2, 3, 1]);   // => [4]
difference([1, NaN, 2], [NaN]);        // => [1, 2]
difference([1, 2, 3], [2, 3, 1, 4]);   // => []
difference([1, , 3], [1]);             // => [3]
difference([1, 2, 3], []);             // => [1, 2, 3]
```

## Companies

None

## Pattern

- [[Set Lookup]]

## 🤔 Thought Process

Use `Set` for O(1) membership checks. `Set` handles `NaN` equality correctly by default (`NaN === NaN` is false in JS, but `Set.has(NaN)` finds `NaN`). For sparse arrays, `i in array` returns `false` for empty slots—use this to skip them.

## 💻 Final Solution

```javascript
export default function difference(array, values) {
  const result = [];
  const valuesSet = new Set(values);

  for (let i = 0; i < array.length; i++) {
    const value = array[i];
    if (!valuesSet.has(value) && !(value === undefined && !(i in array))) {
      result.push(value);
    }
  }
  return result;
}
```

## 🤔 Why This Works

- `Set.has()` gives O(1) lookup vs O(n) `includes()`
- `Set` treats `NaN` as equal to itself (ES6 spec), satisfying requirement
- `i in array` returns `false` for empty slots in sparse arrays
- Combined check: skip if value in `valuesSet` OR if it's an empty slot (`undefined` at missing index)

## 🐞 Bugs I Made

None.

## Production Considerations

Lodash `_.difference` uses same Set approach. Could also use `array.filter()` with `Set.has()`, but manual loop avoids extra callback overhead for large arrays. Alternative: `array.filter(v => !valuesSet.has(v))` but needs additional sparse array handling.

## ⭐ Revision Notes

### Key Facts

- `Set.has(NaN)` returns `true` if Set contains `NaN`
- `i in array` checks index existence (sparse array detection)
- `undefined` can be valid value OR empty slot marker—distinguish with index check

### Common Interview Questions

- **Why not `array.filter(item => !values.includes(item))`?** O(n²) vs O(n) with Set
- **How does `Set` handle `NaN`?** Treats `NaN` equal to itself, unlike `_===`
- **Why check `i in array`?** Detect empty slots in sparse arrays

### Interview Takeaways

- Use Set for membership testing when comparing arrays
- Sparse arrays need special handling—`undefined` ≠ empty slot
- Algorithmic complexity: nested loops O(n²), Set O(n)

### Related

- [[Set Lookup]]
- [[NaN Equality]]
- [[Sparse Arrays]]
- [[Set]]
