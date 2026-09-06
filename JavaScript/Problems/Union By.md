---
title: Implement unionBy(iteratee, ...arrays)
aliases:
  - Union By
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
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Combine multiple arrays into one unique-value array, using an iteratee to compute the uniqueness key.

## Problem

Implement a function `unionBy(iteratee, ...arrays)` that creates an array of unique values, in order, from all given arrays and accepts an iteratee, which is invoked for each element of each array to generate the criterion by which uniqueness is computed.

```js
unionBy(iteratee, ...arrays);
```

**Arguments:**
- `iteratee (Function)`: The iteratee invoked per element. The function is invoked with one argument: `value`.
- `[arrays] (...Array)`: The arrays to inspect.

**Returns:** `(Array)` Returns the new array of combined values.

**Examples**

```js
unionBy((value) => value, [2], [1, 2]); // => [2, 1]

unionBy(Math.floor, [2.1], [1.2, 2.3]); // => [2.1, 1.2]

unionBy((o) => o.x, [{ x: 1 }], [{ x: 2 }, { x: 1 }]);
// => [{ 'x': 1 }, { 'x': 2 }]

unionBy((o) => o.m, []); // => []

unionBy((o) => o.m, [{ n: 1 }], [{ m: 2 }]);
// => [{ n: 1 }, { m: 2 }]
```

The function should return an empty array if the provided arrays are empty and treat falsy values as-is.

## Pattern

- [[Set Lookup]]

## 🤔 Thought Process

* Process arrays left to right.
* For each value, calculate its uniqueness key using `iteratee`.
* `seen` stores keys that have already appeared.
* If the key is already seen → skip it.
* Otherwise:
  * Add the original value to `result`.
  * Add its transformed key to `seen`.

## 💻 Final Solution

```js
export default function unionBy(iteratee, ...arrays) {
  if (arrays.length === 0) return [];

  const seen = new Set();
  const result = [];

  for (const array of arrays) {
    for (const value of array) {
      const key = iteratee(value);

      if (seen.has(key)) {
        continue;
      }

      result.push(value);
      seen.add(key);
    }
  }

  return result;
}
```

## 🤔 Why This Works

Example:

```js
unionBy(Math.floor, [2.1], [1.2, 2.3])
```

Processing:

```text
2.1 → key 2 → not seen → result [2.1], seen {2}

1.2 → key 1 → not seen → result [2.1, 1.2], seen {2, 1}

2.3 → key 2 → already seen → skip
```

Final:

```js
[2.1, 1.2]
```

Notice the important distinction:

```text
key  → used for uniqueness
value → stored in result
```

## 🐞 Bugs I Made

The current solution has no functional bugs.

A previous attempt used:

```js
resultSet.every(set => !set.has(key))
```

That asks whether the key exists in **none** of the arrays, which isn't what `unionBy` needs.

`unionBy` doesn't care whether a key exists in other arrays beforehand. It simply processes values sequentially and remembers what it has already encountered.

## Production Considerations

- In Lodash, `iteratee` is the **last parameter** (optional), but in this question it is a required parameter and the **first parameter** for simplicity.
- Falsy keys (`0`, `false`, `null`, `undefined`) work correctly because `Set` handles them normally.
- Time: O(total number of elements). Space: O(number of unique transformed keys).

## ⭐ Revision Notes

### 🔑 Key Facts

* `unionBy` = combine all arrays + remove duplicates based on iteratee.
* First occurrence wins.
* Preserve the original value in the result.
* `Set` stores the iteratee result, not the original value.
* Process order matters: first array → second array → third array...
* Falsy keys such as `0`, `false`, `null`, and `undefined` work correctly.
* Time: O(total number of elements).
* Space: O(number of unique transformed keys).

### 🧠 Mental Model

```text
value
  ↓
iteratee(value)
  ↓
  key
  ↓
seen?
 ┌───────┴───────┐
 YES             NO
 ↓                ↓
skip          result.push(value)
                  ↓
             seen.add(key)
```

The key phrase:

> **Compare the key, return the original value.**

### Common Interview Questions

- How is `unionBy` different from `Intersection By`? → `unionBy` keeps every value whose key is new; `Intersection By` keeps only values whose key exists in **every** array.
- How is it different from `union`? → `union` compares values directly; `unionBy` compares transformed keys.
- What if the iteratee returns `undefined`? → All values that produce `undefined` are deduplicated together (only the first is kept).

### Interview Takeaways

* Process inputs sequentially, track seen keys with `Set`.
* Store keys, return values.
* `unionBy` is the "merge with dedup by key" pattern.

### Related

- [[Set Lookup]]
- [[Intersection By]]
- [[Group By]]
- [[Count By]]
- [[Unique Array]]