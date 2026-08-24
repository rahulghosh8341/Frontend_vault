---
title: Implement a function `debounce(func, wait)` that delays invoking `func` until after `wait` milliseconds have elapsed since the last time the debounced function was invoked.
aliases:
  - Debounce
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Airbnb]]"
  - "[[Google]]"
  - "[[Lyft]]"
  - "[[Walmart]]"
  - "[[Meta]]"
  - "[[Yelp]]"
  - "[[TikTok]]"
  - "[[ByteDance]]"
  - "[[Microsoft]]"
  - "[[LinkedIn]]"
  - "[[Uber]]"
  - "[[PayPal]]"
  - "[[Shopify]]"
  - "[[Canva]]"
  - "[[Pinterest]]"
  - "[[Ramp]]"
  - "[[Coinbase]]"
  - "[[Robinhood]]"
  - "[[Discord]]"
  - "[[Palantir]]"
  - "[[Snap]]"
  - "[[Snowflake]]"
  - "[[Adobe]]"
  - "[[Roblox]]"
pattern:
  - "[[Debouncing]]"
concepts:
  - "[[Closures]]"
  - "[[setTimeout]]"
  - "[[this]]"
  - "[[call, apply and bind]]"
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Debouncing controls how often a function can execute over time by executing `func` only after `wait` milliseconds have elapsed since the most recent call.

## Problem

Debouncing controls how often a function can execute over time. When a JavaScript function is debounced with a wait time of `wait` milliseconds, it runs only after `wait` milliseconds have elapsed since the debounced function was last called.

Implement `debounce(func, wait)` so that `func` is called only after `wait` milliseconds have passed since the most recent call. The returned function should not invoke `func` immediately. When the delayed call finally runs, it should use the latest arguments and preserve the `this` value from the most recent call.

```js
let i = 0;
function increment() {
  i++;
}
const debouncedIncrement = debounce(increment, 100);

// t = 0: Call debouncedIncrement().
debouncedIncrement(); // i = 0

// t = 50: i is still 0 because 100ms have not passed.
// Call debouncedIncrement() again.
debouncedIncrement(); // i = 0

// t = 100: i is still 0 because it has only been 50ms since t = 50.

// t = 150: Because 100ms have passed since t = 50,
// increment was invoked and i is now 1.
```

---

## Companies

- [[Airbnb]]
- [[Google]]
- [[Lyft]]
- [[Walmart]]
- [[Meta]]
- [[Yelp]]
- [[TikTok]]
- [[ByteDance]]
- [[Microsoft]]
- [[LinkedIn]]
- [[Uber]]
- [[PayPal]]
- [[Shopify]]
- [[Canva]]
- [[Pinterest]]
- [[Ramp]]
- [[Coinbase]]
- [[Robinhood]]
- [[Discord]]
- [[Palantir]]
- [[Snap]]
- [[Snowflake]]
- [[Adobe]]
- [[Roblox]]

---

## Pattern

- [[Debouncing]]

---

## 🤔 Thought Process

* Debounce means the function executes only after **`wait` ms of inactivity**.
* Keep one `timer` in the closure so every call can access the previous timer.
* On every call:

  1. Cancel the previous timer with `clearTimeout()`.
  2. Create a new timer.
* Capture the latest arguments using `...args`.
* Preserve the latest `this` from the returned function.
* When the timer fires, invoke `func` with the captured `this` and latest arguments using `func.call(this, ...args)`.

---

## 💻 Final Solution

```js
export default function debounce(func, wait) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      func.call(this, ...args);
      timer = undefined;
    }, wait);
  };
}
```

---

## 🤔 Why This Works

Each call resets the countdown:

```text
call
 ↓
clear previous timer
 ↓
start new timer
 ↓
another call?
 ├─ yes → cancel + restart
 └─ no  → wait completes → execute func
```

The arrow function inside `setTimeout` preserves the `this` from the returned function because arrow functions don't create their own `this`.

`...args` preserves the arguments from the most recent call.

After a timer fires, its ID remains stored in `timer`. Calling `clearTimeout()` on an already-fired timer is harmless.

---

## 🐞 Bugs I Made

None in the final implementation.

Important clarification during implementation:

* Initially misunderstood `this` inside the returned debounced function.
* Learned that `this` depends on **how the debounced function is called**, not the function itself.
* Understood that `...args` captures the arguments passed to the debounced function.
* Understood that the `setTimeout` callback needs access to the latest `this` and arguments.

---

## Production Considerations

- **Lodash `_.debounce`**: Production libraries like Lodash support option objects with `leading` (execute at start), `trailing` (execute at end, default), and `maxWait` (guarantee maximum latency).
- **Cancellation & Flushing**: Advanced debounce functions attach `.cancel()` (to clear pending timers) and `.flush()` (to immediately trigger delayed invocations) methods onto the returned wrapper.
- **Memory Cleanup**: Resetting `timer = undefined` inside the timer callback helps with state tracking and garbage collection.
- **`wait = 0` Execution**: Setting `wait` to `0` defers execution to the browser macro-task queue via `setTimeout`, making execution asynchronous rather than synchronous.

---

## ⭐ Revision Notes

### Key Facts

* **Pattern:** [[Debouncing]]
* Debounce = **reset the timer on every call**.
* Only the final call in a burst executes.
* `timer` persists between calls because of the closure.
* `...args` captures the latest arguments.
* `func.call(this, ...args)` preserves the caller's `this`.
* Arrow function inside `setTimeout` preserves the surrounding `this`.
* `clearTimeout()` cancels only a pending timer; clearing an already-fired timer is harmless.
* Time between calls must reach `wait` before execution.
* Common production uses: search inputs, resize handlers, autocomplete, form validation, API request reduction.

### Common Interview Questions

- **How does debounce differ from throttle?** → Debounce resets the countdown timer on every invocation (runs only after a quiet period of inactivity); throttle limits execution frequency to at most once per specified time interval.
- **Why use `func.call(this, ...args)` instead of `func(...args)`?** → To ensure that if the debounced function is called as a method on an object (e.g., `obj.debouncedMethod()`), `this` inside `func` properly points to `obj`.
- **Why must the returned wrapper be a regular function instead of an arrow function?** → An arrow function lexically captures `this` at function creation time, whereas a regular function dynamically binds `this` from the call site.
- **What happens if `wait` is `0`?** → The function is queued as a macro-task via `setTimeout`, firing asynchronously after the current synchronous execution context completes.

### Interview Takeaways

- Core mechanism: **clear previous timer, start new timer**.
- Leverage closures to hold the persistent `timer` identifier across invocations.
- Ensure dynamic `this` binding by returning a standard `function` declaration and invoking `func.call(this, ...args)`.
- Use an arrow function for the `setTimeout` callback so `this` refers to the wrapper function's `this`.

### Related

- [[Debouncing]]
- [[Closures]]
- [[this]]
- [[call, apply and bind]]
- [[setTimeout]]
