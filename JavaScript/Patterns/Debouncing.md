---
aliases:
  - Debouncing
  - Rate Limiting
---

## Core Idea

Debouncing delays the execution of a function until a specified period of inactivity (`wait` milliseconds) has elapsed since the last time the function was called. Each subsequent call resets the timer countdown, ensuring that only the final invocation in a burst of continuous calls actually fires.

## Recognition

Use the Debouncing pattern when:
- Events trigger rapidly and repeatedly in bursts (e.g., keystrokes in a search input, window resizing, scroll events, button clicks).
- You only care about the **final result** after the user stops performing the action.
- Executing the callback on every event would cause UI stutter or excessive API network requests.

## Template

```js
function debounce(func, wait) {
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

## Variations

- **Trailing Debounce (Default)**: Executes `func` after `wait` milliseconds of inactivity.
- **Leading Debounce**: Executes `func` immediately on the first call, then suppresses subsequent calls until `wait` milliseconds of inactivity pass.
- **Cancelable / Flushable Debounce**: Exposes `.cancel()` to clear pending timers and `.flush()` to trigger execution immediately.

## Complexity

- **Time Complexity:** $O(1)$ per call to reset/start the timer; the callback execution time depends on `func`.
- **Space Complexity:** $O(1)$ memory stored in the closure to hold `timer`.

## Common Mistakes

- **Using an arrow function for the wrapper**: Arrow functions do not have their own `this` binding, causing `this` to be lexically captured from the outer scope rather than dynamically bound from the caller.
- **Forgetting `func.call(this, ...args)`**: Losing the caller's `this` context or latest arguments when executing inside `setTimeout`.
- **Omitting `clearTimeout(timer)`**: Not clearing the previous timer causes every call to eventually execute instead of debouncing.

## Interview Tips

- Clearly articulate the difference between **Debounce** (wait for pause) and **Throttle** (at most once every X ms).
- Point out that `clearTimeout(undefined)` or `clearTimeout(null)` is a safe no-op in standard JS environments.
- Explain how closures allow state (`timer`) to persist across invocations without polluting global scope.

## Problems Using This Pattern

- [[Debounce]]

## Related Patterns

- [[Throttling]]

## Related Concepts

- [[Closures]]
- [[this]]
- [[call, apply and bind]]
- [[setTimeout]]
