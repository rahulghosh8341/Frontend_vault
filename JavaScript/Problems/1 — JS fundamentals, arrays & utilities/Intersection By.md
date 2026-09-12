---
title: Implement intersectionBy(iteratee, ...arrays)
aliases:
  - Intersection By
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Set Lookup]]"
concepts:
  - "[[Set]]"
  - "[[Iteratee Functions]]"
  - "[[Rest Parameters]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Find intersecting elements across multiple arrays after applying an iteratee, returning original values from the first array.

## Problem

The `intersectionBy` function takes an iteratee function and multiple arrays as arguments. It first applies the iteratee function to transform the values in all arrays. Then, it identifies the set of transformed values that are common across all arrays. Finally, it returns an array containing the original values from the first array that correspond to these common transformed values.

Note: The comparison to find common elements uses the values after the iteratee is applied, but the final returned array contains the original values from the first array (before the iteratee was applied).

```js
intersectionBy(iteratee, ...arrays);
```

The iteratee function is invoked with one argument: `value`, where `value` is the current value being iterated.

**Examples**

```js
// Get the intersection based on the floor value of each number
const result = intersectionBy(Math.floor, [1.2, 2.4], [2.5, 3.6]);
// Compares floored values ([1, 2] vs [2, 3]). The common floor value is 2.
// Original value from the first array that floors to 2 is 2.4.
// => [2.4]

// Get the intersection based on the lowercase value of each string
const result2 = intersectionBy(
  (str) => str.toLowerCase(),
  ['apple', 'banana', 'ORANGE', 'orange'],
  ['Apple', 'Banana', 'Orange'],
);
// Common lowercase results are 'apple', 'banana', 'orange'.
// Returns the corresponding first-occurrence originals from the first array: 'apple', 'banana', 'ORANGE'.
// => ['apple', 'banana', 'ORANGE']

// Single array case
intersectionBy(Math.floor, [1, 2.5, 3]); // => [1, 2.5, 3]
```

**Constraints:**
- The input arrays may contain any type of values.
- The input arrays may have varying lengths.
- The input arrays may be empty.
- The function should not modify the original arrays.
- `0 <= number of arrays <= 20`

## Companies

- None

## Pattern

- [[Set Lookup]]

## 🤔 Thought Process

* If there are no arrays, return `[]`.
* For each array, transform every value using `iteratee()` and store the transformed values in a `Set`.
* Iterate over the **original first array** because the final result must contain original values.
* For each value:

  1. Calculate its transformed key.
  2. Skip it if that key was already returned (`seen`).
  3. Check whether the key exists in **every** transformed Set.
  4. If yes, keep the original value and mark the key as seen.

## 💻 Final Solution

```js
export default function intersectionBy(iteratee, ...arrays) {
  // empty arrays
  if (arrays.length === 0)
    return [];

  // take each array -> create new set of array with each item result of iteratee()
  let resultSet = arrays.map(array => new Set(array.map(value => iteratee(value))));

  // take the first array => compare iteratee(each item) with each item of sets discard if already seen
  let seen = new Set();
  return arrays[0].filter(value => {
    let key = iteratee(value);

    if (seen.has(key))
      return false;

    if (resultSet.every(set => set.has(key))) {
      seen.add(key);
      return true;
    }
    return false;
  });
}
```

## 🤔 Why This Works

There are two separate concepts:

```text
Original value       Transformed key
    2.4       →          2
    ORANGE    →       "orange"
```

* `resultSet` answers: **"Does this transformed value exist in every array?"**
* `seen` answers: **"Have I already returned an original value for this transformed value?"**

This gives the required behavior:

```js
intersectionBy(
  Math.floor,
  [1.2, 2.4],
  [2.5, 3.6]
)
```

```text
First array:  [1.2, 2.4] → [1, 2]
Second:       [2.5, 3.6] → [2, 3]

Common key = 2
Original value = 2.4

Result → [2.4]
```

## 🐞 Bugs I Made

Your current solution is **correct**.

One important thing you got right is adding `seen`:

```js
if (seen.has(key))
  return false;
```

Without it:

```js
['ORANGE', 'orange']
```

would return both values because both transform to `"orange"`.

Your ordering is also correct:

```js
if (seen.has(key)) return false;

if (resultSet.every(set => set.has(key))) {
  seen.add(key);
  return true;
}
```

You only add to `seen` **after confirming the key actually exists in every array**.

## Production Considerations

- In Lodash, `iteratee` is optional and is the **last parameter**, but in this question it is a required parameter and the **first parameter** for simplicity.
- The native ES2024 `Object.groupBy` / `Map.groupBy` don't directly solve intersection, but the Set-based approach here is standard.
- Time complexity: approximately **O(total number of elements)**.
- Space complexity: **O(total number of transformed elements)** for the Sets.

## ⭐ Revision Notes

### 🔑 Key Facts

* `...arrays` collects all arrays after `iteratee`.
* `map()` transforms each array.
* `Set` gives efficient membership checking with `.has()`.
* `every()` means **the transformed key must exist in all arrays**.
* `filter()` operates on the **original first array**, preserving original values.
* `seen` ensures **one result per transformed key**.
* Original arrays are not modified.
* Time complexity: approximately **O(total number of elements)**.
* Space complexity: **O(total number of transformed elements)**.

### 🧠 Mental Model

Remember this pipeline:

```text
             iteratee
Array ─────────────────→ transformed values
                              ↓
                             Set
                              ↓
                         fast lookup
                              ↓
Original first array ──→ filter
                              ↓
                    every Set has key?
                              ↓
                         seen before?
                              ↓
                    return original value
```

The most important distinction:

> **Compare keys, return values.**

### Common Interview Questions

- How does `intersectionBy` differ from `intersection`? → `intersection` compares values directly; `intersectionBy` transforms values via iteratee before comparing, but returns original values.
- Why is the `seen` Set necessary? → To deduplicate by transformed key when the first array contains multiple elements that map to the same transformed value.
- What happens with `NaN`? → `Set` uses SameValueZero, so `NaN` works correctly.

### Interview Takeaways

`intersectionBy` is essentially:

```text
intersection()
     +
iteratee transformation
     +
preserve original first-array values
     +
deduplicate by transformed key
```

Your implementation is a **good, clean solution** for this problem.

### Related

- [[Set Lookup]]
- [[Intersection]]
- [[Intersection With]]
- [[Difference]]
- [[Unique Array]]