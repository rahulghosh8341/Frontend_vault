---
title: Test Runner IV
aliases:
  - Test Runner IV
difficulty: Hard
time: 25 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Closure]]"
  - "[[DFS Recursion]]"
  - "[[Method Chaining]]"
concepts:
  - "[[Async/Await]]"
  - "[[Promises]]"
  - "[[Tree Traversal]]"
  - "[[DFS Recursion]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-28
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 25 min
> Extend `createTestRunner()` to support asynchronous specs and asynchronous lifecycle hooks (`async/await`) executed sequentially.

## Problem

Follow-up to [[Test Runner III]]. Support asynchronous specs and hooks:
- `spec(name, fn)`: `fn` can be synchronous or async (returns a Promise).
- `setupEach(fn)` and `cleanupEach(fn)`: Hooks can be synchronous or async.
- `run()`: Returns `Promise<RunResult>`.
- Specs and hooks must be awaited sequentially in declaration order. Setup/cleanup ordering rules remain unchanged.

```js
const { spec, check, run } = createTestRunner();

spec('loads data', async () => {
  const value = await Promise.resolve(42);
  check(value).not.toBe(0);
});

await run();
// {
//   total: 1,
//   passed: 1,
//   failed: 0,
//   results: [{ name: 'loads data', status: 'passed' }],
// }
```

## Companies

- [[Meta]]

## Pattern

- [[Closure]]
- [[DFS Recursion]]
- [[Method Chaining]]

## 🤔 Thought Process

1. **Synchronous Registration Stays Synchronous**:
   - `suite()`, `spec()`, `setupEach()`, and `cleanupEach()` do not change. The tree structure is assembled synchronously.
2. **Asynchronous Execution Phase**:
   - `run()` and the recursive traversal function `runSuite()` become `async`.
   - Use `await` sequentially for every step:
     - `await runSuite(...)` for sub-suites.
     - For each spec:
       1. `for (const hook of setupHooks) await hook();`
       2. `await child.fn();`
       3. `for (const hook of cleanupHooks) await hook();`
3. **Promise & Rejection Handling**:
   - `await` naturally handles both synchronous returns and Promises (unwrapping resolved values or throwing on rejection).
   - Wrap the awaited pipeline in `try/catch` to catch rejected promises and normalize errors: `error instanceof Error ? error.message : String(error)`.
   - `run()` resolves to the summary even when tests fail.

## 💻 Final Solution

```js
/**
 * @typedef {() => void | Promise<void>} HookFn
 * @typedef {() => void | Promise<void>} SpecFn
 * @typedef {{ toBe(expected: unknown): void, readonly not: Matcher }} Matcher
 * @typedef {{
 *   total: number,
 *   passed: number,
 *   failed: number,
 *   results: Array<
 *     | { name: string, status: 'passed' }
 *     | { name: string, status: 'failed', error: string }
 *   >,
 * }} RunResult
 * @typedef {{
 *   suite(name: string, fn: () => void): void,
 *   setupEach(fn: HookFn): void,
 *   cleanupEach(fn: HookFn): void,
 *   spec(name: string, fn: SpecFn): void,
 *   check(actual: unknown): Matcher,
 *   run(): Promise<RunResult>,
 * }} TestRunner
 */
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

  async function runSuite(
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
        await runSuite(
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
          await hook();
        }

        await child.fn();

        for (const hook of cleanupHooks) {
          await hook();
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
    async run() {
      const results = [];

      await runSuite(rootSuite, [], [], [], results);

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

- **Universal `await`**: In JavaScript, `await nonPromiseValue` resolves synchronously in a microtask. Writing `await hook()` transparently handles both sync functions (`() => {}`) and async functions (`async () => {}`) without type branching.
- **Strict Sequential Order**: Using standard `for ... of` loops with `await` guarantees that asynchronous tests do not run concurrently, preserving deterministic test execution order and shared state safety.
- **Unhandled Rejection Capture**: Any rejected Promise inside `await hook()` or `await child.fn()` is caught by the surrounding `try/catch` and recorded as a failed test rather than crashing the Node.js/browser process.

## 🐞 Bugs / Pitfalls

- **Using `Promise.all`**: Running specs concurrently via `Promise.all(specs.map(...))` breaks deterministic test order and causes race conditions when tests share state.
- **Forgetting to `await runSuite`**: Recursive suite calls must be awaited; otherwise, `run()` resolves before nested suites finish running.
- **Letting `run()` reject**: If an error escapes `run()`, the caller receives an unhandled promise rejection instead of the expected summary object.
- **Making `suite()` async**: `suite()` registration must remain synchronous to avoid race conditions on `suiteStack`.

## Production Considerations

- Production test runners implement timeouts (e.g. 5000ms default) using `Promise.race` against a timer.
- Teardown (`cleanupEach`) should run even if `setupEach` or `spec` fails (implemented with `try ... finally`).

## ⭐ Revision Notes

### Key Facts

- Registration remains synchronous; only execution (`run` and `runSuite`) becomes `async`.
- `await` works uniformly on both Promises and synchronous primitives/functions.
- `for ... of` loops preserve serial execution with async/await, unlike `.forEach()` or `Promise.all()`.

### Common Interview Questions

- Why not use `Array.prototype.forEach` with `async`? `forEach` does not await promises; it fires all callbacks concurrently and returns immediately.
- How do you guarantee cleanup hooks run if the spec throws? Wrap spec execution in `try ... finally` or catch errors, run cleanups, then re-throw or record failure.

### Interview Takeaways

- Evolving an architecture from synchronous to asynchronous: Keep the domain model and tree structure synchronous; change only the execution visitor layer into async sequential steps.

### Related

- [[Test Runner]]
- [[Test Runner II]]
- [[Test Runner III]]
- [[DFS Recursion]]
- [[Async/Await]]
