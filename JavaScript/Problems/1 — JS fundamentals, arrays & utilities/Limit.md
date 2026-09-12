---
title: Implement limit(func, n) to cap a function's invocation count
aliases:
  - Limit
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Closure]]"
concepts:
  - "[[Higher-Order Functions]]"
  - "[[Function.prototype.apply]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Restrict a function's invocation to at most n times; subsequent calls return the result of the last invocation.

## Problem

Implement a function that accepts a callback and a number `n`, which restricts invocation of the callback to at most `n` times. Subsequent calls to the created function will return the result of the last invocation of the callback. The callback is invoked with the `this` binding and arguments from the created function.

**Examples**

```js
let i = 1;

function incrementBy(value) {
  i += value;
  return i;
}

const incrementByAtMostThrice = limit(incrementBy, 3);
incrementByAtMostThrice(2); // i is now 3; the function returns 3.
incrementByAtMostThrice(3); // i is now 6; the function returns 6.
incrementByAtMostThrice(4); // i is now 10; the function returns 10.
incrementByAtMostThrice(5); // i is still 10 because this is the 4th invocation; the function returns 10 because it is the result of the last invocation.
i = 4;
incrementByAtMostThrice(2); // i is still 4 because it is not modified; the function still returns 10.
```

## Pattern

- [[Closure]]

## 🤔 Thought Process

* `calls` tracks how many times `func` has actually executed.
* `result` stores the result of the last execution.
* Return a closure so both variables persist between calls.
* If the limit hasn't been reached:
  * execute `func`
  * save its result
  * increment `calls`
  * return the result
* Once `n` calls have happened, simply return the saved result.

## 💻 Final Solution

```js
/**
 * @param {(...args: Array<unknown>) => unknown} func
 * @param {number} n
 * @returns {(...args: Array<unknown>) => unknown}
 */
export default function limit(func, n) {
  let calls = 0;
  let result;

  return function(...args) {
    if (calls === n) {
      return result;
    }
    result = func.apply(this, args);
    calls++;
    return result;
  };
}
```

## 🤔 Why This Works

```js
if (calls === n) {
  return result;
}
```

After the nth execution:

```text
calls = n
```

Every subsequent invocation skips:

```js
func.apply(this, args);
```

and returns the previously saved result.

For `n = 3`:

```text
Call 1 → func → result = 3  → calls = 1
Call 2 → func → result = 6  → calls = 2
Call 3 → func → result = 10 → calls = 3
Call 4 → blocked → return 10
Call 5 → blocked → return 10
```

`apply(this, args)` correctly preserves both the `this` binding and the arguments.

## 🐞 Bugs I Made

No functional bugs.

One small improvement:

```js
if (calls == n)
```

Prefer strict equality:

```js
if (calls === n)
```

Also, the condition could conceptually be:

```js
if (calls >= n)
```

This is slightly more defensive, although `_===` is completely sufficient because the code increments `calls` exactly once per invocation.

## Production Considerations

- Lodash ships `_.before(n, func)` with identical semantics.
- For rate-limiting over **time**, use `Debounce` / `Throttle` instead.
- For memoizing the **result of each unique input**, use `Memoize`.

## ⭐ Revision Notes

### 🔑 Key Facts

* Closure keeps `calls` and `result` alive between invocations.
* `apply(this, args)` forwards:
  * the calling context (`this`)
  * all arguments.
* The callback executes at most `n` times.
* Calls after the limit don't execute the callback.
* They return the last callback result.
* Time per allowed call: O(1) excluding the callback itself.
* Extra state: O(1).

### 🧠 Mental Model

Think of it as a counter + cached result:

```text
             limit(func, 3)
                    ↓
              returned function
                    ↓
          ┌─────────┴─────────┐
          ↓                   ↓
       calls               result
          ↓                   ↓
      how many?          last answer
          │                   │
          └───────┬───────────┘
                  ↓
          calls < n ?
          /       \
        YES        NO
         ↓          ↓
       func()    return result
         ↓
    save result
    increment calls
```

### Common Interview Questions

- How is `limit` different from `Memoize`? → `Memoize` caches by input args; `limit` caps total invocations regardless of inputs.
- How does it differ from `Debounce` / `Throttle`? → `limit` is a hard invocation cap (no time element). `Debounce` waits for quiet; `Throttle` enforces a minimum gap between calls.
- Why use `apply(this, args)` instead of calling `func(...args)`? → `apply` preserves the caller's `this` binding, which is part of the problem requirements.

### Interview Takeaways

* Closure + counter + cached result = hard cap on invocations.
* Use `apply(this, args)` (not `func(...args)`) to preserve `this`.
* Strict equality (`_===`) over loose (`_==`).

### Related

- [[Closure]]
- [[Debounce]]
- [[Debounce II]]
- [[Function.prototype.apply]]
- [[Function Chaining]]