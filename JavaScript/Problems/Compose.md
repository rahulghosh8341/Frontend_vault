---
title: Implement compose to chain functions in reverse order
aliases:
  - Compose
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Higher Order Mapping]]"
concepts:
  - "[[Function Composition]]"
  - "[[Closure]]"
solved: true
solvedDate: 2026-09-10
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Compose multiple functions into a single function, executing them right-to-left.

## Problem

Implement a function `compose(...fns)` that takes multiple functions as arguments and returns a new function that applies those functions in reverse order. The output of one function becomes the input of the next function, creating a chain of function compositions.

If no functions are passed to `compose`, return a new function that simply returns the input it receives (identity function).

```js
compose(f3, f2, f1)(value) // executes f1 → f2 → f3
```

Each function accepts a single parameter.

## Pattern

- [[Higher Order Mapping]]

## 🤔 Thought Process

* `compose` executes functions **right-to-left**.
* Start with the input value as `result`.
* Iterate from the last function to the first.
* Pass current `result` into each function, replace `result` with the return value.
* Return the final `result`.
* When no functions are provided, return the input unchanged.

## 💻 Final Solution

```js
export default function compose(...fns) {
  return function(value) {
    let result = value;
    for (let i = fns.length - 1; i >= 0; i--) {
      result = fns[i](result);
    }
    return result;
  };
}
```

## 🤔 Why This Works

The loop runs from the last function (`fns.length - 1`) down to the first (`0`). Each iteration replaces `result` with the output of the current function. This naturally handles:

- **Empty `fns`**: loop doesn't run, returns `value` unchanged.
- **Single function**: runs once, returns its result.
- **Multiple functions**: executes right-to-left as required.

## 🐞 Bugs I Made

* **Wrong loop direction**: must iterate from end to start, not start to end.
* **Wrong accumulator**: don't start at `0` or add results; start with the input value.
* **Wrong function call**: pass `result`, not `args`.
* **Wrong return**: need to return the composed function, not the result of one call.

## Production Considerations

- Lodash provides `_.compose` with identical semantics.
- For large numbers of functions, consider using a pipeline approach for readability.
- Functions should be pure for predictable composition.

## ⭐ Revision Notes

### 🔑 Key Facts

* `compose` executes functions **right-to-left**.
* `fns.length - 1` is the last function's index.
* `result` tracks the current value through the chain.
* Empty `fns` → identity function.
* Time complexity: O(n) where n is number of functions.

### 🧠 Mental Model

```text
value
  ↓
 fns[fns.length - 1]
  ↓
 fns[fns.length - 2]
  ↓
 ...
  ↓
 fns[0]
  ↓
final result
```

### Common Interview Questions

- How would you compose more than 3 functions? The same pattern extends — just iterate over the array in reverse.
- What if functions have different arities? The problem guarantees single-parameter functions.
- Why not use `Array.prototype.reduceRight`? `reduceRight` works, but a simple loop is clearer for interview settings.

### Interview Takeaways

* `compose` = reverse iteration over functions, accumulating a single value.
* Use `fns.length - 1` as starting index.
* Return the composed function, not the result of one call.

### Related

- [[Higher Order Mapping]]
- [[Closure]]
- [[Function.prototype.apply]]
- [[Function Chaining]]

## Related Concepts

- [[Lexical Scope]]
- [[Function Composition]]
- [[Currying]]