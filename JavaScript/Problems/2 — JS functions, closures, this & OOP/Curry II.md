---
title: Curry II
aliases:
  - Curry II
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
  - "[[Amazon]]"
pattern:
  - "[[Currying]]"
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[Function Length]]"
  - "[[Function.prototype.apply]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-13
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Implement advanced `curry(func)` that accepts any number of arguments per call until at least `func.length` arguments are provided.

## Problem

Implement `curry` accepting a function. Return a function that accepts any number of arguments (not just one at a time) and can be repeatedly called until at least the minimum number of arguments (`func.length`) has been provided. Then invoke `func` with the provided arguments and preserved `this`.

```js
function multiplyThree(a, b, c) {
  return a * b * c;
}
const curried = curry(multiplyThree);
curried(4)(5)(6);    // 120
curried(4)(5, 6);    // 120
curried(4, 5)(6);    // 120
curried(4, 5, 6);    // 120
```

## Companies

- [[Meta]]
- [[Amazon]]

## Pattern

- [[Currying]]
- [[Closure]]

## 🤔 Thought Process

- Generalization of `Curry`: instead of unary steps `(arg) => ...`, each step must accept arbitrary argument counts `(...arg) => ...`.
- Collect accumulated arguments across invocations using rest parameter `...args`.
- Base condition: if `args.length >= func.length`, invoke `func.apply(this, args)`.
- Recursive step: return a function taking `(...arg)`. Spread both accumulated and newly provided arguments `[...args, ...arg]`, calling `curried.apply(this, ...)`.
- Preserves caller `this` and allows batches of arguments per call.

## 💻 Final Solution

```js
export default function curry(func) {

    return function curried(...args){
      if(args.length >= func.length){
        return func.apply(this,args)
      }

      return (...arg) => {
        return arg === undefined ?
        curried.apply(this,args)
        :curried.apply(this,[...args,...arg]);
      }
    }
}
```

## 🤔 Why This Works

- Rest parameter `...arg` collects all arguments passed into subsequent calls into an array.
- Spreading `[...args, ...arg]` creates a fresh array instance for the next invocation, preventing cross-talk between divergent branch calls like `const f4 = curried(4); f4(5); f4(6);`.
- `curried.apply(this, ...)` forwards the `this` binding context.
- Once total arguments accumulated reach or exceed `func.length`, execution terminates and returns result of `func`.

## 🐞 Bugs I Made

None.

## Production Considerations

- `func.length` only reflects parameters up to the first parameter with a default value, and ignores rest parameters (`...rest`).
- Lodash `_.curry` works similarly, supporting arity override: `_.curry(fn, [arity=fn.length])`.
- In production, functions with dynamic arity or zero-arity (`func.length === 0`) invoke immediately on first call.

## ⭐ Revision Notes

### Key Facts

- Curry I vs Curry II: Curry I expects 1 arg per call; Curry II accepts variable number of args `(...arg)` per call.
- Rest parameters + array spreading ensures argument immutability per call branch.
- Termination condition remains identical: `args.length >= func.length`.
- Preserves `this` binding using `apply`.

### Common Interview Questions

- How does Curry II differ from Curry I? Curry II allows multi-argument batches per invocation (e.g. `fn(1, 2)(3)` vs strictly `fn(1)(2)(3)`).
- What happens if `func.length === 0`? If `args.length >= 0`, it executes immediately on first invocation.
- Why return an arrow function inside? Arrow function captures outer `this` lexically, forwarding context into `curried.apply(this, ...)`.

### Interview Takeaways

- Flexible currying (batching args) matches real-world library implementations like Lodash and Ramda.
- Rest parameter `...arg` simplifies handling variable arity compared to `arguments` object.

### Related

- [[Currying]]
- [[Curry]]
- [[Curry III]]
- [[Closure]]
- [[Function Length]]
