---
title: "Assert Lite"
aliases:
  - "assertLite"
  - "Assert Lite"
difficulty: "Medium"
source: GreatFrontEnd
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Assert Lite

> [!info] Problem
> Implement a tiny assertion library with truthy and strict equality checks

## Problem

## Assert Lite

Node.js ships with an [`assert`](https://nodejs.org/api/assert.html) module for writing test expectations.

In this question, implement a tiny assertion library inspired by `node:assert/strict`.

This first question is intentionally limited:

- Export `AssertionError`, `ok(value, message?)`, and `strictEqual(actual, expected, message?)`.
- Passing assertions should return `undefined`.
- Failing assertions should throw `AssertionError`.
- `strictEqual()` should use `Object.is()` semantics.

## Examples

```javascript
ok('hello');
strictEqual(3, 3);
// No error.
```

```javascript
strictEqual(3, '3');
// Throws AssertionError {
//   name: 'AssertionError',
//   message: 'Expected values to be strictly equal',
//   actual: 3,
//   expected: '3',
//   operator: 'strictEqual',
// }
```

Custom messages should override the default failure message.

```javascript
ok('', 'Name is required');
// Throws AssertionError {
//   message: 'Name is required',
//   operator: 'ok',
// }
```

## API

### class AssertionError extends Error

Represents a failed assertion.

Instances must have:

- `name = 'AssertionError'`
- `message` from the provided custom message or the default failure message
- `operator` describing which assertion failed

For assertions that compare an actual value with an expectation, also include:

- `actual`
- `expected`

An `ok()` failure records the received value as `actual` and `true` as `expected`. A `strictEqual()` failure records its two input values.

### ok(value, message?)

Throws unless `value` is truthy.

Default message: `Expected value to be truthy`

### strictEqual(actual, expected, message?)

Throws unless `Object.is(actual, expected)` is `true`.

Default message: `Expected values to be strictly equal`

## Returns

All passing assertions return `undefined`.

Failing assertions throw `AssertionError`.

## Notes

- Keep the API intentionally small and interview-scoped.
- You do not need diff formatting, loose equality assertions, negated assertions, or stack trace customization.
- Tests in this question only use ordinary interview values; you do not need to optimize for every edge case of the real Node.js module.

## Resources

- [Node.js `assert`](https://nodejs.org/api/assert.html)

## Hints

### Hint : What should every failure have in common?

## 🤔 Thought Process

- **Immediate Recognition:** Building an assertion test library mimicking `node:assert/strict`.
- **Core Requirements:**
  - Export `AssertionError` class inheriting from `Error`.
  - Export `ok(value, message?)`: Throws `AssertionError` if `value` is falsy.
  - Export `strictEqual(actual, expected, message?)`: Throws `AssertionError` if `!Object.is(actual, expected)`.
  - On pass: Return `undefined`.
  - On fail: Throw `AssertionError` with appropriate message and error metadata.
- **Custom Error Construction:**
  - `class AssertionError extends Error` must set `this.name = 'AssertionError'`.
  - Store metadata like `actual`, `expected`, `operator` on the error instance.

---

## 🧠 Mental Model

Think of **Contract Invariants & Defensive Programming**:
- Assertions assert that a condition *must* be true at runtime.
- If it fails, execution halts immediately with a rich diagnostic error detailing what was expected vs what was actually received.

---

## 🔑 Key Concepts

- Custom `Error` subclassing (`class AssertionError extends Error`)
- `Object.is()` strict equality vs `===` (`Object.is(NaN, NaN)` is `true`, `Object.is(+0, -0)` is `false`)
- Truthy vs Falsy evaluation
- Diagnostic metadata on error instances

---

## ⚠️ Edge Cases / Traps

- **`NaN === NaN` vs `Object.is(NaN, NaN)`:**
  In standard JavaScript, `NaN === NaN` is `false`. But in strict assertions, `assert.strictEqual(NaN, NaN)` must PASS. `Object.is()` correctly handles this.
- **`+0` vs `-0`:**
  `+0 === -0` is `true`, but `Object.is(+0, -0)` is `false`. Strict assertion standards demand `Object.is`.
- **Default Error Messages:** If no custom `message` string is passed, generate an informative default (e.g. `Expected ${actual} to equal ${expected}`).

---

## ⭐ Interview Takeaway

- Always use `Object.is(actual, expected)` when building strict equality assertions in JavaScript.
- When subclassing `Error`:
  ```javascript
  class AssertionError extends Error {
    constructor(options = {}) {
      super(options.message);
      this.name = 'AssertionError';
    }
  }
  ```
- Subclassing errors and writing test assertions is standard in SDK, framework, and tooling interviews.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why use `Object.is()` instead of `===` for `strictEqual`?" (`Object.is` correctly identifies `NaN` equality and `-0` vs `+0` difference).
- "How do you properly subclass `Error` in modern ES6?" (Extend `Error`, call `super(message)`, set `this.name`).

### Follow-up Questions
- "How would you implement `deepStrictEqual(actual, expected)`?" (Recursive structural equality check for objects, arrays, Sets, Maps, and Dates).
- "How does Jest's `expect().toBe()` differ from `assert.strictEqual()`?"

### Conceptual Questions
- "Why is `Error.captureStackTrace` used in Node.js assertion libraries?" (To omit internal assertion library frames from user stack traces).

---

## 🔄 Variations

- **Test Runner (I, II, III, IV):** Building the test execution harness (`describe`, `it`, `expect`) that uses these assertions.
- **Deep Equal:** Implementing structural object comparison.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
export class AssertionError extends Error {
  constructor(options = {}) {
    super(options.message || 'Assertion failed');
    this.name = 'AssertionError';
    this.actual = options.actual;
    this.expected = options.expected;
    this.operator = options.operator;
  }
}

export function ok(value, message) {
  if (!value) {
    throw new AssertionError({
      message: message || 'Expected value to be truthy',
      actual: value,
      expected: true,
      operator: 'ok',
    });
  }
}

export function strictEqual(actual, expected, message) {
  if (!Object.is(actual, expected)) {
    throw new AssertionError({
      message: message || `Expected ${actual} to strictly equal ${expected}`,
      actual,
      expected,
      operator: 'strictEqual',
    });
  }
}
```

---

## Official Solution

## Assert Lite ( Official solution )

Premium
Languages

## Solution

This is a tiny function-style assertion API. Each exported helper checks one condition, returns `undefined` when that condition passes, and throws the same `AssertionError` type when it fails. There is no matcher object, chaining API, or negated form such as `.not` in this interview-scoped version.

Once the thrown error shape is consistent, the helpers stay small. Handle two cases:

- Model every assertion failure with `AssertionError`.
- Keep each assertion helper focused on deciding pass vs fail.

That gives the library a simple rule: successful assertions have no result to inspect, and failed assertions are represented by one structured error type. Callers can catch `AssertionError` and read the same metadata fields regardless of which helper failed.

`AssertionError` extends `Error`, uses either the custom message or a default message, restores the prototype chain with `Object.setPrototypeOf()`, and stores metadata on the instance:

- `name: 'AssertionError'`
- `message`
- `operator`
- `actual` and `expected` when the assertion has meaningful comparison values

The assertion helpers then become direct checks:

- `ok()` only needs to reject falsy values.
- `strictEqual()` should compare with `Object.is()` so cases like `NaN` and `-0` follow modern strict-equality semantics.
- Custom messages replace the default failure message.

That is enough to model the important parts of strict assertions without recreating the full Node.js API surface.

The pass/fail table is intentionally small:

| Helper | Pass condition | Failure metadata |
| --- | --- | --- |
| `ok(value)` | `value` is truthy | `actual: value`, `expected: true`, `operator: 'ok'` |
| `strictEqual(actual, expected)` | `Object.is(actual, expected)` | both compared values and `operator: 'strictEqual'` |

```jsx
/**
 * @typedef {{
 *   message?: string,
 *   actual?: unknown,
 *   expected?: unknown,
 *   operator?: string,
 * }} AssertionErrorOptions
 */

export class AssertionError extends Error {
  /**
   * @param {AssertionErrorOptions} [options={}]
   */
  constructor(options = {}) {
    super(options.message ?? 'Assertion failed');
    Object.setPrototypeOf(this, new.target.prototype);

    this.name = 'AssertionError';
    this.actual = options.actual;
    this.expected = options.expected;
    this.operator = options.operator;
  }
}

/**
 * @param {unknown} value
 * @param {string} [message]
 * @returns {void}
 */
export function ok(value, message) {
  if (value) {
    return;
  }

  throw new AssertionError({
    message: message ?? 'Expected value to be truthy',
    actual: value,
    expected: true,
    operator: 'ok',
  });
}

/**
 * @param {unknown} actual
 * @param {unknown} expected
 * @param {string} [message]
 * @returns {void}
 */
export function strictEqual(actual, expected, message) {
  if (Object.is(actual, expected)) {
    return;
  }

  throw new AssertionError({
    message: message ?? 'Expected values to be strictly equal',
    actual,
    expected,
    operator: 'strictEqual',
  });
}
```

## Common pitfalls

- **Throwing a plain `Error`:** The tests need to distinguish assertion failures from other errors. Throwing a plain `Error` loses the `AssertionError` class, `operator`, and comparison metadata.
- **Using `===` for `strictEqual()`:** `===` is close, but it does not match the requested `Object.is()` semantics. `Object.is(NaN, NaN)` is `true`, and `Object.is(0, -0)` is `false`.
- **Forgetting the success return value:** Passing assertions should return `undefined`. They should not return booleans, the checked value, or the assertion object.
- **Building a larger matcher API:** This question asks for exported assertion functions only. Do not add matcher chaining, negated assertions, loose equality helpers, diff formatting, or stack trace customization.

## Notes

- `Object.is()` is a good fit for strict equality because it mirrors modern `assert.strictEqual()` semantics closely enough for this interview-scoped version.
- Returning `undefined` on success keeps the assertion helpers simple and easy to compose in tests.
- A dedicated `AssertionError` makes it straightforward for callers to distinguish assertion failures from other thrown errors.
- `ok()` records `actual` as the falsy input and `expected` as `true`. `strictEqual()` records both values being compared.
- The helper intentionally avoids formatting diffs or stack traces. Those features are useful in production test frameworks, but they are separate from the core pass/fail rule here.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
Which calls complete without throwing? Select all that apply.
