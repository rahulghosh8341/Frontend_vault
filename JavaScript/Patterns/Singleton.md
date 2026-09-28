---
aliases:
  - Singleton
---

## Core Idea

The Singleton pattern restricts the instantiation of a class or resource to a single instance and provides a single global point of access to it across the entire application lifecycle.

## Recognition

- Need global shared state or single coordination point (e.g., config, logger, global cache/store, thread pool/connection pool).
- Requirement says: "All calls must return the exact same instance / reference (`a === b`)".
- Preventing duplicate initialization of heavy or stateful objects.

## Template

### Module-level Closure / Static Access

```js
let instance;

const Singleton = {
  getInstance() {
    if (!instance) {
      instance = createInstance();
    }
    return instance;
  },
};

export default Singleton;
```

### Class with Static Instance / Private Constructor

```js
class Singleton {
  static #instance;

  constructor() {
    if (Singleton.#instance) {
      return Singleton.#instance;
    }
    Singleton.#instance = this;
  }

  static getInstance() {
    if (!Singleton.#instance) {
      Singleton.#instance = new Singleton();
    }
    return Singleton.#instance;
  }
}
```

## Variations

1. **Lazy Initialization**: Instance created on first `getInstance()` invocation.
2. **Eager Initialization**: Instance created immediately at module load time (built-in ES module singleton behavior).
3. **Target Resource as Singleton**: The singleton access object returns an instance of an existing type (like `Map` or `EventEmitter`), rather than an instance of the accessor itself.

## Complexity

- **Time Complexity:** O(1) for instance check and retrieval.
- **Space Complexity:** O(1) metadata overhead beyond the stored instance.

## Common Mistakes

- **Returning wrapper object instead of target instance**: If requirements specify `getInstance() instanceof Map`, wrapping `Map` inside a wrapper class breaks `instanceof` checks.
- **Leaking internal mutability or state reset**: Forgetting to provide reset mechanics if needed for testing environments.
- **Overusing Singletons**: Creates tight coupling and makes unit testing difficult due to hidden shared state across test runs.

## Interview Tips

- In modern JavaScript, **ES Modules are singletons by default** because module evaluation is cached per URL/specifier.
- Point out lazy initialization vs module-level export of a single pre-created instance: `export default new Map()`.
- Discuss testing implications: singletons require explicit reset mechanisms so tests don't leak state into one another.

## Problems Using This Pattern

- [[Singleton]]

## Related Patterns

- [[Closure]]

## Related Concepts

- ES Modules caching
- Lazy initialization
- Static private class fields (`#instance`)
