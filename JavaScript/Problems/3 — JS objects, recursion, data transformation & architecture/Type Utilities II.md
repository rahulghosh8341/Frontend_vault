---
title: Type Utilities II
aliases:
  - Type Utilities II
difficulty: Easy
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/type-utilities-ii"
pattern:
  - "[[Type Checking]]"
concepts:
  - "[[Type Checking]]"
  - "[[Prototype]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Type Utilities II

> [!info] Problem
> Implement utilities to determine non-primitive variable types in JavaScript

## Problem

## Type Utilities II

Yangshun Tay
Ex-Meta Staff Engineer
JavaScript is a dynamically typed language, which means the types of variables can change at runtime. Many interview questions involve recursively traversing objects that can hold different value types, and each type may require different handling (e.g. different code is needed to iterate over an array vs. an object). Understanding JavaScript types is crucial to solving questions like [Deep Clone](/questions/javascript/deep-clone) and [Deep Equal](/questions/javascript/deep-equal).

The [Type Utilities](/questions/javascript/type-utilities) question covered utility functions for primitive values. Implement the following utility functions to determine the types of non-primitive values.

- `isArray(value)`: Return `true` if `value` is an array, `false` otherwise.
- `isFunction(value)`: Return `true` if `value` is a function, `false` otherwise.
- `isObject(value)`: Return `true` if `value` is an object (e.g. arrays, functions, plain objects, etc., excluding `null` and `undefined`), `false` otherwise.
- `isPlainObject(value)`: Return `true` if `value` is a plain object, `false` otherwise (for arrays, functions, etc).
  - A plain object, or what is commonly known as a Plain Old JavaScript Object (POJO), is any object whose prototype is `Object.prototype` or an object created via `Object.create(null)`.

## Hints

### Hint 1 : Which values are object-like here?

### Hint 2 : What makes an object plain?

## 🤔 Thought Process

- **Immediate Recognition:** Non-primitive type identification (`isArray`, `isFunction`, `isObject`, `isPlainObject`).
- **Core Problem:** Differentiating between generic objects, callable objects, arrays, and Plain Old JavaScript Objects (POJOs).
- **Rules per Helper:**
  - `isArray`: Delegate directly to native `Array.isArray(value)` (handles cross-iframe realm boundaries).
  - `isFunction`: Use `typeof value === 'function'` (matches arrow functions, class constructors, function expressions).
  - `isObject`: Exclude `null` and `undefined` via `value != null`, then check `typeof value === 'object' || typeof value === 'function'`.
  - `isPlainObject`: Must be an object whose prototype is strictly `Object.prototype` (e.g. `{}`) OR `null` (e.g. `Object.create(null)`).
- **Prototype Inspection:** Use `Object.getPrototypeOf(value)` to inspect the prototype directly rather than reading `.constructor` (which can be forged or shadowed).

---

## 🧠 Mental Model

Think of **JavaScript's Object Hierarchy**:
- At the top of the prototype chain is `Object.prototype` (whose prototype is `null`).
- **POJOs** sit exactly 1 step away from `Object.prototype` (`Object.getPrototypeOf(obj) === Object.prototype`) or have no prototype (`Object.create(null)`).
- **Exotic / Complex Objects** (`Array`, `Date`, `Map`, `RegExp`, custom class instances) have intermediate prototypes in their chain (`Array.prototype`, `Date.prototype`).
- Functions are objects with a special internal `[[Call]]` slot, causing `typeof` to report `'function'`.

---

## 🔑 Key Concepts

- [[Type Checking]]
- Plain Old JavaScript Object (POJO) definition
- Prototype chain inspection (`Object.getPrototypeOf`)
- Cross-realm object identity and `Array.isArray`
- `typeof` bifurcation: functions are objects, but `typeof` says `'function'`

---

## ⚠️ Edge Cases / Traps

- **`Object.create(null)` POJO:** Objects created without prototypes (`Object.create(null)`) have prototype `null`. They are valid plain objects! Checking only `Object.getPrototypeOf(val) === Object.prototype` fails on them.
- **Functions are Objects:** In JS, functions are objects. If an interviewer asks "is a function an object?", the language definition says yes. `isObject` must include `typeof value === 'function'`.
- **Class Instances are NOT Plain Objects:** An instance of `class User {}` has `User.prototype`, not `Object.prototype`. It must return `false` for `isPlainObject`.
- **Shadowed `.constructor` Property:** Avoid checking `value.constructor === Object`. A plain object `{ constructor: 'fake' }` breaks constructor checks. Always check the actual prototype.
- **Cross-Realm Arrays:** Using `value instanceof Array` fails across iframes or web worker windows because `window1.Array !== window2.Array`. Always use `Array.isArray()`.

---

## ⭐ Interview Takeaway

1. **POJO Definition:** An object is "plain" if and only if `Object.getPrototypeOf(val) === Object.prototype || Object.getPrototypeOf(val) === null`.
2. **Defensive Non-Null Guard:** Always guard with `if (value == null || typeof value !== 'object') return false;` before attempting prototype queries.
3. **`Array.isArray` Over `instanceof`:** `Array.isArray` is cross-realm safe and standard ECMAScript.

---

## 🎯 Common Interview Questions

### Direct Questions
- What defines a "plain object" (POJO) in JavaScript?
- Why does `instanceof Array` fail across multiple execution contexts (like iframes)?
- Why is `typeof value === 'function'` included in `isObject`?

