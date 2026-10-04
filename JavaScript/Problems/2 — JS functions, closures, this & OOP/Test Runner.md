---
title: Test Runner
aliases:
  - Test Runner
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Closure]]"
  - "[[Method Chaining]]"
concepts:
  - "[[Closure]]"
  - "[[Object.is]]"
  - "[[Error Handling]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-28
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Implement a minimal test runner (`createTestRunner()`) with `spec()`, chainable `check().toBe()`, and synchronous `run()` reporting.

## Problem

Test runners like Jest and Vitest register tests, execute them, and summarize results. Implement a runner factory `createTestRunner()` returning:
- `spec(name, fn)`: Registers a synchronous test.
- `check(actual).toBe(expected)`: Asserts equality using `Object.is()`. Throws descriptive error on failure.
- `check(actual).not.toBe(expected)`: Flips assertion expectation. Chainable (`.not.not...`).
- `run()`: Executes registered specs in declaration order, catches failures without aborting the suite, and returns a structured summary:
  `{ total, passed, failed, results: [{ name, status, error? }] }`.

```js
const { spec, check, run } = createTestRunner();

spec('adds numbers', () => {
  check(1 + 2).toBe(3);
});

spec('compares strings', () => {
  check('runner').not.toBe('jest');
});

run();
// {
//   total: 2,
//   passed: 2,
//   failed: 0,
//   results: [
//     { name: 'adds numbers', status: 'passed' },
//     { name: 'compares strings', status: 'passed' },
//   ],
// }
```

## Companies

- [[Meta]]

## Pattern

- [[Closure]]
- [[Method Chaining]]

## 🤔 Thought Process

The key is **separating registration from execution**:
1. **Registration**: `spec(name, fn)` only appends to an internal `specs` array in closure. It must not run immediately.
2. **Assertion & Negation**:
   - `check(actual)` returns a matcher.
   - Use getter `get not()` returning `createMatcher(!isNot)` to cleanly flip boolean negation flag without mutating existing matchers.
   - `toBe(expected)` compares with `Object.is(actual, expected)`. Throws formatted error if condition fails.
3. **Execution**:
   - `run()` loops through `specs`.
   - Wrap each `fn()` in `try/catch` so a failing assertion never crashes the test runner loop.
   - Derive summary counts (`total`, `passed`, `failed`) from `results` array.

## 💻 Final Solution

```js
function normalizeError(error) {
  return error instanceof Error ? error.message : String(error);
}

/**
 * @typedef {() => void} SpecFn
 *
 * @typedef {{
 *   toBe: (expected: unknown) => void,
 *   not: Matcher,
 * }} Matcher
 *
 * @typedef {{
 *   spec: (name: string, fn: SpecFn) => void,
 *   check: (actual: unknown) => Matcher,
 *   run: () => RunResult,
 * }} TestRunner
 */

/**
 * @returns {TestRunner}
 */
export default function createTestRunner() {
  const specs = [];

  return {
    spec(name, fn) {
      specs.push({ name, fn });
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
            // Reuse the same matcher shape and only flip the expectation flag.
            return createMatcher(!isNot);
          },
        };
      }

      return createMatcher(false);
    },
    run() {
      const results = [];

      for (const { name, fn } of specs) {
        try {
          fn();
          results.push({ name, status: 'passed' });
        } catch (error) {
          results.push({
            name,
            status: 'failed',
            error: normalizeError(error),
          });
        }
      }

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

- **Closure Isolation**: `specs` lives inside `createTestRunner()`, ensuring every runner instance has isolated state with zero module-level leaks.
- **Recursive Matcher Negation**: `get not()` returns a new matcher with `!isNot`. This allows arbitrary chaining (`.not.not.toBe()`) naturally.
- **`Object.is()` Semantics**: Accurately distinguishes `NaN === NaN` (true) and `-0 === +0` (false), matching Jest/Vitest strict assertions.
- **Error Normalization**: `error instanceof Error ? error.message : String(error)` prevents crashes if non-Error primitives are thrown.

## 🐞 Bugs / Pitfalls

- **Executing specs during `spec()`**: Must defer execution to `run()`.
- **Using `_===` instead of `Object.is()`**: Misses correct `NaN` and `+0`/`-0` comparison.
- **Global `try/catch` around entire loop**: Wrapping the loop in a single `try/catch` stops execution on the first failure. The `try/catch` must bracket individual `fn()` calls.
- **Mutable `.not` property**: Modifying an internal boolean flag in-place on the same matcher breaks if the matcher object reference is reused.

## Production Considerations

- Real test runners run tests concurrently in worker threads or child processes with timeouts.
- Matchers typically provide rich diff output using colored terminal formatters.

## ⭐ Revision Notes

### Key Facts

- Separation of concerns: Registration (`spec`) vs Assertion (`check`) vs Execution (`run`).
- `get not()` creates an elegant immutable negation chain.
- Summary counts should be derived from `results.length` to avoid counter drift bugs.

### Common Interview Questions

- Why use `Object.is()` instead of `_===`? Handles `NaN` equality and `+0`/`-0` distinction correctly.
- How does `.not.not.toBe()` work? Each `.not` getter access returns a fresh matcher with inverted boolean flag (`!isNot`).
- Where is the failure boundary? At the individual spec level within the loop, allowing later tests to run.

### Interview Takeaways

- Factory function returning an object (`createTestRunner()`) encapsulates private arrays via closures.
- First step in building extensible architecture is separating recording from execution.

### Related

- [[Test Runner II]]
- [[Test Runner III]]
- [[Test Runner IV]]
- [[Closure]]
- [[Method Chaining]]
