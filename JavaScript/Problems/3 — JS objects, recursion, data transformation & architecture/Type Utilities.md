---
title: Type Utilities
aliases:
  - Type Utilities
difficulty: Easy
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/type-utilities"
pattern:
  - "[[Type Checking]]"
concepts:
  - "[[Type Checking]]"
  - "[[typeof]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Type Utilities

> [!info] Problem
> Implement utilities to determine primitive variable types in JavaScript

## Problem

## Type Utilities

Yangshun Tay
Ex-Meta Staff Engineer
JavaScript is a dynamically typed language, which means the types of variables can change at runtime. Many interview questions involve recursively traversing values that contain different value types, and each type may require different handling (e.g. different code is needed to iterate over an array vs. an object). Understanding JavaScript types is crucial to solving questions like [Deep Clone](/questions/javascript/deep-clone) and [Deep Equal](/questions/javascript/deep-equal).

Implement the following utility functions to determine the types of primitive values.

- `isBoolean(value)`: Return `true` if `value` is a boolean, `false` otherwise.
- `isNumber(value)`: Return `true` if `value` is a number, `false` otherwise. Note that `NaN` is considered a number.
- `isNull(value)`: Return `true` if `value` is `null`, `false` otherwise.
- `isString(value)`: Return `true` if `value` is a string, `false` otherwise.
- `isSymbol(value)`: Return `true` if `value` is a symbol primitive, `false` otherwise.
- `isUndefined(value)`: Return `true` if `value` is `undefined`, `false` otherwise.

## Hints

### Hint : Which checks answer the exact type question?

## 🤔 Thought Process

- **Immediate Recognition:** Standard primitive type predicates foundational for deep data transformations (`deepClone`, `deepEqual`).
- **Core Problem:** Determining whether a value is strictly of a given JavaScript primitive type without type coercion, boxing, or false positives.
- **Rules per Type:**
  - `isBoolean`: Use `val === true || val === false` (rejects truthy/falsy non-booleans like `1`, `""`).
  - `isNumber`: Use `typeof val === 'number'` (includes `NaN` and `Infinity` per spec).
  - `isNull`: Use `val === null` (critical trap: `typeof null === 'object'`).
  - `isString`: Use `typeof val === 'string'` (excludes boxed `new String()`).
  - `isSymbol`: Use `typeof val === 'symbol'`.
  - `isUndefined`: Use `val === undefined` (excludes `null`; avoid `val == null` which collapses both).
- **Simplicity:** Every helper should be a pure, side-effect-free, non-coercive boolean expression.

---

## 🧠 Mental Model

Think of **JavaScript's Primitive Matrix**:
- 5 primitives have unique `typeof` tags: `'string'`, `'number'`, `'boolean'`, `'symbol'`, `'bigint'`.
- 1 primitive has an irregular `typeof` bug: `typeof null === 'object'` (historic 31-year-old JS quirk where the type tag `000` represented both objects and null pointer).
- 1 primitive is uniquely undefined: `typeof undefined === 'undefined'`.
- Strict equality (`===`) is the only reliable way to check singletons (`null`, `undefined`).

---

## 🔑 Key Concepts

- [[Type Checking]]
- JavaScript Primitives vs Object Wrappers
- The `typeof null === 'object'` historical quirk
- Strict equality (`===`) vs Loose equality (`==`)
- `NaN` identity (`typeof NaN === 'number'`)

---

## ⚠️ Edge Cases / Traps

- **`typeof null` Trap:** `typeof null === 'object'`. Checking `typeof val === 'null'` is always `false`. Must use `val === null`.
- **`null == undefined` Collapse:** Using loose equality `val == null` matches both `null` and `undefined`. Strict equality `===` is required.
- **Truthiness vs Boolean:** Testing `Boolean(val)` or `if (val)` checks truthiness, not type. `0` is falsy but is a number; `'hello'` is truthy but is a string.
- **`NaN` is a Number:** `typeof NaN === 'number'` evaluates to `true`. Unless explicitly asked for `Number.isFinite`, `NaN` must return `true` for `isNumber`.
- **Boxed Primitives:** `new Boolean(false)` or `new Number(5)` are object instances (`typeof === 'object'`), not primitive values.

---

## ⭐ Interview Takeaway

1. **Primitive Checking Rule of Thumb:** Use `typeof` for strings, numbers, booleans, and symbols; use strict identity (`===`) for `null` and `undefined`.
2. **`isBoolean` Precision:** `val === true || val === false` is the cleanest and fastest predicate in JavaScript engines.
3. **Building Block for Complex Recursion:** Interviewers ask this to test your defensive programming skills before implementing `deepClone` or schema validators.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why is `typeof null === 'object'` in JavaScript?
- How does `isNumber(NaN)` behave under `typeof` versus `Number.isNaN()`?
- Why should you avoid `val == undefined` when checking if a variable is `undefined`?

### Follow-up Questions
- How would you modify `isNumber` to exclude `NaN` and infinite values? (`typeof val === 'number' && Number.isFinite(val)`)
- How would you handle boxed primitives created via `Object('hello')`?
- How does `Object.prototype.toString.call(val)` compare to using `typeof`?

