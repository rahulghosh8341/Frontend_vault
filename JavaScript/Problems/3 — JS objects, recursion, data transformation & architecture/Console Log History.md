---
title: "Console Log History"
aliases:
  - "consoleLogHistory"
  - "Console Log History"
difficulty: "Medium"
source: GreatFrontEnd
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Console Log History

> [!info] Problem
> Implement a function that intercepts console.log() calls and records the arguments they were invoked with

## Problem

## Console Log History

Sometimes it is useful to inspect what has been logged to the console earlier in a page's lifetime.

Implement `consoleLogHistory()`, a function that installs console log tracking and returns a function for reading the recorded history.

Assume `consoleLogHistory()` is called once before any `console.log()` calls that you want to track. When called, it may replace the default `console` with a custom implementation.

Only `console.log()` and `console.clear()` matter for this question:

- `console.log(...args)` should record one history entry containing the exact argument list for that call.
- `console.clear()` should clear the stored history.
- Wrapped `console.log()` and `console.clear()` should still call through to the original console methods.

## Examples

```javascript
import consoleLogHistory from './console-log-history';

const getHistory = consoleLogHistory();

console.log('hello');
console.log('count', 2);

getHistory();
// [['hello'], ['count', 2]]
```

Calling `console.clear()` should reset the history.

```javascript
const getHistory = consoleLogHistory();

console.log('first');
console.clear();
console.log('second');

getHistory();
// [['second']]
```

## Arguments

`consoleLogHistory()` does not accept any arguments.

## Returns

Returns a function that reads the history of `console.log()` calls recorded since tracking was installed. Each history entry is an array of the arguments passed to one `console.log()` call.

## Notes

- Only `console.log()` and `console.clear()` need to be handled.
- Do not worry about matching browser console formatting.
- Other console methods should continue to work.
- You can assume `consoleLogHistory()` is called once before the logs you want to track.

## Hints

### Hint 1 : How can the wrapper avoid calling itself?

### Hint 2 : What is one history entry?

## Asked at these companies

Meta

## 🤔 Thought Process

- **Immediate Recognition:** Monkey-patching global browser APIs (`console.log`, `console.clear`) with stateful interception.
- **Core Requirements:**
  - Intercept every call to `console.log(...args)` and record `args` into an internal history list.
  - Intercept `console.clear()` to empty the stored history list.
  - Wrapped methods must still forward calls to the original native implementations (`originalLog.apply(console, args)`).
  - Return a function that returns a snapshot/copy of the recorded history.
- **State Management & Closure:**
  - Store history array in the closure of `consoleLogHistory()`.
  - Return getter function returning the history array.

---

## 🧠 Mental Model

Think of a **Spy / Proxy Middleware**:
```
User Call: console.log('hello', 42)
       │
       ▼
Custom Interceptor
       ├──> Push ['hello', 42] to History Log Array
       └──> Forward to Native console.log('hello', 42)
```

---

## 🔑 Key Concepts

- Monkey-patching / Method wrapping
- Function interception & Spying (`originalMethod.apply(target, args)`)
- Closures for encapsulated private state
- Preserving execution context (`this`)

---

## ⚠️ Edge Cases / Traps

- **Preserving Native Context:** When delegating to original methods, always call with `console` as `this` context: `origLog.apply(console, args)`. In some environments, unbound console methods throw `Illegal invocation`.
- **Array Mutation Defense:** Should the history getter return the internal array or a shallow copy? If callers mutate the returned array (`history.pop()`), it shouldn't corrupt future records.
- **`console.clear()` Side Effect:** Must invoke original `console.clear()` *and* reset `history = []`.
- **Multiple Arguments:** `console.log('a', 'b', 'c')` must store the full argument list `['a', 'b', 'c']`.

---

## ⭐ Interview Takeaway

- Save reference to original method before overwriting:
  `const origLog = console.log;`
- Forward calls using spread or `.apply`:
  `origLog.apply(console, args);`
- Always consider restoration/unmounting cleanup in real systems (`uninstall()` method), even if not strictly asked.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why do you need `.apply(console, args)` when calling the original console function?" (Browser console methods often require the `console` object as their context).
- "How does monkey-patching differ from using an ES6 `Proxy`?" (Monkey patching mutates the object directly; `Proxy` creates an intercepting wrapper without mutating the original target).

