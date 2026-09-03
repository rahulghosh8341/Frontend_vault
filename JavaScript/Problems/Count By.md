---
title: Implement countBy(array, iteratee)
aliases:
  - Count By
difficulty: Medium
time: 15 min
languages:
  - JavaScript
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Frequency Counting]]"
  - "[[Iteratee Functions]]"
solved: true
solvedDate: 2026-09-04
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Creates an object counting the frequency of elements in an array grouped by the result of an iteratee function.

## Problem

Implement `countBy(array, iteratee)` so it creates an object whose keys are the results of calling `iteratee` on each element of `array`. The value of each key is the number of elements that produced that result. Do not modify the original array.

```js
countBy([6.1, 4.2, 6.3], Math.floor);
// => { '4': 1, '6': 2 }

countBy([{ n: 3 }, { n: 5 }, { n: 3 }], (o) => o.n);
// => { '3': 2, '5': 1 }

countBy([], (o) => o); 
// => {}

countBy([{ n: 1 }, { n: 2 }], (o) => o.m); 
// => { undefined: 2 }
```

---

## Pattern

- [[Array Traversal]]

---

## 🤔 Thought Process

1. Create an empty result object.
2. Loop through every element in the array.
3. Pass each element to `iteratee`.
4. Use the returned value as the key.
5. If the key already has a count, increment it.
6. Otherwise, initialize it to `1`.
7. Return the result.

---

## 💻 Final Solution

```js
export default function countBy(array, iteratee) {
  let result = {};
  if(array.length == 0)
    return {};
  for(let item of array){
    let item_result;
    item_result = iteratee(item);
    result[item_result] =  (result[item_result] || 0) + 1;
  }
  return result;
}
```

---

## 🤔 Why This Works

### The iteratee determines the key

```js
const key = iteratee(item);
```

For:

```js
countBy([6.1, 4.2, 6.3], Math.floor);
```

the transformations are:

```text
6.1 → 6
4.2 → 4
6.3 → 6
```

Therefore:

```js
{
  6: 2,
  4: 1
}
```

### Counting pattern

```js
result[key] = (result[key] || 0) + 1;
```

First occurrence:

```text
undefined → 0 + 1 → 1
```

Second occurrence:

```text
1 → 1 + 1 → 2
```

This avoids needing a separate `if` condition.

### `undefined` is also a valid key

```js
countBy([{ n: 1 }, { n: 2 }], o => o.m);
```

Both produce:

```js
undefined
```

so:

```js
{
  undefined: 2
}
```

---

## 🐞 Bugs I Made

### Didn't handle a new key

You had:

```js
if (Object.hasOwn(result, item_result))
  result[item_result] = (result[item_result] || 0) + 1;
```

This only handles keys that **already exist**.

For a new key, nothing happens.

Simpler:

```js
result[item_result] = (result[item_result] || 0) + 1;
```

---

## Production Considerations

- In Lodash / production utilities, `iteratee` can be a shorthand string (e.g. `'length'` or `'n'`) in addition to a function.
- Using `reduce()` is a popular functional alternative in JS: `array.reduce((acc, item) => { const k = iteratee(item); acc[k] = (acc[k] || 0) + 1; return acc; }, {})`.

---

## ⭐ Revision Notes

### 🔑 Key Facts

* `iteratee` determines the grouping key.
* Each element contributes exactly `1` to its key's count.
* First occurrence → count `1`.
* Subsequent occurrences → increment.
* `undefined` can be an object key.
* Object property keys are converted to strings.
* Empty array naturally returns `{}`.

### 🧠 Mental Model

```text
Array
  ↓
for each element
  ↓
iteratee(element)
  ↓
   key
  ↓
result[key] = result[key] + 1
  ↓
aggregate object
```

### Common Interview Questions

- What happens if the `iteratee` returns `undefined` or `null`? → They are stringified to `"undefined"` or `"null"` as object keys and counted normally.
- How can you write `countBy` using `Array.prototype.reduce`? → By passing an initial empty object `{}` and setting `acc[key] = (acc[key] || 0) + 1`.
- How does `countBy` differ from `groupBy`? → `groupBy` collects matching elements into an array (`{ key: [elem1, elem2] }`), whereas `countBy` returns the count (`{ key: 2 }`).

### ⭐ Interview Takeaway

> **`countBy` is essentially a frequency counter where the key is produced by an iteratee.**

Core pattern to remember:

```js
const key = iteratee(item);
result[key] = (result[key] || 0) + 1;
```

This is the cleanest approach for this problem.

### Related

- [[Array Traversal]]
- [[Group By]]
- [[Array.prototype.reduce]]
