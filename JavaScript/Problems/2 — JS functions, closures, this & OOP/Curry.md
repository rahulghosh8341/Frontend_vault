---
title: Curry
aliases:
  - Curry
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Amazon]]"
  - "[[Meta]]"
  - "[[Rippling]]"
pattern:
  - "[[Currying]]"
  - "[[Closure]]"
concepts:
  - "[[Closure]]"
  - "[[Function Length]]"
  - "[[Function.prototype.apply]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-12
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Implement `curry(fn)` to convert a multi-argument function into a sequence of unary/partial functions until `fn.length` arguments are collected.

## Problem

Implement `curry(fn)` to return a curried version of `fn`. Collect one argument at a time until at least `fn.length` arguments have been provided, then call `fn` with collected arguments.

Calls with no arguments should be ignored and return another function. Preserve the `this` value from the first call in the chain when `fn` is finally invoked.

```js
function add(a, b) {
  return a + b;
}

const curriedAdd = curry(add);
curriedAdd(3)(4); // 7

const alreadyAddedThree = curriedAdd(3);
alreadyAddedThree(4); // 7
```

## Companies

- [[Amazon]]
- [[Meta]]
- [[Rippling]]

## Pattern

- [[Currying]]
- [[Closure]]

## 🤔 Thought Process

Currying means **collecting arguments across multiple function calls until we have enough to call the original function**.

The important value is:

```js
func.length
```

It tells us how many arguments the original function expects.

For:

```js
function add(a, b) {
  return a + b;
}
```

```text
func.length = 2
```

So:

```text
curried(3)
→ collected: [3]
→ not enough → return function

(4)
→ collected: [3, 4]
→ enough → execute fn
```

## 💻 Final Solution

```js
export default function curry(func) {
  return function curried(...args){

    if(args.length  >= func.length){
      return func.apply(this,args);
    }
    return (arg)=> 
      arg === undefined 
      ?
      curried.apply(this,args)
      : curried.apply(this,[...args,arg]);
  }
} 
```

## 🤔 Why This Works

The returned function forms a **closure** over the previously collected arguments.

Conceptually:

```text
curry(add)
   ↓
curried([ ])
   ↓
curried([3])
   ↓
curried([3, 4])
   ↓
add(3, 4)
```

Each new function remembers the arguments from the previous call.

The key check is:

```js
args.length >= func.length
```

Once enough arguments are available, execute `func`.

## 🐞 Bugs I Made

### Shared `collectedArgs`

An initial approach like:

```js
let collectedArgs = [];
```

outside the returned function causes different chains to share the same arguments.

For example:

```js
curried(2);
curried(3);
```

For a one-argument function, these must be independent:

```text
curried(2) → 4
curried(3) → 9
```

They cannot share the same `collectedArgs`.

### Important lesson

Each currying chain needs its **own closure/state**.

## Production Considerations

- `func.length` ignores rest parameters (`...args`) and default parameters (stops counting at first parameter with default).
- For practical usage (like Lodash `_.curry`), accepting multiple arguments per invocation (`curried(1, 2)(3)`) provides greater utility.
- Memory consumption increases linearly with call chain depth due to closure retaining reference frames until resolution.

## ⭐ Revision Notes

### Key Facts

* `func.length` = number of parameters expected by `func`.
* Arguments can arrive across multiple calls.
* Empty calls `()` should be ignored.
* Not enough arguments → return another function.
* Enough arguments → execute `func`.
* Each chain needs independent collected arguments.
* `this` from the **first call** must be preserved until `func` executes.
* `apply()` can be used to invoke `func` with the preserved `this` and collected arguments.

### Common Interview Questions

- What does `fn.length` return when default or rest parameters are present? Only counts parameters before the first default parameter; rest parameters are ignored.
- Difference between Currying and Partial Application? Currying transforms a function into a unary chain of functions; partial application fixes a subset of arguments and produces a function expecting the remainder.
- How to prevent chain state collisions? Avoid mutable shared state across invocations; pass accumulated argument copies down recursive calls.

### 🧠 Mental Model

```text
            curry(fn)
                ↓
        How many does fn need?
             fn.length
                ↓
        Collect arguments
                ↓
       enough arguments?
          /           \
        NO             YES
        ↓               ↓
 return function      fn(...)
        ↓
 remember arguments
        ↓
     next call
        ↓
 collect more
```

For `square`:

```js
function square(x) {
  return x * x;
}
```

```text
fn.length = 1

curried(2)
   ↓
1 >= 1
   ↓
square(2)
   ↓
4
```

That's why:

```js
curried(2) // 4
curried(3) // 9
```

must be independent.

### Interview Takeaways

The core of currying is:

> **Keep returning functions until the number of collected arguments reaches `fn.length`; then execute the original function.**

The three concepts this problem combines are:

```text
Closure
   +
Higher-Order Functions
   +
this / apply
   ↓
Currying
```

For this particular problem, the hardest part isn't detecting the final call—it is **maintaining independent argument state for each chain while preserving the first call's `this`**.

### Related

- [[Currying]]
- [[Closure]]
- [[Function Length]]
- [[Curry II]]
- [[Curry III]]