### Follow-up Questions
- "How would you implement an `uninstall()` function to restore original console behavior?"
- "What if multiple tracking libraries patch `console.log` simultaneously?" (Forms an interceptor chain; order of unwrapping matters).

### Conceptual Questions
- "How do error tracking SDKs like Sentry or Datadog capture breadcrumbs?" (By monkey-patching `console.log`, `fetch`, and `addEventListener`).

---

## 🔄 Variations

- **Spy Function:** Implementing Jest/Sinon `spyOn(object, method)`.
- **Event Emitter with History:** Event hub that maintains an audit log of fired events.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
export default function consoleLogHistory() {
  const history = [];
  const origLog = console.log;
  const origClear = console.clear;

  console.log = function (...args) {
    history.push(args);
    origLog.apply(console, args);
  };

  console.clear = function () {
    history.length = 0;
    origClear.apply(console);
  };

  return function getHistory() {
    return history;
  };
}
```

---

## Official Solution

## Console Log History ( Official solution )

Premium
Languages
The risky part is wrapping a browser global without changing what callers expect from it. The tracker must observe `console.log()` and `console.clear()`, delegate to the original methods, and avoid exposing the mutable history array it owns.

## Solution

The cleanest shape is to install the wrapper when `consoleLogHistory()` is called and keep the history inside that function's closure. The closure owns the history, while the wrapped console methods still delegate to the original console methods.

There are two identity boundaries to preserve. The original methods must be bound before replacement so they still run with the original console receiver, and each history entry should keep the exact argument array for one `console.log()` call.

That leaves these jobs:

- Capture the current `console`, `console.log`, and `console.clear` before overriding anything.
- Store log history in a local array inside `consoleLogHistory()`.
- Replace `globalThis.console` with a wrapper object that keeps every existing console method available, but overrides `log()` and `clear()`.
- In `log(...args)`, push `args` into the history and then call the original `log`.
- In `clear()`, reset the history and then call the original `clear`.
- Return a getter function that reads the current history.

The key trick here is using `Object.create(originalConsole)`. It preserves the rest of the console API without requiring every method to be reimplemented manually.

For example:

| Operation | Internal history | Original console call |
| --- | --- | --- |
| install tracker | `[]` | none |
| `console.log('hello')` | `[['hello']]` | original `log('hello')` |
| `console.log('count', 2)` | `[['hello'], ['count', 2]]` | original `log('count', 2)` |
| `console.clear()` | `[]` | original `clear()` |

```jsx
/**
 * @returns {() => Array<Array<unknown>>}
 */
export default function consoleLogHistory() {
  let history = [];

  const originalConsole = globalThis.console;
  // Native console methods expect the original console object as `this`.
  const originalLog = originalConsole.log.bind(originalConsole);
  const originalClear = originalConsole.clear.bind(originalConsole);

  // Inherit every other console method without reimplementing them.
  const wrappedConsole = Object.create(originalConsole);

  wrappedConsole.log = (...args) => {
    // Store each call as the exact argument list that was logged.
    history.push(args);
    originalLog(...args);
  };

  wrappedConsole.clear = (...args) => {
    // Clearing the console also resets the captured log history.
    history = [];
    originalClear(...args);
  };

  globalThis.console = wrappedConsole;

  return function getHistory() {
    // Return a copy so callers cannot mutate the internal history array.
    return history.slice();
  };
}
```

## Common pitfalls

- Capture and bind the original methods before overriding `globalThis.console`; otherwise the wrapper can recurse into itself.
- Store each log call as its exact argument tuple, including objects, functions, symbols, `null`, and `undefined`.
- Keep other console methods available through the original console prototype chain.
- Return a shallow copy of the outer history array so callers cannot replace the internal list.
- The shallow copy protects the list of entries; it does not deep-clone logged objects, which matches how console logging usually preserves object references.

## Notes

- A closure is a natural place to keep the history for the installed tracker.
- Capturing the original methods first prevents accidental recursive calls into the wrapped methods.
- Returning a shallow copy of the history keeps callers from replacing the outer history array directly.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
After installing the tracker, a caller logs a mutable record:

```javascript
const record = { count: 1 };
console.log(record);
record.count = 2;
const snapshot = getHistory();
snapshot.pop();
```

Does removing the snapshot entry erase internal history? What count does a fresh `getHistory()[0][0].count` read?

Your notes (optional)
