---
title: Implement classNames II with deduplication, class toggling, and function evaluation
aliases:
  - Classnames II
difficulty: Hard
time: 30 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Higher Order Mapping]]"
concepts:
  - "[[Set Manipulation]]"
  - "[[Function Execution]]"
  - "[[Class Toggling]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-09-10
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 30 min
> Enhanced `classNames` utility supporting deduplication, class toggling (`{ foo: false }` removes `'foo'`), and executing function arguments.

## Problem

The original `classnames` utility does not handle deduplication, turning off classes via falsey object properties, or evaluating function values.

Implement an improved `classNames` function that handles:

1. **Deduplication:** `classNames('foo', 'foo')` → `'foo'`
2. **Class Toggling / Turn off:** `classNames('foo', 'bar', { foo: false })` → `'bar'`
3. **Function Values:** `classNames('foo', () => 'bar')` → `'foo bar'`

**Examples**

```js
classNames('foo', 'foo'); // 'foo'
classNames({ foo: true }, { foo: true }); // 'foo'
classNames({ foo: true, bar: true }, { foo: false }); // 'bar'
classNames('foo', () => 'bar'); // 'foo bar'
classNames('foo', () => 'foo'); // 'foo'
```

## Pattern

- [[Higher Order Mapping]]

## 🤔 Thought Process

* Use a single shared `Set` to store active class names.
* Traverse arguments in order:
  * Ignore falsy values.
  * `string` / `number`: add to `Set`.
  * `function`: invoke `arg()`. If truthy, add result to `Set`.
  * `Array`: recursively process each element.
  * `object`: iterate own keys. If property value is truthy → `classes.add(key)`. If falsey → `classes.delete(key)`.
* Joining the `Set` with spaces handles both deduplication and class removal, while preserving the "later overrides earlier" order.

## 💻 Final Solution

```js
/**
 * @typedef {Record<string, unknown>} ClassDictionary
 * @typedef {Array<ClassValue>} ClassArray
 * @typedef {string | number | null | boolean | undefined | (() => unknown) | ClassDictionary | ClassArray} ClassValue
 */

/**
 * @param {...ClassValue} args
 * @returns {string}
 */
export default function classNames(...args) {
  // One shared Set lets deeper recursive calls add or remove classes while
  // preserving the global "later values win" ordering.
  const classes = new Set();

  function classNamesImpl(...args) {
    args.forEach((arg) => {
      // Ignore falsey values.
      if (!arg) {
        return;
      }

      const argType = typeof arg;

      // Handle string and numbers.
      if (argType === 'string' || argType === 'number') {
        classes.add(String(arg));
        return;
      }

      // Handle functions.
      if (argType === 'function') {
        const result = arg();
        if (!result) {
          return;
        }

        classes.add(String(result));
      }

      // Arrays recurse before objects because `typeof []` is `'object'`.
      if (Array.isArray(arg)) {
        for (const cls of arg) {
          classNamesImpl(cls);
        }

        return;
      }

      // Objects can both enable and disable keys, so they must mutate the
      // shared Set directly instead of returning partial strings.
      if (argType === 'object') {
        for (const key in arg) {
          if (Object.hasOwn(arg, key)) {
            arg[key] ? classes.add(key) : classes.delete(key);
          }
        }

        return;
      }
    });
  }

  classNamesImpl(args);

  return Array.from(classes).join(' ');
}
```

## 🤔 Why This Works

* **Shared `Set` State:** `Set.prototype.add()` adds new keys, while `Set.prototype.delete()` removes disabled keys.
* **Order Preservation:** `Set` iterates items in insertion order. When `{ foo: false }` deletes `'foo'`, subsequent additions like `'foo'` re-insert it at the end.
* **Function Evaluation:** Functions are invoked synchronously, and their returned values are coerced to strings and added to the `Set`.
* **Array Recursion:** Recursing through arrays flattens nested inputs while maintaining left-to-right evaluation order.

## 🐞 Bugs I Made

* None in final submission.
* **Key Detail:** Array check (`Array.isArray(arg)`) must happen **before** generic object check (`argType === 'object'`) because `typeof [] === 'object'`.

## Production Considerations

- Functions should be pure and side-effect free since they are executed during string building.
- Using a `Set` provides $O(1)$ additions, deletions, and lookup, making overall processing $O(N)$ where $N$ is total elements/keys evaluated.

## ⭐ Revision Notes

### 🔑 Key Facts

* `Set` methods `add()` and `delete()` enable dynamic state toggling.
* `typeof fn === 'function'` identifies function arguments to invoke.
* Order of type checks: Primitive → Function → Array → Plain Object.
* `Array.from(set).join(' ')` converts the final set of classes to space-separated output.

### 🧠 Mental Model

```text
Input args
    ↓
Iterate each item
    ├─ Primitive → classes.add(val)
    ├─ Function  → run fn() → classes.add(result)
    ├─ Array     → recurse each item
    └─ Object    → key value true ? classes.add(key) : classes.delete(key)
    ↓
Array.from(classes).join(' ')
```

### Common Interview Questions

- What happens if a function returns an object or array? You can pass the function result through `classNamesImpl` recursively if supporting nested function return types.
- Why is `Set` ideal here? It natively deduplicates and provides $O(1)$ removal via `delete(key)`.

### Interview Takeaways

* Shared `Set` handles both deduplication and explicit property deletion cleanly.
* Maintain strict left-to-right order when processing mixed types.

### Related

- [[Classnames]]
- [[Higher Order Mapping]]
- [[Type Checking]]
