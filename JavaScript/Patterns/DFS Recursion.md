---
aliases:
  - DFS Recursion
---

## Core Idea

DFS (Depth-First Search) Recursion traverses deeply into nested structures (like arrays, objects, trees, or graphs) before processing sibling elements. Each step checks whether an item is a primitive leaf or a nested container requiring a recursive traversal.

## Recognition

Apply this pattern when:
- Processing arbitrarily nested arrays or object trees.
- Operating on tree-like hierarchical data (e.g. DOM nodes, nested JSON, file systems).
- The operation must evaluate nested children completely before assembling the outer result.

## Template

```js
function traverseDFS(node) {
  // Base case: leaf node
  if (!isContainer(node)) {
    return processLeaf(node);
  }

  // Recursive case: container node
  const result = [];
  for (const child of node) {
    const subResult = traverseDFS(child);
    result.push(...subResult);
  }
  return result;
}
```

## Variations

- **Array Flattening**: Checking `Array.isArray(item)`.
- **Tree Traversal**: Visiting `node.children` recursively.
- **Deep Cloning**: Recursively copying object properties and arrays.

## Complexity

- **Time Complexity**: $O(n)$ where $n$ is the total number of elements/nodes.
- **Space Complexity**: $O(d)$ where $d$ is the maximum recursion depth (call stack frame size).

## Common Mistakes

- Forgetting base cases, leading to infinite recursion.
- Overwriting intermediate results (`result = traverse(child)`) instead of concatenating/accumulating (`result.push(...traverse(child))`).
- Stack overflow on arbitrarily deep or circular structures.

## Interview Tips

- Always state your base case and recursive case clearly before coding.
- Mention stack overflow limits ($O(d)$ stack depth) and offer an iterative stack/queue approach if the depth is unconstrained.

## Problems Using This Pattern

- [[Flatten]]

## Related Patterns

- [[Array Traversal]]

## Related Concepts

- [[Recursion]]
