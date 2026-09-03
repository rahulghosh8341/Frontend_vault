---
title: Implement Array.prototype.concat
aliases:
  - Array.prototype.concat
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Apple]]"
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Sparse Arrays]]"
  - "[[Prototype Method Polyfill]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Merges two or more arrays/values into a new array, preserving sparse array holes and flattening arguments one level deep.

## Problem

Implement `Array.prototype.concat`. To avoid overwriting the actual `Array.prototype.concat`, implement it as `Array.prototype.myConcat`.

The `myConcat` method merges two or more arrays or values into a newly returned array without mutating the original arrays.

```js
[1, 2, 3].myConcat([4, 5, 6]);   // [1, 2, 3, 4, 5, 6]
[1, 2, 3].myConcat(4, 5, 6);      // [1, 2, 3, 4, 5, 6]
[1, 2, 3].myConcat(4, [5, 6]);    // [1, 2, 3, 4, 5, 6]
```

---

## Companies

- [[Apple]]

---

## Pattern

- [[Array Traversal]]

---

## 🤔 Thought Process

1. Create a **new result array** so original arrays aren't modified.
2. Maintain a separate `resultIndex` for the output position.
3. Process `this` first.
4. For each argument:

   * Array → copy its elements **one level deep**.
   * Non-array → add it as a single element.
5. For sparse arrays, check `i in item` before assigning.
6. Increment `resultIndex` even when the source index is a hole.
7. Set `result.length = resultIndex` at the end so trailing holes are preserved.

---

## 💻 Final Solution

```js
/**
 * @template T
 * @param {...(T | Array<T>)} items
 * @return {Array<T>}
 */
Array.prototype.myConcat = function (...items) {
  const result = [];
  let resultIndex = 0;

  for (let i = 0; i < this.length; i++) {
    if (i in this) {
      result[resultIndex] = this[i];
    }
    resultIndex++;
  }

  // process each item
  for (const item of items) {
    if (Array.isArray(item)) {
      for (let i = 0; i < item.length; i++) {
        if (i in item) {
          result[resultIndex] = item[i];
        }
        resultIndex++;
      }
    } else {
      result[resultIndex] = item;
      resultIndex++;
    }
  }
  result.length = resultIndex;
  
  return result;
};
```

---

## 🤔 Why This Works

### One-level flattening

```js
[1, 2].myConcat(3, [4, 5])
```

becomes:

```text
[1, 2] + 3 + [4, 5]
       ↓
[1, 2, 3, 4, 5]
```

Only arrays directly passed to `concat` are spread. Nested arrays aren't recursively flattened.

### Sparse Arrays

```js
[2, , 3]
```

has:

```text
index 0 → 2
index 1 → hole
index 2 → 3
```

Use:

```js
if (i in item) {
  result[resultIndex] = item[i];
}
resultIndex++;
```

The index advances even when there is a hole.

### `result.length = resultIndex`

Incrementing `resultIndex` doesn't change the actual array:

```js
resultIndex++;
```

So trailing holes such as:

```js
new Array(2)
```

would otherwise disappear.

Setting:

```js
result.length = resultIndex;
```

makes the result have the correct length while keeping those positions as holes.

---

## 🐞 Bugs I Made

### 1. Didn't preserve holes

Initially:

```js
result[resultIndex++] = item[i];
```

This converts a hole into an actual `undefined` value.

Correct:

```js
if (i in item) {
  result[resultIndex] = item[i];
}
resultIndex++;
```

### 2. Didn't preserve trailing holes

For:

```js
new Array(2)
```

there are two holes but no actual properties.

`resultIndex` advances, but `result.length` doesn't.

Fix:

```js
result.length = resultIndex;
```

---

## Production Considerations

- Native `Array.prototype.concat` checks the `Symbol.isConcatSpreadable` property on objects/arrays to control whether an object is spread or added as a single unit.
- Spread operator `[...arr1, ...arr2]` converts holes into `undefined` values (`[1, , 2]` spread becomes `[1, undefined, 2]`), whereas `concat()` preserves sparse holes (`[1, <empty>, 2]`).

---

## ⭐ Revision Notes

### 🔑 Key Facts

* `concat()` **does not mutate** the original arrays.
* Arrays are flattened **one level only**.
* Non-array arguments are added as single values.
* `i in array` detects both own **and inherited** indexed properties.
* A hole is different from an explicit `undefined`.
* Advancing an index variable does **not** automatically increase array length.
* `result.length = resultIndex` preserves trailing holes.

### 🧠 Mental Model

```text
                    myConcat()
                        ↓
                Process `this`
                        ↓
              Process each item
                 ↙           ↘
             Array         Non-array
               ↓               ↓
        copy positions      add one value
               ↓
        index exists?
          ↙       ↘
        yes        no
         ↓          ↓
      copy value   preserve hole
          \         /
           ↓       ↓
          advance resultIndex
                  ↓
       result.length = resultIndex
                  ↓
               result
```

### Common Interview Questions

- What is the difference between `[...a, ...b]` and `a.concat(b)` regarding sparse arrays? → Spread `[...]` fills holes with `undefined`, whereas `concat` preserves sparse holes (`<empty>`).
- How deep does `concat` flatten nested array arguments? → Exactly one level deep (it does not recursively flatten `[1, [2, [3]]]`).
- How do you check if an index exists in a sparse array? → Using the `in` operator (e.g. `i in arr`).

### ⭐ Interview Takeaway

> **For `concat`, think "copy positions", not simply "push values."**

The important edge case is **sparse arrays**: preserve the position even when no value exists there.

### Related

- [[Array Traversal]]
- [[Sparse Arrays]]
- [[Array.prototype.map]]
- [[Array.prototype.filter]]
