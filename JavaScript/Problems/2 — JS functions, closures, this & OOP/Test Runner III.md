---
title: Test Runner III
aliases:
  - Test Runner III
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Closure]]"
  - "[[DFS Recursion]]"
  - "[[Method Chaining]]"
concepts:
  - "[[Closure]]"
  - "[[Tree Traversal]]"
  - "[[DFS Recursion]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-28
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Extend `createTestRunner()` to support inherited `setupEach(fn)` and `cleanupEach(fn)` lifecycle hooks across nested suites.

## Problem

Follow-up to [[Test Runner II]]. Add per-spec hooks:
- `setupEach(fn)`: Registers a setup hook for the current suite. Inherited setups run **outer-to-inner** before each spec.
- `cleanupEach(fn)`: Registers a cleanup hook for the current suite. Inherited cleanups run **inner-to-outer** after each spec.
- Hooks are scoped to their suite and inherited by all descendant specs, rerunning for every matching spec.

```js
const log = [];
const { suite, spec, check, run, setupEach, cleanupEach } = createTestRunner();

suite('math', () => {
  setupEach(() => log.push('setup math'));
  cleanupEach(() => log.push('cleanup math'));

  suite('add', () => {
    setupEach(() => log.push('setup add'));
    cleanupEach(() => log.push('cleanup add'));

    spec('works', () => {
      check('spec').not.toBe('other');
      log.push('spec');
    });
  });
});

run();
log;
// [
//   'setup math',
//   'setup add',
//   'spec',
//   'cleanup add',
//   'cleanup math',
// ]
```

## Companies

- [[Meta]]

## Pattern

- [[Closure]]
- [[DFS Recursion]]
- [[Method Chaining]]

## 🤔 Thought Process

1. **Suite Node Storage**:
   - Each suite node gets two arrays: `setupHooks: []` and `cleanupHooks: []`.
   - `setupEach(fn)` and `cleanupEach(fn)` push `fn` to `getCurrentSuite()`.
2. **Hook Inheritance During Traversal**:
   - As we traverse down into a child suite:
     - `setupHooks`: Outer setups run first, so append child setups to inherited setups:
       `[...inheritedSetups, ...suite.setupHooks]`.
     - `cleanupHooks`: Inner cleanups run first, so prepend child cleanups before inherited cleanups:
       `[...suite.cleanupHooks, ...inheritedCleanups]`.
3. **Per-Spec Execution Pipeline**:
   - For every spec node:
     1. Run all accumulated `setupHooks` in order.
     2. Run `child.fn()`.
     3. Run all accumulated `cleanupHooks` in order.
     4. Record pass or catch failure.

## 💻 Final Solution

```js
function normalizeError(error) {
  return error instanceof Error ? error.message : String(error);
}

/**
 * @returns {TestRunner}
 */
export default function createTestRunner() {
  const rootSuite = {
    type: 'suite',
    name: '',
    setupHooks: [],
    cleanupHooks: [],
    children: [],
  };
  const suiteStack = [rootSuite];

  function getCurrentSuite() {
    return suiteStack[suiteStack.length - 1];
  }

  function runSuite(
    suite,
    suitePath,
    inheritedSetups,
    inheritedCleanups,
    results,
  ) {
    // Setups run outer-to-inner, while cleanups need the reverse scope order.
    const setupHooks = [...inheritedSetups, ...suite.setupHooks];
    const cleanupHooks = [...suite.cleanupHooks, ...inheritedCleanups];

    for (const child of suite.children) {
      if (child.type === 'suite') {
        runSuite(
          child,
          [...suitePath, child.name],
          setupHooks,
          cleanupHooks,
          results,
        );
        continue;
      }

      const name = [...suitePath, child.name].join(' > ');

      try {
        for (const hook of setupHooks) {
          hook();
        }

        child.fn();

        for (const hook of cleanupHooks) {
          hook();
        }

        results.push({ name, status: 'passed' });
      } catch (error) {
        results.push({
          name,
          status: 'failed',
          error: normalizeError(error),
        });
      }
    }
  }

  return {
    suite(name, fn) {
      const suite = {
        type: 'suite',
        name,
        setupHooks: [],
        cleanupHooks: [],
        children: [],
      };

      getCurrentSuite().children.push(suite);
      suiteStack.push(suite);

      try {
        fn();
      } finally {
        suiteStack.pop();
      }
    },
    setupEach(fn) {
      getCurrentSuite().setupHooks.push(fn);
    },
    cleanupEach(fn) {
      getCurrentSuite().cleanupHooks.push(fn);
    },
    spec(name, fn) {
      getCurrentSuite().children.push({
        type: 'spec',
        name,
        fn,
      });
    },
    check(actual) {
      function createMatcher(isNot) {
        return {
          toBe(expected) {
            const matches = Object.is(actual, expected);

            if (isNot ? matches : !matches) {
              throw new Error(
                `Expected ${String(actual)} ${isNot ? 'not ' : ''}to be ${String(expected)}`,
              );
            }
          },
          get not() {
            return createMatcher(!isNot);
          },
        };
      }

      return createMatcher(false);
    },
    run() {
      const results = [];

      runSuite(rootSuite, [], [], [], results);

      const failed = results.filter(
        (result) => result.status === 'failed',
      ).length;

      return {
        total: results.length,
        passed: results.length - failed,
        failed,
        results,
      };
    },
  };
}
```

## 🤔 Why This Works

- **Proper Symmetrical Ordering (Onion Architecture)**:
  - Setup: Parent -> Child (opening resources).
  - Cleanup: Child -> Parent (closing resources in reverse order, avoiding dangling dependent state).
- **Scope Isolation (No Sibling Leakage)**:
  - Hooks are passed down through parameters in recursive calls (`runSuite`).
  - When returning from a child suite, the parent's hook arrays are completely unmodified.
- **Rerun Per Spec**:
  - Hooks run directly inside the spec execution loop, guaranteeing clean state before and after every test.

## 🐞 Bugs / Pitfalls

- **Incorrect Cleanup Ordering**: Appending cleanups in FIFO order instead of LIFO order (cleanups must run innermost suite first).
- **Global Hook Storage**: Storing hooks on the runner instance instead of suite nodes causes child hooks to leak into sibling or parent specs.
- **Running Hooks Once Per Suite**: Confusing `beforeEach` (`setupEach`) with `beforeAll`. `setupEach` must run for every spec.

## Production Considerations

- In full-scale runners, cleanup hooks must execute inside a `finally` block even if the test fails, so teardown always runs.
- Real runners also implement `beforeAll` / `afterAll` which execute once per suite block.

## ⭐ Revision Notes

### Key Facts

- `setupEach` order: Outer -> Inner (`[...inherited, ...local]`).
- `cleanupEach` order: Inner -> Outer (`[...local, ...inherited]`).
- Scope boundary: Descendants inherit hooks; siblings never share hooks.

### Common Interview Questions

- Why must cleanup hooks run in reverse order of setup hooks? If setup A opens a database connection and setup B starts a transaction on it, cleanup B must rollback the transaction before cleanup A closes the connection.
- How do you prevent hooks leaking between sibling suites? Pass accumulated hook arrays down the traversal stack rather than mutating a shared global hook registry.

### Interview Takeaways

- Hierarchical context propagation is cleanly implemented by combining parent state with local state during recursive DFS descent.

### Related

- [[Test Runner]]
- [[Test Runner II]]
- [[Test Runner IV]]
- [[DFS Recursion]]
