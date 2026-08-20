---
title: Implement a function `flatten` that recursively flattens a nested array
aliases:
  - Flatten
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
  - "[[Apple]]"
  - "[[Lyft]]"
  - "[[Meta]]"
  - "[[Rippling]]"
  - "[[Salesforce]]"
  - "[[Google]]"
  - "[[Roblox]]"
  - "[[ByteDance]]"
  - "[[TikTok]]"
  - "[[Airbnb]]"
  - "[[Palantir]]"
  - "[[PayPal]]"
pattern:
  - "[[DFS Recursion]]"
concepts:
  - "[[Recursion]]"
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Recursively flattens nested arrays into a single-level array.

## Problem

Implement a function `flatten(array)` that returns a newly created array with all subarray elements concatenated recursively into a single level.

```js
flatten([1, 2, 3]);               // [1, 2, 3]
flatten([1, [2, 3]]);            // [1, 2, 3]
flatten([[1, 2], [3, 4]]);        // [1, 2, 3, 4]
flatten([1, [2, [3, [4, [5]]]]]); // [1, 2, 3, 4, 5]
```

---

## Companies

- [[Amazon]]
- [[Apple]]
- [[Lyft]]
- [[Meta]]
- [[Rippling]]
- [[Salesforce]]
- [[Google]]
- [[Roblox]]
- [[ByteDance]]
- [[TikTok]]
- [[Airbnb]]
- [[Palantir]]
- [[PayPal]]

---

## Pattern

- [[DFS Recursion]]

---

## 🤔 Thought Process

* Traverse the array one item at a time.
* Classify each item:
  * **Non-array** → it's a leaf value, so add it directly to the result.
  * **Array** → it may contain more nested arrays, so recursively call `flatten()`.
* The recursive call returns a flattened array.
* Spread that returned array into the current result using `result.push(...flatten(item))`.
* Continue until every nested array has been processed.
* Return a new flattened array without modifying the input.

---

## 💻 Final Solution

```js
export default function flatten(array) {
  const result = [];

  for (const item of array) {
    if (Array.isArray(item)) {
      result.push(...flatten(item));
    } else {
      result.push(item);
    }
  }

  return result;
}
```

---

## 🤔 Why This Works

Each item has only two possibilities:

```text
item
├── value → push directly
└── array → recursively flatten → push flattened values
```

Recursion continues until every item is a leaf value. Using `push(...flatten(item))` preserves the original left-to-right order while combining nested results.

---

## 🐞 Bugs I Made

* Initially tried:

  ```js
  result = flatten(item);
  ```

  This incorrectly **replaced** the current result instead of adding the recursively flattened elements.
* `result` was declared with `const`, so reassignment would also cause an error.
* Corrected it to:

  ```js
  result.push(...flatten(item));
  ```

---

## Production Considerations

- In production code (ES2019+), use built-in `Array.prototype.flat(Infinity)` to flatten nested arrays without writing custom code.
- Write custom `flatten` when skipping/keeping specific nested structures (e.g. TypedArrays), flattening object trees, avoiding stack overflow on extremely deep nesting with iterative queues/stacks, or lazily generating values via generator functions (`function*`).

---

## ⭐ Revision Notes

### Key Facts

* Classify each item → **leaf or nested structure**.
* Nested array → recursively process it.
* Leaf → add to result.
* `result.push(...flatten(item))` merges the recursive result.
* Creates a **new array**; does not mutate the input.
* Time: **O(n)** for `n` total elements processed.
* Space: **O(d)** recursion stack + **O(n)** output, where `d` is maximum nesting depth.

### Common Interview Questions

* How do you flatten an array in modern JS? → `Array.prototype.flat(Infinity)`
* What happens if array nesting is extremely deep? → Risk of maximum call stack size exceeded (`StackOverflow`). Solved using an iterative stack/queue.
* Does `flatten` mutate the original array? → No, returns a newly created array.

### Interview Takeaways

* Know `Array.prototype.flat(Infinity)` for production, but recursion is the important interview pattern.
* `reduce()`, `flatMap()`, and iterative approaches are alternative implementations—not necessary to memorize for this problem.

### Related

- [[DFS Recursion]]
- [[Recursion]]