### Follow-up Questions
- How does Lodash implement `isPlainObject` for objects across different realms? (Iterates `Object.getPrototypeOf` until reaching the root prototype).
- How would you test if a value is an iterable object (like `Map` or `Set`)?
- What does `Object.prototype.toString.call(val)` return for class instances versus plain objects?

### Conceptual Questions
- What is the difference between `Object.getPrototypeOf(obj)` and `obj.__proto__`?
- How does the prototype chain determine property inheritance at runtime?

---

## 🔄 Variations

- **Deep Plain Object Verification:** Checking whether all nested properties are also plain objects or primitives.
- **Empty Object Predicate:** `isEmptyObject(val)` checking `Reflect.ownKeys(val).length === 0`.
- **Record / Dictionary Type Guard:** TypeScript type predicates (`value is Record<string, unknown>`).

---

## 📝 Revision Notes

- **Core idea:** `isArray` uses `Array.isArray()`; `isObject` includes functions but excludes null/undefined; `isPlainObject` checks prototype is `Object.prototype` or `null`.
- **Remember:** `Object.create(null)` is a valid plain object.
- **Watch out for:** Never rely on `val.constructor === Object` because `constructor` can be overwritten.
- **Complexity:** Time: $O(1)$; Space: $O(1)$.

## Official Solution
## Type Utilities II ( Official solution )

Yangshun Tay
Ex-Meta Staff Engineer
Languages

## Solution

Each helper is a small predicate over a single value. The flow is:

1. Use the most specific built-in check when one exists.
2. Exclude `null` and `undefined` before reading properties or prototypes.
3. Use the value's prototype only for deciding whether an object is plain.

| Helper | Broad rule | Representative true values | Representative false values |
| --- | --- | --- | --- |
| `isArray` | native array check | `[]`, `new Array(3)` | typed arrays, objects |
| `isFunction` | `typeof value === 'function'` | functions, arrows, classes | objects, arrays |
| `isObject` | non-null object or function | arrays, dates, maps, functions | primitives, `null`, `undefined` |
| `isPlainObject` | prototype is `Object.prototype` or `null` | `{}`, `Object.create(null)` | arrays, dates, class instances |

### isArray

`Array.isArray()` is the recommended answer. It exists specifically for this check and handles cross-realm arrays correctly.

As a fallback, a constructor check can work for this interview-sized version, but it still needs to guard `null` and `undefined`.

### isFunction

This one is straightforward: `typeof value === 'function'`. Function declarations, arrow functions, classes, and callable objects all report the `function` type.

### isObject

For this helper, "object" means any non-null object-like value, including arrays, dates, errors, regular expressions, maps, sets, wrapper objects, class instances, and functions.

The code first rejects `null` and `undefined` with `value == null`. After that, it accepts both JavaScript categories that should count here:

- `typeof value === 'object'`
- `typeof value === 'function'`

That function branch matters because functions are objects in the sense used by this question, but `typeof` reports them separately.

### isPlainObject

A value counts as a plain object when its prototype is either `Object.prototype` or `null`. That covers both common cases:

1. Objects without prototypes, created with `Object.create(null)`.
2. Objects created from object literals such as `{}`.

Checking the prototype keeps arrays, dates, maps, sets, errors, regular expressions, and custom class instances out of the result. It also avoids relying on the `constructor` property, which can be missing or shadowed by an own property.

The setup also includes an `isPlainObjectAlternative` implementation based on Lodash's more defensive prototype-chain walk. That broader library-style check is useful when comparing against the root object prototype of the value's own realm, but the direct prototype check is enough for this exercise's definition.

```jsx
/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isArray(value) {
  return Array.isArray(value);
}

// Alternative to isArray.
/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isArrayAlt(value) {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  return value.constructor === Array;
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isFunction(value) {
  return typeof value === 'function';
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isObject(value) {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  const type = typeof value;
  return type === 'object' || type === 'function';
}

/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isPlainObject(value) {
  // For null and undefined.
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
}

// Alternative to isPlainObject, Lodash's implementation.
/**
 * @param {unknown} value
 * @returns {boolean}
 */
export function isPlainObjectAlternative(value) {
  if (!isObject(value)) {
    return false;
  }

  // For objects created via Object.create(null);
  if (Object.getPrototypeOf(value) === null) {
    return true;
  }

  let proto = value;
  while (Object.getPrototypeOf(proto) !== null) {
    proto = Object.getPrototypeOf(proto);
  }

  return Object.getPrototypeOf(value) === proto;
}
```

## Common pitfalls

- **Using `typeof value === 'object'` for everything:** `typeof null` is `'object'`, so `null` must be handled before accepting object values. Functions also need special handling because `typeof fn` is `'function'`, but `isObject(fn)` should still return `true`.
- **Reading `value.constructor` without a null check:** `null` and `undefined` do not have properties. Any fallback that reads `value.constructor` must reject those values first.
- **Using `constructor` to detect plain objects:** Plain object detection should be based on prototypes, not `value.constructor`. An object literal can have its own `constructor` property, and `Object.create(null)` has no inherited `constructor` at all.

## Notes

- `isObject()` is intentionally broad: arrays, functions, wrapper objects, built-in object instances, and custom instances all return `true`.
- `isPlainObject()` is intentionally narrow: only object literals and objects with a `null` prototype return `true`.
- `Array.isArray()` is preferred over the constructor fallback because it handles arrays from other realms.

## Techniques

- Familiarity with JavaScript types.
- Objects and prototypes.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A plain-object detector returns `value.constructor === Object`. Give one valid plain object it rejects because constructor is missing, and another it rejects because constructor is shadowed. What property should the detector inspect instead?

Your notes (optional)
