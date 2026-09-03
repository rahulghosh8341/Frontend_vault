---
title: Implement cycle() that returns a function cycling through provided values
aliases:
  - Cycle
difficulty: Easy
time: 15 min
languages:
  - JavaScript
companies:
  - None
pattern:
  - "[[Closure]]"
concepts:
  - "[[Rest Parameters]]"
  - "[[Closures]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement a function that takes one or more values and returns a function that cycles through those values each time it is called.

## Problem

Implement a function that takes one or more values and returns a function that cycles through those values each time it is called.

```js
const helloFn = cycle('hello');
console.log(helloFn()); // "hello"
console.log(helloFn()); // "hello"

const onOffFn = cycle('on', 'off');
console.log(onOffFn()); // "on"
console.log(onOffFn()); // "off"
console.log(onOffFn()); // "on"
```

## Companies

- None

## Pattern

- [[Closure]]

## 🤔 Thought Process

1. `cycle()` needs to accept any number of values → use rest parameter `...values`.
2. The returned function must remember which value comes next → create `index` inside `cycle()`.
3. Each call uses the current index.
4. Increment `index` after getting the value.
5. Use `% values.length` so the index wraps back to `0`.

## 💻 Final Solution

```js
export default function cycle(...values) {
  let index = 0;

  return function () {
    let current = values[index % values.length]
    index++;
    return current;
  };
}
```

## 🤔 Why This Works

### Closure

```js
let index = 0;
```

belongs to `cycle()`, but the returned function still has access to it.

So `index` survives between calls:

```text
call 1 → index 0 → "on" → index 1
call 2 → index 1 → "off" → index 2
call 3 → index 2 → 2 % 2 = 0 → "on"
```

### Modulo for Cycling

```js
values[index % values.length]
```

For two values:

```text
0 % 2 → 0
1 % 2 → 1
2 % 2 → 0
3 % 2 → 1
```

Therefore the index automatically cycles.

## 🐞 Bugs I Made

None. Your solution is correct for the stated behavior.

## 🔑 Key Facts

* `...values` collects all arguments into an array.
* The returned function creates the **closure**.
* `index` persists between function calls.
* `% values.length` wraps the index back to the beginning.
* Increment the index **after** retrieving the current value.

## 🧠 Mental Model

```text
cycle('on', 'off')
       ↓
values = ['on', 'off']
index = 0
       ↓
returned function
       ↓
call()
  ↓
values[index % length]
  ↓
"on"
  ↓
index++
  ↓
next call → "off"
  ↓
next call → "on"
```

## ⭐ Interview Takeaway

**"Need a function to remember state between calls?" → Think closure.**

Here the closure remembers `index`, while `%` handles the cycling.

### Common Interview Questions

- Why does `index` persist between calls? → It is captured in a closure; the returned function references the outer `index` variable.
- How do you make the cycle repeat? → Use modulo (`%`) against `values.length`.

### Interview Takeaways

- **Interview keywords:** closure · rest parameters · state persistence · modulo cycling.

### Related

- [[Closure]]
- [[Debounce]]