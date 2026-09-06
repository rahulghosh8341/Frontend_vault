---
aliases:
  - Closure
---

## Core Idea

A closure is a function bundled together with references to its surrounding lexical scope. Variables declared in the outer function remain accessible to the inner function even after the outer function has returned. This makes closures the natural tool for building **stateful wrappers** (memoization, throttling, event handlers, factories, partial application).

## Recognition

Use this pattern when a returned function needs to:
- Remember values across multiple invocations (counters, caches, last result).
- Hide internal state from the outside world.
- Capture arguments or configuration at creation time and reuse them on every call.
- Preserve a private scope for each instance (factory pattern).

## Template

```js
function makeWrapper(input, options) {
  // private state, persists across calls
  let count = 0;
  let lastResult;

  return function(...args) {
    // use input, options, count, lastResult, this, args
    count++;
    lastResult = doWork(input, ...args);
    return lastResult;
  };
}
```

## Variations

- **Counter + cap (`Limit`)** — track invocations, block past `n`.
- **Cache by input (`Memoize`)** — store `result = fn(...args)` keyed by arguments.
- **Time window (`Debounce`, `Throttle`)** — track timestamps / timers in closure.
- **Once-only (`Once`)** — run on first call only, reuse last result.
- **Currying / partial application** — capture args progressively.

## Complexity

Time: **O(1)** per invocation (excluding the wrapped work itself).
Space: **O(1)** per wrapper instance, plus whatever the wrapped function allocates.

## Common Mistakes

- Forgetting to forward `this` and `args` via `apply(this, args)` / `call(this, ...args)`.
- Declaring the cache/state in the outer scope but returning an arrow function that captures the wrong `this`.
- Re-creating closures inside loops instead of once per instance.
- Mutating outer-scope variables that other instances also share.

## Interview Tips

- Lead with the mental model: "a function + the variables it was born with."
- Distinguish closure from class: both encapsulate private state, but closures do it without `this` or `new`.
- When asked about `Limit` / `Once`, the answer is almost always "closure over a counter."
- Mention memory: closures keep referenced outer variables alive until the closure is released.

## Problems Using This Pattern

- [[Limit]]
- [[Debounce]]
- [[Debounce II]]
- [[Function Chaining]]

## Related Patterns

- [[Function Chaining]]
- [[Debouncing]]
- [[Higher Order Mapping]]

## Related Concepts

- [[Lexical Scope]]
- [[Higher-Order Functions]]
- [[Currying]]
- [[Memoization]]
