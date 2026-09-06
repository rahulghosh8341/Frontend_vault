---
title: Implement intersectionWith(comparator, ...arrays)
aliases:
  - Intersection With
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Comparator Functions]]"
  - "[[Rest Parameters]]"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Find elements from the first array that have a comparator-matching element in every other array, preserving first-array order.

## Problem

The `intersectionWith` function takes a custom comparator function and multiple arrays as arguments. It compares the elements of the arrays using the comparator function to determine equality. The function returns a new array containing the elements that are present in all given arrays.

Implement `intersectionWith(comparator, ...arrays)` so it returns the matching values from the first array in the same order they appear there. If no arrays are provided, or any array is empty, return `[]`.

```js
intersectionWith(comparator, ...arrays);
```

**Arguments:**
- `comparator (Function)`: The function used to compare two elements. It is invoked with `(arrVal, othVal)` and should return `true` when the values should be treated as equal.
- `arrays (...Array)`: The arrays to inspect.

**Returns:** `(Array)` Returns the intersecting values from the first array.

**Examples**

```js
const arr1 = [
  { x: 1, y: 2 },
  { x: 2, y: 3 },
];
const arr2 = [
  { y: 2, x: 1 },
  { x: 3, y: 4 },
];

const result = intersectionWith(
  (a, b) => a.x === b.x && a.y === b.y,
  arr1,
  arr2,
); // => [{ x: 1, y: 2 }]

intersectionWith((a, b) => a === b, [1, 2, 3], [2, 3, 4], [3, 4, 5]);
// => [3]
```

**Notes:**
- In Lodash, `comparator` is optional and is the last parameter, but in this question it is a required parameter for simplicity.
- The order of elements in the resulting array is determined by the order in which they appear in the first array.
- If no arrays are provided, the function will return an empty array.
- If any of the arrays are empty, the function will return an empty array.

## Companies

- None

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

* `...arrays` collects all input arrays.
* If there are no arrays or any array is empty → `[]`.
* Take each value from the **first array**.
* For that value, check **every other array**.
* In each other array, use `some()` to find **one matching value** using the comparator.
* If every array has a match, keep the original value from the first array.

Core structure:

```js
arrays[0].filter(value =>
  arrays.slice(1).every(array =>
    array.some(otherValue =>
      comparator(value, otherValue)
    )
  )
)
```

## 💻 Final Solution

```js
export default function intersectionWith(comparator, ...arrays) {
  if (arrays.length === 0)
    return [];

  if (arrays.some(array => array.length === 0)) {
    return [];
  }

  return arrays[0].filter(value => {
    return arrays.slice(1).every(array => {
      return array.some(otherValue => {
        return comparator(value, otherValue)
      });
    });
  });
}
```

## 🤔 Why This Works

Three array methods have three different jobs:

```text
filter → which values from the first array should I return?
every  → does this value exist in ALL other arrays?
some   → does this value match ANY element in this particular array?
```

For:

```text
arr1 = [A, B]
arr2 = [C, B]
```

For `B`:

```text
filter → consider B
every  → check arr2
some   → compare B against C, then B
comparator(B, B) → true
```

Therefore `B` is kept.

## 🐞 Bugs I Made

The key confusion was thinking:

```js
arrays[0]
```

is passed as `value`.

It isn't.

```js
arrays[0]
```

is the **whole first array**, but:

```js
arrays[0].filter(value => ...)
```

makes `value` one element from that array.

Similarly:

```js
array.some(otherValue => ...)
```

makes `otherValue` one element from the other array.

So the comparator receives:

```js
comparator(
  oneValueFromFirstArray,
  oneValueFromOtherArray
)
```

not two arrays.

## Production Considerations

- In Lodash, `comparator` is optional and is the **last parameter**, but in this question it is a required parameter and the **first parameter** for simplicity.
- Complexity: **O(n₁ × n₂ × ...)** in the worst case for multiple arrays; for two arrays, **O(n × m)**.
- For deep-equality on plain objects, an alternative is JSON.stringify-based compare or a deep-equal helper, but a custom comparator is the explicit, fast path.

## ⭐ Revision Notes

### 🔑 Key Facts

* `intersectionWith` uses a **custom comparator**, so `Set` cannot directly solve the comparison.
* `filter + every + some` is the natural approach.
* `filter()` preserves the **original values and order from the first array**.
* `some()` stops as soon as it finds a match.
* `every()` stops as soon as one array has no match.
* Complexity: **O(n₁ × n₂ × ...)** in the worst case for multiple arrays; for two arrays, **O(n × m)**.
* No mutation of input arrays.

### 🧠 Mental Model

Think:

```text
FIRST ARRAY
    ↓
take ONE value
    ↓
check EVERY other array
    ↓
in each array, find SOME matching value
    ↓
comparator(a, b)
    ↓
all arrays matched?
    ↓
YES → keep original value
NO  → discard
```

### Common Interview Questions

- How does `intersectionWith` differ from `intersectionBy`? → `intersectionBy` derives a single comparison key per value via iteratee and uses `Set.has()`; `intersectionWith` delegates the comparison entirely to a user-supplied comparator.
- Why `Set` does not work directly here? → `Set` uses SameValueZero on references/values; a comparator may define equality differently (e.g., comparing object shapes).
- What does `arrays.slice(1)` do? → It excludes the first array (already the source of values being filtered) so `every` only iterates over the other arrays.

### Interview Takeaways

The pattern to remember is:

```js
filter → every → some → comparator
```

Read it as:

> **Filter the first array by keeping values for which every other array has some value that matches according to the comparator.**

### Related

- [[Array Traversal]]
- [[Intersection]]
- [[Intersection By]]
- [[Difference]]