---
aliases:
  - Recursion
---

## Core Idea

A recursive function solves a problem by breaking it down into smaller, similar subproblems. It calls itself with modified inputs until it reaches a base case that can be solved directly. The solutions to the subproblems are then combined to solve the original problem.

## Recognition

Use recursion when the data structure or problem has a self-similar nature:
- Nested data (trees, deeply nested objects/arrays).
- Problems that can be divided into smaller versions of themselves (factorial, Fibonacci, traversal).
- When the depth is unknown or potentially large.

## Template

```js
function recurse(input) {
  // Base case: simplest form that can be solved directly
  if (input is simple) {
    return solution;
  }

  // Recursive case: break into smaller pieces
  const smallerPieces = decompose(input);
  
  // Solve each piece recursively
  const solutions = smallerPieces.map(piece => recurse(piece));
  
  // Combine solutions to form the final answer
  return combine(solutions);
}
```

## Variations

- **Tree/Object traversal** — visit each node/property, recurse on children.
- **Accumulator pattern** — pass partial result down through recursive calls.
- **Helper recursion** — outer function sets up state, inner function does the recursion.
- **Tail recursion** — recursive call is the last operation (can be optimized by engines).

## Complexity

Time: Depends heavily on the problem structure. Often expressed as T(n) = a*T(n/b) + f(n) where a is number of subproblems, b is size reduction factor.
Space: O(d) where d is the maximum recursion depth due to call stack.

## Common Mistakes

- Forgetting or incorrectly defining base cases → infinite recursion.
- Not making progress toward the base case in recursive calls.
- Returning the wrong thing from recursive calls (e.g., returning `undefined` instead of processed result).
- Mutating shared state between recursive calls.
- Stack overflow on deep structures → consider iterative approach or trampolining.

## Interview Tips

- Always identify and state the base case first.
- Trace through a small example to verify the recursion logic.
- Be explicit about what each recursive call returns and how the parent uses it.
- For tree problems, distinguish between visiting a node vs. processing its children.
- Mention stack overflow risk for very deep structures.

## Problems Using This Pattern

- [[Compact II]]
- [[Deep Clone]]
- [[Deep Equal]]
- [[JSON.stringify]]

## Related Patterns

- [[DFS Recursion]]
- [[Tree Traversal]]
- [[Deep Traversal]]

## Related Concepts

- [[Call Stack]]
- [[Base Case]]
- [[Tail Call Optimization]]