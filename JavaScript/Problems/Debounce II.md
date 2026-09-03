---
title: Implement a function `debounce(func, wait)` that delays invoking `func` until after `wait` milliseconds have elapsed since the last time the debounced function was invoked, with `cancel()` and `flush()` methods.
aliases:
  - Debounce II
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Airbnb]]"
  - "[[Google]]"
  - "[[Lyft]]"
  - "[[Meta]]"
  - "[[Walmart]]"
  - "[[Yelp]]"
  - "[[Microsoft]]"
  - "[[LinkedIn]]"
pattern:
  - "[[Debouncing]]"
concepts:
  - "[[Closures]]"
  - "[[setTimeout]]"
  - "[[this]]"
  - "[[call, apply and bind]]"
solved: true
solvedDate: 2026-08-26
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Advanced debounce with `cancel()` and `flush()` methods. The returned function must preserve `this` and arguments from the most recent invocation, and provide methods to cancel or immediately flush pending invocations.

## Problem

Debouncing controls how often a function can execute over time. When a JavaScript function is debounced with a wait time of `wait` milliseconds, it runs only after `wait` milliseconds have elapsed since the debounced function was last called.

Implement a debounce function that accepts a callback function and a wait duration. Calling `debounce()` returns a function that debounces invocations of the callback function following the behavior described above.

The returned function accepts the same arguments as the callback. When the callback eventually runs, it receives the `this` value and arguments from the most recent invocation of the debounced function.

Additionally, the debounced function comes with two extra methods:

- `cancel()` method to cancel pending invocations.
- `flush()` method to immediately invoke any delayed invocations.

**Examples:**

```js
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: Cancel the delayed increment.
debouncedIncrement.cancel();

// t = 100: increment() was not invoked and i is still 0.
```

```js
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: i is still 0 because 100ms have not passed.
// t = 51:
debouncedIncrement.flush(); // i is now 1 because flush() causes the callback to be invoked immediately.

// t = 100: i is already 1. The callback has been called before
// and won't be called again.
```

## Companies

- [[Airbnb]]
- [[Google]]
- [[Lyft]]
- [[Meta]]
- [[Walmart]]
- [[Yelp]]
- [[Microsoft]]
- [[LinkedIn]]

## Pattern

- [[Debouncing]]

## 🤔 Thought Process

Start with the basic debounce implementation: keep a timer in the closure and reset it on every invocation.

Store the latest `this` and arguments when the debounced function is called:

```js
lastThis = this;
lastArgs = args;
```

Create `cancel()` and `flush()` as properties on the returned function.

`cancel()` only cancels the pending timer and clears the timer reference.

`flush()` checks whether a timer is pending. If so:
- cancel the timer,
- immediately invoke `func`,
- use the latest `this` and arguments.

Return the debounced function with its attached methods.

## 💻 Final Solution

```js
/**
 * @typedef {((...args: Array<unknown>) => void) & {
 *   cancel: () => void,
 *   flush: () => void,
 * }} DebouncedFunction
 */

/**
 * @param {Function} func
 * @param {number} [wait=0]
 * @return {DebouncedFunction}
 */
export default function debounce(func, wait) {
  let timer, lastThis, lastArgs;

  let debounced = function(...args) {
    lastThis = this;
    lastArgs = args;
    clearTimeout(timer);
    timer = setTimeout(() => {
      func.call(lastThis, ...lastArgs);
      timer = undefined;
    }, wait);
  }

  debounced.cancel = function() {
    clearTimeout(timer);
    timer = undefined;
  }

  debounced.flush = function() {
    if (timer) {
      clearTimeout(timer);
      timer = undefined;
      func.call(lastThis, ...lastArgs);
    }
  }
  return debounced;
}
```

## 🤔 Why This Works

Functions are objects in JavaScript, so methods can be attached directly:

```js
debounced.cancel = function () {};
debounced.flush = function () {};
```

All three functions share the same closure:

```
debounce()
│
├── timer
├── lastThis
├── lastArgs
│
├── debounced()
├── cancel()
└── flush()
```

The latest invocation updates:
- `lastThis`  → latest caller
- `lastArgs`  → latest arguments
- `timer`     → latest pending timer

Therefore `flush()` can execute:

```js
func.call(lastThis, ...lastArgs);
```

even though `flush()` itself has a different `this`.

## 🐞 Bugs I Made

Initially stored `lastThis` and `lastArgs` inside the timer, which was too late. They must be captured when the debounced function is called.

Initially used:

```js
func.call(this, args);
```

which passes the entire arguments array as one argument.

Corrected it to:

```js
func.call(lastThis, ...lastArgs);
```

Learned that `flush()` cannot rely on its own `this`; it must use the `this` saved from the latest debounced invocation.

## Production Considerations

- **Lodash `_.debounce`**: Production libraries like Lodash support option objects with `leading` (execute at start), `trailing` (execute at end, default), and `maxWait` (guarantee maximum latency).
- **Cancellation & Flushing**: Advanced debounce functions attach `.cancel()` (to clear pending timers) and `.flush()` (to immediately trigger delayed invocations) methods onto the returned wrapper.
- **Memory Cleanup**: Resetting `timer = undefined` inside the timer callback helps with state tracking and garbage collection.
- **`wait = 0` Execution**: Setting `wait` to `0` defers execution to the browser macro-task queue via `setTimeout`, making execution asynchronous rather than synchronous.

## ⭐ Revision Notes

### Key Facts

- **Pattern:** [[Debouncing]]
- Functions are objects → methods can be attached as properties.
- `cancel()` → cancel pending invocation; do not execute `func`.
- `flush()` → cancel pending timer and execute immediately.
- Store latest:
  - `lastThis = this;`
  - `lastArgs = args;`
- Execute with:
  - `func.call(lastThis, ...lastArgs);`
- `flush()` should do nothing when there is no pending invocation.
- `debounced()` must be a regular function to dynamically capture the caller's `this`.
- The `setTimeout` callback can be an arrow because it should preserve the wrapper's `this`.
- `cancel()` and `flush()` are attached to the returned function.
- Debounce II = Debounce I + closure state + `cancel()` + `flush()`.

### Common Interview Questions

- **How does `flush()` differ from `cancel()`?** → `cancel()` discards the pending invocation without executing it; `flush()` executes it immediately and clears the timer.
- **Why must `debounced` be a regular function, not an arrow function?** → To dynamically capture the caller's `this` at invocation time.
- **Why does `flush()` use `lastThis` instead of `this`?** → `flush()` is called with a different `this` context; the saved `lastThis` from the last debounced call is the correct one.
- **What if `flush()` is called when no timer is pending?** → It should do nothing (no-op).
- **What if `cancel()` is called when no timer is pending?** → It should do nothing (no-op).

### Interview Takeaways

- Core mechanism: **clear previous timer, start new timer**.
- Leverage closures to hold the persistent `timer`, `lastThis`, and `lastArgs` across invocations.
- Ensure dynamic `this` binding by returning a standard `function` declaration and invoking `func.call(lastThis, ...lastArgs)`.
- Use an arrow function for the `setTimeout` callback so `this` refers to the wrapper function's `this`.
- Attach `cancel()` and `flush()` as methods on the returned function object.

### Related

- [[Debouncing]]
- [[Closures]]
- [[this]]
- [[call, apply and bind]]
- [[setTimeout]]
- [[Debounce]]