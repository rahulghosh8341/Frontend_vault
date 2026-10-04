---
title: Test Runner II
aliases:
  - Test Runner II
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
> Extend `createTestRunner()` to support nested `suite(name, fn)` blocks and hierarchical spec names (`' > '`).

## Problem

Follow-up to [[Test Runner]]. Support nested `suite(name, fn)` blocks alongside flat specs:
- `suite(name, fn)`: Registers a suite and executes `fn()` immediately so nested specs/suites attach to it.
- Full spec names in results should be joined with `' > '` (e.g. `'math > add > handles zero'`).
- Result summary shape, `check()`, `.toBe()`, and `.not` remain unchanged.

```js
const { suite, spec, check, run } = createTestRunner();

suite('math', () => {
  suite('add', () => {
    spec('handles zero', () => {
      check(2 + 0).toBe(2);
    });
  });
});

run();
// {
//   total: 1,
//   passed: 1,
//   failed: 0,
//   results: [
//     { name: 'math > add > handles zero', status: 'passed' },
//   ],
// }
```

## Companies

- [[Meta]]

## Pattern

- [[Closure]]
- [[DFS Recursion]]
- [[Method Chaining]]

## 🤔 Thought Process

1. **Tree Representation for Registration**:
   - Flat arrays cannot track nested ancestry. Represent tests as a suite tree.
   - Maintain a `rootSuite` node (`{ type: 'suite', name: '', children: [] }`) and a `suiteStack = [rootSuite]`.
   - The current parent is always `suiteStack[suiteStack.length - 1]`.
2. **Immediate Execution of `suite()`**:
   - `suite(name, fn)` creates a suite node, appends to current parent, pushes onto `suiteStack`, and runs `fn()` immediately inside a `try/finally` block.
   - The `finally` ensures `suiteStack.pop()` runs even if registration throws, preventing stack corruption.
3. **Execution via DFS Traversal**:
   - In `run()`, traverse the tree depth-first.
   - Pass `suitePath` (array of parent suite names) down recursive calls.
   - Root suite has empty name `''` and is excluded from `suitePath`.
   - Construct spec name: `[...suitePath, child.name].join(' > ')`.

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
    children: [],
  };
  const suiteStack = [rootSuite];

  function getCurrentSuite() {
    return suiteStack[suiteStack.length - 1];
  }

  function runSuite(suite, suitePath, results) {
    // Depth-first traversal preserves declaration order for nested suites.
    for (const child of suite.children) {
      if (child.type === 'suite') {
        runSuite(child, [...suitePath, child.name], results);
        continue;
      }

      const name = [...suitePath, child.name].join(' > ');

      try {
        child.fn();
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

      runSuite(rootSuite, [], results);

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

- **Stack-Based Scope Tracking**: Because JavaScript is single-threaded, running `fn()` synchronously while `suite` is on top of `suiteStack` guarantees all enclosed `spec` and child `suite` calls attach directly to that parent.
- **`try/finally` Stack Guarantee**: If a suite declaration throws an error during setup, `finally { suiteStack.pop(); }` ensures subsequent suites don't erroneously nest inside the failed suite.
- **Preserved Declaration Order**: `children` holds an ordered mix of specs and sub-suites in the exact order they were declared.

## 🐞 Bugs / Pitfalls

- **Deferring `suite()` callback until `run()`**: Calling `fn()` inside `run()` instead of immediately breaks registration scoping.
- **Leaking Root Suite Name**: If `rootSuite` name is included, test names start with ` > `. Root suite must have empty name and be filtered/omitted from path.
- **Separate arrays for suites and specs**: Storing `suites` and `specs` in separate arrays loses relative declaration order between sibling specs and sub-suites.

## Production Considerations

- Real frameworks (like Mocha/Jest) allow suite-level options (retries, timeouts) attached to the suite node.
- Depth-first execution mirrors the DOM event bubble/capture and hierarchical AST walkers.

## ⭐ Revision Notes

### Key Facts

- Suite callbacks run immediately at registration time to populate the suite tree.
- A `suiteStack` tracking the active parent enables arbitrarily deep nesting.
- `[...suitePath, child.name].join(' > ')` builds hierarchical breadcrumb names.

### Common Interview Questions

- Why run `suite()` callback immediately but defer `spec()`? `suite()` builds the tree structure; `spec()` contains the actual test code to be measured and reported during `run()`.
- Why is `try / finally` necessary around `fn()` in `suite`? Guarantees the stack pops back to parent even if a syntax or runtime error occurs inside the suite definition.

### Interview Takeaways

- Tree construction via immediate synchronous callbacks and an execution stack is the standard design pattern for declarative DSLs (Jest `describe`, Mocha, Vitest).

### Related

- [[Test Runner]]
- [[Test Runner III]]
- [[Test Runner IV]]
- [[DFS Recursion]]
