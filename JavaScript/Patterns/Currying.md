---
aliases:
  - Currying
---

## Core Idea

Currying converts a function expecting multiple arguments into a sequence of functions that each take arguments one by one (or in partial batches) until all required arguments are collected, at which point the original function evaluates.

## Recognition

- Problem asks to transform multi-argument function into callable chain: `f(a)(b)(c)`.
- Partial application or incremental evaluation where arguments arrive over time or from different sources.
- "Accumulate arguments until `fn.length` is reached".

## Template

```js
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function (...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}
```

## Variations

1. **Strict Single-Argument Currying**: Each returned function takes strictly one argument `f(a)(b)(c)`.
2. **Loose / Flexible Currying (Lodash style)**: Allows multiple arguments per call `f(a, b)(c)` or empty calls `f()(a)`.
3. **Placeholder Currying (`_`)**: Allows arguments to be provided out of order using placeholder symbols (e.g. `Curry II` / `Curry III`).
4. **Infinite / Dynamic Currying**: Accumulates arguments indefinitely until called with no arguments `sum(1)(2)()` or coerces via `valueOf` / `toString`.

## Complexity

- **Time Complexity:** O(1) per partial call invocation; final evaluation matches the target function `fn`.
- **Space Complexity:** O(N) memory where N is number of arguments accumulated across closures.

## Common Mistakes

- **Shared State Across Chains**: Using a shared external array in closure instead of passing accumulated arguments down the recursive call tree.
- **Losing `this` Context**: Failing to preserve `this` from the caller or invoking with unbound context.
- **Handling Empty Calls `()`**: Not properly ignoring or handling calls with 0 arguments if specified by problem requirements.
- **Handling Default Parameters & Rest Parameters**: `fn.length` only counts parameters before the first default parameter and ignores rest parameters.

## Interview Tips

- Immediately mention `fn.length` for detecting arity.
- Discuss difference between Partial Application vs Currying.
- Explain why immutability of arguments list per invocation prevents crosstalk between branched chains.

## Problems Using This Pattern

- [[Curry]]
- [[Curry II]]
- [[Curry III]]

## Related Patterns

- [[Closure]]
- [[Function Chaining]]

## Related Concepts

- Higher-Order Functions
- `Function.length`
- Arity
- `Function.prototype.apply`
