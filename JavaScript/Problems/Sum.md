---
title: Implement a sum function that accepts a number and can be called repeatedly with more numbers
aliases:
  - Sum
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
pattern:
  - "[[Function Chaining]]"
concepts:
  - "[[Closures]]"
  - "[[Currying]]"
  - "[[Partial Application]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement a function `sum` that can be called repeatedly with numbers, returning the total when invoked without arguments.

## Problem

Implement a `sum` function that accepts a number and can be called repeatedly with more numbers. Calling the function without an argument will sum up all the arguments so far and return the total.

```js
sum(1)();        // 1
sum(1)(2)();     // 3
sum(1)(2)(-3)(); // 0
```

> A partially applied chain can be reused. Extending one branch should not change the result of another branch created from the same earlier call.

---

## Companies

- [[Amazon]]

---

## Pattern

- [[Function Chaining]]

---

## 🤔 Thought Process

1. `sum(value)` needs to return a function.
2. Store the current total in a `const`.
3. When the returned function receives `undefined`, return the total.
4. Otherwise, calculate a **new total**.
5. Create a new stage using:

   ```js
   return sum(total + nextValue);
   ```
6. This creates a new closure with its own `const total`.

---

## 💻 Final Solution

```js
export default function sum(value) {
  const total = value;

  return function (nextValue){
    if(nextValue == undefined){
      return total;
    }
    return sum(total + nextValue)
  }
}
```

---

## 🤔 Why This Works

### Fresh Closure Per Call

```js
return sum(total + nextValue);
```

Calling `sum()` again creates a **new execution context** and therefore a new:

```js
const total = value;
```

Example:

```text
sum(1)
  ↓
total = 1
  ↓
(2)
  ↓
sum(3)
  ↓
total = 3
  ↓
(5)
  ↓
sum(8)
  ↓
total = 8
```

The previous `total` is never modified.

### Independent Branches

```js
const a = sum(1);

const b = a(2);
const c = a(5);
```

Each branch gets its own closure:

```text
        a
        │
     total=1
      /   \
     ↓     ↓
   a(2)   a(5)
     ↓     ↓
 total=3 total=6
```

Therefore:

```js
b(); // 3
c(); // 6
```

---

## 🐞 Bugs I Made

### Trying to redeclare `total` using itself

You had:

```js
const total = total + nextValue;
```

The new `total` cannot use itself during its own initialization.

Instead, create a new `sum()` invocation:

```js
return sum(total + nextValue);
```

This gives the next closure a fresh `total`.

### Mutating the existing total

Avoid:

```js
total += nextValue;
```

The problem specifically requires reusable branches. Mutating shared state could make different branches affect each other.

---

## Production Considerations

- Infinite currying / variadic chaining with termination conditions (`sum(1)(2)()`) is standard in functional programming libraries (like Ramda / Lodash `curry`).
- Avoid using `if (!nextValue)` for termination checking because `0` is a falsy value and `sum(0)` would terminate prematurely. Check `nextValue === undefined` instead.

---

## ⭐ Revision Notes

### 🔑 Key Facts

* **Closure** preserves state between calls.
* `const` means each closure's `total` isn't reassigned.
* `sum(total + nextValue)` creates a **new closure**.
* Each chain stage owns its own total.
* Use `nextValue === undefined` to terminate.
* Don't use `if (!nextValue)` because `0` is a valid value.

### 🧠 Mental Model

```text
sum(1)
  │
  └── closure { total: 1 }
          │
        (2)
          │
          ↓
       sum(3)
          │
          └── closure { total: 3 }
                  │
                (5)
                  │
                  ↓
               sum(8)
                  │
                  ↓
                 ()
                  │
                  ↓
                  8
```

### Common Interview Questions

- Why must `sum(total + nextValue)` return a new function invocation rather than mutating an internal variable? → To ensure immutability and allow independent branching without shared state side-effects.
- How can you handle value inspection without an empty function call `()`? → Override `valueOf` or `toString` on the returned function (`fn.valueOf = () => total`), though explicit `()` is safer.

### ⭐ Interview Takeaways

> **When a chained function needs independent state, don't mutate the existing closure. Return a new function/closure containing the updated state.**

For this problem, the elegant pattern is:

```js
return sum(total + nextValue);
```

**Pattern to remember:** **closure + recursion + immutable state + termination with `undefined`**.

### Related

- [[Function Chaining]]
- [[Closures]]
- [[Currying]]
- [[Curry]]
- [[Curry II]]
