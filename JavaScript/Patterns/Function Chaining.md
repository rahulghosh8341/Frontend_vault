---
aliases:
  - Function Chaining
---

## Core Idea

Function Chaining (or Curried Accumulator) returns a new function at each invocation stage to accumulate state inside a closure. By returning a brand new function call with updated parameters rather than mutating local variables, each chain remains immutable and branchable.

## Recognition

Use this pattern when:
- Designing API calls that can be chained repeatedly (`fn(a)(b)(c)()`).
- Writing curried math/builder functions where intermediate calls must be reusable independently.
- You need a terminating call (e.g. Empty invocation `()` or special signal argument) to return the accumulated result.

## Template

```js
function chain(accumulatedState) {
  return function (nextArg) {
    // Terminating condition check
    if (nextArg === undefined) {
      return accumulatedState;
    }
    
    // Immutable propagation: return new closure with updated accumulated state
    return chain(accumulatedState + nextArg);
  };
}
```

## Variations

- **Implicit valueOf/toString**: Attaching `.valueOf()` / `[Symbol.toPrimitive]` to the returned function so coercion automatically extracts the total without a terminating `()` call.
- **Variadic Currying**: Accepting multiple arguments per call step (`fn(1, 2)(3)()`).

## Complexity

- **Time Complexity**: $O(1)$ per function invocation step.
- **Space Complexity**: $O(k)$ call stack / closure allocation where $k$ is the chain length.

## Common Mistakes

- Mutating a shared variable across calls, breaking branch independence (`fn(1)` shared by two separate paths).
- Checking `if (!nextArg)` instead of `if (nextArg === undefined)`, which incorrectly terminates on `0` or `""`.
- Trying to re-declare `const` variables inside the same scope.

## Interview Tips

- Emphasize that returning a fresh function invocation (`chain(nextState)`) ensures pure immutability and branch safety.

## Problems Using This Pattern

- [[Sum]]
- [[Curry]]
- [[Curry II]]

## Related Patterns

- [[DFS Recursion]]

## Related Concepts

- [[Closures]]
- [[Currying]]
- [[Partial Application]]
