---
title: Implement Array.prototype.reduce
aliases:
  - Array.prototype.reduce
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Adobe]]"
  - "[[Amazon]]"
  - "[[Apple]]"
  - "[[ByteDance]]"
  - "[[Pinterest]]"
  - "[[Canva]]"
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[this]]"
  - "[[Array Methods]]"
  - "[[Higher Order Functions]]"
  - "[[Sparse Arrays]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement `Array.prototype.myReduce` to reduce array elements into a single value using a reducer callback function.

## Problem

`Array.prototype.reduce` is a way of "reducing" elements in an array by calling a "reducer" callback function on each element in order and passing along the return value from the previous callback. The final result of running the reducer across all elements of the array is a single value.

Implement `Array.prototype.reduce` as `Array.prototype.myReduce`.

```js
[1, 2, 3].myReduce((prev, curr) => prev + curr, 0); // 6
[1, 2, 3].myReduce((prev, curr) => prev + curr, 4); // 10
```

## Companies

- [[Adobe]]
- [[Amazon]]
- [[Apple]]
- [[ByteDance]]
- [[Pinterest]]
- [[Canva]]

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

Determine whether an initialValue was provided.
If provided, use it as the accumulator.
If not provided, find the first existing array element and use it as the accumulator.
Start iterating from the next index.
Skip sparse-array holes.
Call the reducer with (accumulator, currentValue, index, array).
Store the callback's return value as the new accumulator.
Return the final accumulator.

## 💻 Final Solution

```js
Array.prototype.myReduce = function (callbackFn, initialValue) {
  const noiInitialValue = arguments.length <= 1;
  const len = this.length;

  let k = 0;
  let acc;

  if (noiInitialValue) {
    while (k < len && !(k in this))
      k++;
    if (k >= len) {
      throw new TypeError("Reduce of empty array with no initial value");
    }
    acc = this[k];
    k++;
  } else {
    acc = initialValue;
  }

  for (; k < len; k++) {
    if (k in this) {
      acc = callbackFn(acc, this[k], k, this);
    }
  }

  return acc;
};
```

## 🤔 Why This Works

### Initial Value
```js
[1, 2, 3].myReduce((a, b) => a + b, 10);
```

starts with:

```js
acc = 10
```

Without an initial value:

```js
[1, 2, 3].myReduce((a, b) => a + b);
```

the first existing element becomes the accumulator:

```js
acc = 1
```

### arguments.length

Use:

```js
arguments.length <= 1
```

instead of:

```js
initialValue === undefined
```

because:

```js
myReduce(callback)
```

and:

```js
myReduce(callback, undefined)
```

are different cases.

### Sparse Arrays

For:

```js
[, , 5, 10]
```

the first existing element is 5, not undefined.

```js
while (k < len && !(k in this)) {
  k++;
}
```

`k in this` checks whether the index actually exists.

### Accumulator Update
```js
acc = callbackFn(acc, this[k], k, this);
```

The callback's return value becomes the accumulator for the next iteration.

### Skipping Holes
```js
if (k in this) {
  // call reducer
}
```

Native `reduce()` does not invoke the callback for missing indexes.

## 🐞 Bugs I Made

None.

## Production Considerations

- Native `Array.prototype.reduce` is available in all modern JS runtimes.
- Time: **O(n)**, Space: **O(1)**.

## ⭐ Revision Notes

### Key Facts

- `reduce()` produces one final value.
- With initialValue: accumulator starts with initialValue.
- Without initialValue: first existing element becomes the accumulator.
- Callback signature: `(previousValue, currentValue, currentIndex, array)`.
- Callback's return value becomes the next accumulator.
- Sparse-array holes are skipped.
- Empty array + no initial value → TypeError.
- `arguments.length` distinguishes omitted initialValue from explicitly passing undefined.
- `k in this` checks property/index existence.

### Mental Model

```text
                reduce()
                   │
          initialValue provided?
             /             \
           YES              NO
            │                │
       acc = initial     find first
                          existing element
                              │
                         acc = element
                              │
                              ↓
                       loop remaining
                           indexes
                              │
                       index exists?
                         /          \
                       NO            YES
                       │              │
                     skip       callback(...)
                                    │
                                    ↓
                              new accumulator
                                    │
                                    ↓
                              return acc
```

### Common Interview Questions

- What is the callback signature of `reduce`? → `(accumulator, currentValue, index, array)`.
- What happens if reduce is called on an empty array without initialValue? → Throws `TypeError`.
- How do you detect if `initialValue` was omitted vs passed as `undefined`? → Check `arguments.length <= 1`.

### Interview Takeaways

- **Interview keywords:** prototype method · accumulator pattern · `arguments.length` · sparse arrays · holes · `k in this` · `TypeError`.

### Related

- [[Array Traversal]]
- [[Array.prototype.map]]
- [[Array.prototype.filter]]