### Conceptual Questions
- What are all the primitive data types in modern JavaScript (ES2020+)? (string, number, bigint, boolean, undefined, symbol, null)
- What is the difference between value types (primitives) and reference types (objects) in memory allocation?

---

## 🔄 Variations

- **Type Utilities II:** Handling non-primitives (`isArray`, `isFunction`, `isObject`, `isPlainObject`).
- **Strict Number Predicate:** Requiring `Number.isFinite(val)`.
- **Universal Tag Checker:** Implementing `toType(val)` via `Object.prototype.toString.call(val).slice(8, -1).toLowerCase()`.

---

## 📝 Revision Notes

- **Core idea:** Use `typeof` for category primitives; use `===` for `null` and `undefined`.
- **Remember:** `typeof null === 'object'` and `null == undefined` are classic traps.
- **Watch out for:** `isBoolean` must test `val === true || val === false` to reject truthy/falsy values.
- **Complexity:** Time: $O(1)$; Space: $O(1)$.

## Official Solution
## Type Utilities ( Official solution )

Yangshun Tay
Ex-Meta Staff Engineer
Languages

## Solution

The code here is short, but the interview skill is choosing the exact runtime check for each type instead of relying on coercion, truthiness, or overly broad checks.

Ask the narrow runtime question for each helper: "Is this value exactly this primitive type?" Most helpers are one-line predicates, but the details matter because JavaScript has a few special cases.

Each helper accepts one primitive category and rejects lookalikes. None of these checks should coerce the value, allocate wrapper objects, or call methods on the input.

Derive each predicate by first choosing the JavaScript operation that answers the exact question:

1. Prefer `typeof` when the runtime type string is unique for the primitive.
2. Prefer strict equality when the value is a singleton primitive like `null` or `undefined`.
3. Spell out booleans as the two possible boolean primitives instead of asking whether the input is truthy.

These helpers fall into three buckets:

- `typeof` works well for primitive categories like strings, numbers, and symbols.
- `null` and `undefined` need exact equality checks because `typeof null` is `'object'` and `null == undefined` is `true`.
- Booleans are easiest to express as `value === true || value === false`, which excludes truthy and falsy non-booleans.

```javascript
typeof NaN; // 'number'
typeof null; // 'object'
null == undefined; // true
```

`isNumber()`, `isString()`, and `isSymbol()` can use `typeof` directly. This also means `NaN` correctly counts as a number, as required by the prompt.

`isNull()` and `isUndefined()` use strict equality so the two values do not collapse into each other. `value === null` only accepts `null`, and `value === undefined` only accepts `undefined`.

`isBoolean()` checks for the two boolean primitive values explicitly. This avoids accidentally accepting truthy values like `1` or `'true'`, and avoids rejecting `false` just because it is falsy.

| Helper | Accepted primitive values | Main trap |
| --- | --- | --- |
| `isBoolean` | `true`, `false` | truthiness is not type checking |
| `isNumber` | all numbers, including `NaN` | finiteness checks are too narrow |
| `isNull` | only `null` | `typeof null` is `'object'` |
| `isString` | string primitives | boxed strings are objects |
| `isSymbol` | symbol primitives | symbols are not strings |
| `isUndefined` | only `undefined` | loose equality collapses `null` and `undefined` |

```jsx
/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isBoolean(value) {
  return value === true || value === false;
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isNumber(value) {
  return typeof value === 'number';
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isNull(value) {
  return value === null;
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isString(value) {
  return typeof value === 'string';
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isSymbol(value) {
  return typeof value === 'symbol';
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isUndefined(value) {
  return value === undefined;
}
```

## Common pitfalls

- **Using truthiness for booleans:** `Boolean(value)` and `!!value` answer whether a value is truthy, not whether it is a boolean. They would treat `1`, `'hello'`, and many objects as `true`, while `isBoolean()` should only return `true` for `true` and `false`.
- **Using loose equality for nullish values:** `value == null` matches both `null` and `undefined`. That is useful in some application code, but not here because `isNull()` and `isUndefined()` must distinguish the two values.
- **Filtering out `NaN`:** `NaN` is still a number according to `typeof`, and the question explicitly says it should return `true` from `isNumber()`. Do not use `Number.isFinite()` or similar checks that exclude `NaN`.

## Notes

- These helpers check primitive values at runtime. Boxed wrapper objects such as `new String('hello')` or `new Boolean(false)` have type `'object'` and are not accepted by the corresponding primitive checks.
- `isNumber()` accepts every value whose runtime type is `'number'`, including `NaN`, `Infinity`, and `-Infinity`.
- BigInts are not part of this question. `typeof 1n` is `'bigint'`, so `isNumber(1n)` would return `false`.
- Because symbols can throw when coerced to strings, avoiding coercion is not just cleaner; it keeps the predicates safe for every tested primitive.

## Techniques

- Exact runtime type checks

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A number predicate is implemented as `!isNaN(value)` using the coercing global isNaN. Which input proves that this checks convertibility rather than the primitive number type?
