---
title: Implement intersection(...arrays) to return the unique values present in every provided array
aliases:
  - Intersection
difficulty: Easy
time: 10 min
languages:
  - JavaScript
pattern:
  - "[[Set Lookup]]"
concepts:
  - "[[Set]]"
  - "[[Rest Parameters]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 10 min
> Return unique values appearing in every provided array, preserving first-array order, without mutating inputs.

## Problem

Implement `intersection(...arrays)` so it returns a new array containing the unique values that appear in every provided array. Preserve the order from the first array and do not modify the original arrays. If no arrays are provided, return an empty array.

```js
const arr1 = [1, 2, 3];
const arr2 = [2, 3, 4];
const arr3 = [3, 4, 5];

intersection(arr1, arr2, arr3); // => [3]

const arr4 = [1, 5, 7, 9, 7];
intersection(arr4); // => [1, 5, 7, 9]

intersection(); // => []
```

**Constraints:** values may be any type, arrays may vary in length or be empty, originals must not be mutated, `0 <= number of arrays <= 20`.

## Companies

- None

## Pattern

- [[Set Lookup]]

## 🤔 Thought Process

1. Receive any number of arrays using rest parameters: `function intersection(...arrays)`
2. If no arrays are provided, return `[]`.
3. Convert each array into a `Set` for fast membership checks and automatic deduplication.
4. Use the **first array as the reference order**.
5. Keep a value only if it exists in **every Set**.
6. Return a new array.

## 💻 Final Solution

```js
export default function intersection(...arrays) {
  if (arrays.length === 0) return [];

  const sets = arrays.map(array => new Set(array));

  return [...sets[0]].filter(value =>
    sets.every(set => set.has(value))
  );
}
```

## 🤔 Why This Works

**Rest parameters.** `intersection([1,2], [2,3], [2,4])` collects into `arrays = [[1,2], [2,3], [2,4]]`.

**Set.** `arrays.map(array => new Set(array))` gives two properties at once — duplicates removed (`new Set([1,2,2,3])` is `Set {1,2,3}`) and O(1) membership via `set.has(value)`.

**Order preservation.** The problem requires first-array order, so iteration starts at `[...sets[0]]`. Spreading a Set yields values in insertion order, so `intersection([3,1,2], [1,2,3])` returns `[3,1,2]`, not a sorted result.

**`every()`.** `sets.every(set => set.has(value))` reads as "does this value exist in every array?" Only when all Sets contain the value does `filter()` keep it — that is exactly the definition of intersection.

**No mutation.** `map`, `Set`, spread, and `filter` all produce new values; inputs stay untouched.

## 🐞 Bugs I Made

- Used `arrays.length == 0` instead of `_===`.
- Declared `sets` with `let` when it is never reassigned — should be `const`.

## Production Considerations

- Lodash ships `_.intersection(...arrays)` with identical semantics.
- Complexity: building Sets is O(total input elements); checking is O(n × k) where n = unique values in the first array and k = number of arrays. Space O(total input elements).
- Set uses SameValueZero, so `NaN` matches `NaN` but objects compare by reference — `[{a:1}]` and `[{a:1}]` do not intersect.
- Pick the **smallest** input array as the driver instead of the first when order is not required; fewer candidates means fewer `every()` passes.

## ⭐ Revision Notes

### Key Facts

- `[...new Set(arr)]` is the one-liner for unique values, in first-seen order.
- `filter` + `every` composes into "keep x if it satisfies all of these predicates".
- Rest parameters always produce an array, so `arrays.length === 0` is the no-input guard.
- `sets.every(...)` on an empty `sets` array returns `true` — unreachable here because of the early return.

### Common Interview Questions

- Why Sets instead of `includes()`? `includes()` inside a filter is O(n·m); `Set.has()` is O(1), so the whole pass is O(n + m).
- What if the first array is huge and the last is tiny? Drive iteration from the smallest Set when order does not matter — fewer candidates to test.
- Does this handle `NaN`? Yes. Set uses SameValueZero, so `has(NaN)` is `true`.

### Interview Takeaways

- For array intersection: Set for membership, first array for order, `every()` for the all-arrays check.
- Normalize inputs (arrays to Sets) before applying logic, same move as [[In Range]].

### Related

- [[Set Lookup]]
- [[Difference]]
- [[Unique Array]]
- [[From Pairs]]
