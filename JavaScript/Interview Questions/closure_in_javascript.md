---
title: What is a closure in JavaScript, and how/why would you use one?
aliases: closure in JavaScript
tags:
  - javascript
  - interview
  - closure
  - scope
solved: true
solvedDate: 2026-09-07
type: quiz
---

> [!info] 🟢 Difficulty: Medium 📂 Category: JavaScript Interview Question ⏱️ Review Time: ~8 minutes

## TL;DR

A **closure** is created when a function retains access to its **lexical scope** even when that function is executed outside that scope.

In simple terms:

> A function remembers the variables that were in scope when it was created.

```js
function outerFunction() {
  const outerVar = 'I am outside of innerFunction';

  function innerFunction() {
    console.log(outerVar);
  }

  return innerFunction;
}

const inner = outerFunction();
inner(); // "I am outside of innerFunction"
```

Closures are commonly used for private state, callbacks, event handlers, asynchronous code, factories, partial application, and memoization. citeturn0search0

## Interview Answer (30–60 sec)

> A closure is when a function retains access to its lexical scope even when that function is executed outside that scope. This happens because the function keeps a reference to the lexical environment where it was created. A common use case is encapsulating private state. For example, a counter factory can create a `count` variable and return functions that can access and update it, while outside code cannot access `count` directly. Closures are also heavily used in callbacks, event handlers, asynchronous code, factories, and memoization.

## Key Takeaways

- **Closure = function + retained lexical environment.**
- JavaScript uses **lexical scoping**.
- A function remembers variables from the scope where it was **created**, not where it is called.
- The outer function can finish executing while its lexical environment remains reachable through the closure.
- Closures can create **private state**.
- Each factory call can create an independent closure and independent state.
- Closures are neither inherently synchronous nor asynchronous.
- Long-lived closures can keep referenced objects alive.

## 🧠 Mental Model

```text
Function
   +
Lexical Environment
   ↓
Closure
```

```js
function makeCounter() {
  let count = 0;

  return () => ++count;
}

const counter = makeCounter();

counter(); // 1
counter(); // 2
counter(); // 3
```

Think:

```text
create variable
     ↓
create inner function
     ↓
inner function captures variable
     ↓
outer function returns
     ↓
captured variable remains reachable
```

## Common Interview Traps

### "A closure is just an inner function."

Not quite. The important part is that the function retains access to variables from its surrounding lexical scope.

### "The closure stores a snapshot of the variables."

Not exactly. It retains access to the lexical environment, so multiple calls can observe and update the same binding.

### "The outer function must still be running."

Incorrect. The outer function can finish; the captured environment remains reachable through the closure.

### "Closures are asynchronous."

Incorrect. A closure can be invoked synchronously or asynchronously. Closure is about lexical scope, not timing.

### "`let` creates closures, while `var` does not."

Incorrect. Both can participate in closures. Their difference is especially important in loops because `let` is block-scoped while `var` is function-scoped.

## Common Follow-ups

- What is lexical scoping?
- How does a closure retain state?
- How are closures used for data privacy?
- What is the difference between a closure and a class?
- Can closures cause memory leaks?
- Why does `var` in a loop produce unexpected closure behavior?
- What is a stale closure in React?
- How are closures used in React hooks?
- How can closures be used for memoization?
- What is partial application?

## My Notes


# Detailed Reference

## Definition

GreatFrontEnd uses the definition from *You Don't Know JS*:

> Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

Functions have access to variables that were in their scope at the time of creation. A closure retains access to those variables even after the outer function has finished executing. citeturn0search0

## How a closure retains state

The function object keeps a reference to the lexical environment where it was created, even after the outer call returns. The retained environment is not simply a snapshot of each value; multiple calls can observe updated private state. citeturn0search0

```js
function createCounter() {
  let count = 0;

  return function () {
    count += 1;
    return count;
  };
}

const counter = createCounter();

console.log(counter()); // 1
console.log(counter()); // 2
```

## Understanding closures

### 1. Lexical scoping

JavaScript uses lexical scoping, so a function's access to variables is determined by where the function is written in the source code.

### 2. Function creation

When a function is created, it keeps access to its lexical scope.

### 3. Maintaining state

Closures can maintain state because captured variables are not directly accessible from outside the closure. citeturn0search0

## ES6 syntax and closures

Arrow functions can also form closures:

```js
const createCounter = () => {
  let count = 0;

  return () => {
    count += 1;
    return count;
  };
};

const counter = createCounter();

counter(); // 1
counter(); // 2
```

## Closures compared with classes

Closures and classes can both encapsulate state.

```js
function makeCounter() {
  let count = 0;

  return {
    inc: () => ++count,
    get: () => count,
  };
}

class Counter {
  #count = 0;

  inc() {
    return ++this.#count;
  }

  get() {
    return this.#count;
  }
}
```

| Concern | Closure | Class with private fields |
|---|---|---|
| Privacy | Lexical scope | Private field |
| Memory | New closure scope/functions per factory call | State per instance; methods can be shared |
| `this` | Not required for closed-over state | Methods use `this` |
| Prototype sharing | Not supported for closure functions | Supported |
| Typical use | Factories, callbacks, partial application, functional programming | Long-lived objects, inheritance |

GreatFrontEnd's guidance is that closures fit small numbers of encapsulated instances when inheritance is not needed, while classes fit cases involving many instances, prototype sharing, inheritance, or `instanceof`. citeturn0search0

## Closures in React

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Increment
    </button>
  );
}
```

`handleClick` forms a closure over values from its surrounding render scope. citeturn0search0

### Stale closures in `useEffect`

A `useEffect` callback captures the values it references when that effect runs. With an empty dependency array, a callback can continue referring to the value captured by the first effect execution. citeturn0search0

Common solutions include:

```js
useEffect(() => {
  const interval = setInterval(() => {
    setCount((prev) => prev + 1);
  }, 1000);

  return () => clearInterval(interval);
}, []);
```

The functional updater avoids needing to capture the current state value.

## Memoization with closures

A closure can hold a cache:

```js
function memoize(fn) {
  const cache = new Map();

  return function (...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) return cache.get(key);

    const result = fn.apply(this, args);
    cache.set(key, result);

    return result;
  };
}
```

The `cache` is accessible only through the returned function, demonstrating the private-state property of closures. citeturn0search0

## Common example: `var` in a loop

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// 3
// 3
// 3
```

All callbacks close over the same `var` binding. By the time the callbacks run, the loop has finished and `i` is `3`.

Using `let` creates a separate binding per iteration:

```js
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// 0
// 1
// 2
```

## Why use closures?

1. **Data encapsulation** — create private variables and functions.
2. **Functional programming** — enable patterns such as partial application and currying.
3. **Event handlers and callbacks** — retain access to variables from the creation scope.
4. **Module patterns** — create private and public parts. citeturn0search0

## Common questions

### When is a closure preferable to a class?

Use a closure when a small number of instances need encapsulated state and inheritance is not needed. Use a class when many instances share behavior, prototype sharing is useful, or inheritance and `instanceof` are required. citeturn0search0

### Can closures cause memory leaks?

A long-lived closure can keep referenced objects alive. For example, an event listener that is never removed can keep its closure and referenced objects reachable. citeturn0search0

### Are closures synchronous or asynchronous?

Neither. A closure is a function that captures lexical scope. It can be invoked synchronously or asynchronously. citeturn0search0
