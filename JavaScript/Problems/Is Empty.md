---
title: Implement isEmpty(value)
aliases:
  - Is Empty
difficulty: Medium
time: 15 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Type Checking]]"
concepts:
  - "[[Array]]"
  - "[[Object]]"
  - "[[Map]]"
  - "[[Set]]"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 15 min
> Check if a value has no items to inspect across various collection types.

## Problem

Implement `isEmpty(value)` so it returns `true` when `value` has no items to inspect.

- Arrays and strings are empty when their length is 0.
- Maps and sets are empty when their size is 0.
- Plain objects are empty when they have no own enumerable properties.
- For this question, non-collection values such as `null`, booleans, numbers, symbols, and regular expressions should also be treated as empty.
- You do not need to handle DOM or jQuery-like collections.

```js
isEmpty(null); // => true
isEmpty(true); // => true
isEmpty(1); // => true
isEmpty(new Map()); // => true
isEmpty([1, 2, 3]); // => false
isEmpty({ a: 1 }); // => false
```

## Pattern

- [[Type Checking]]

## 🤔 Thought Process

First handle `null`/`undefined` → empty.
Strings and arrays → check `.length`.
Maps and Sets → check `.size`.
Objects → check `Object.keys().length`.
Everything else (numbers, booleans, symbols, regex, etc.) → empty.

## 💻 Final Solution

```js
export default function isEmpty(collection) {
  if (typeof collection === 'undefined' || collection === null)
    return true;

  if (typeof collection === 'string' || Array.isArray(collection))
    return collection.length === 0;

  if (collection instanceof Map || collection instanceof Set)
    return collection.size === 0;

  if (typeof collection === 'object')
    return Object.keys(collection).length === 0;

  return true;
}
```

## 🤔 Why This Works

Different collection types expose their item count differently:

| Type | Property |
|------|----------|
| Array / String | `length` |
| Map / Set | `size` |
| Object | `Object.keys().length` |
| Other values | treated as empty |

For example:

```js
isEmpty([1, 2])        // false
isEmpty([])            // true

isEmpty(new Map())     // true
isEmpty(new Set([1]))  // false

isEmpty({})            // true
isEmpty({ a: 1 })      // false

isEmpty(123)           // true
isEmpty(null)          // true
```

## 🐞 Bugs I Made

No functional bug in the solution.

One improvement: ternary `condition ? true : false` is unnecessary because the condition already produces a boolean.

```js
// Instead of:
return collection.length === 0 ? true : false;

// Just:
return collection.length === 0;
```

Same for `collection.size === 0` and `Object.keys(collection).length === 0`.

## Production Considerations

- Lodash's `_.isEmpty` covers additional types like arguments objects, buffers, etc.
- `typeof null === "object"` — must handle `null` before the generic object check.
- `Array.isArray()` is preferred over `instanceof Array` for cross-frame compatibility.
- `Object.keys()` only checks own enumerable properties, matching the requirement.

## ⭐ Revision Notes

### 🔑 Key Facts

* `typeof null === "object"` → handle `null` before the generic object check.
* `Array.isArray()` is preferred for arrays.
* Map and Set use `.size`, not `.length`.
* `Object.keys()` only checks own enumerable properties, matching the requirement.
* Primitive/non-collection values are treated as empty.
* The order of checks matters because arrays, Maps, Sets, and `null` can all interact with `typeof ... === "object"`.

### 🧠 Mental Model

```text
What kind of collection is this?
          ↓
┌─────────┼──────────┐
String    Array      → length
Map       Set        → size
Object               → Object.keys()
Everything else      → empty
```

### Common Interview Questions

- Why does `typeof null === "object"` matter? → Without a `null` check first, `null` would fall through to the object branch and `Object.keys(null)` throws.
- What about `arguments` object? → It is array-like but not an array; `Object.keys(arguments).length` works. The problem says you don't need to handle it.
- How is this different from `Size`? → `Size` returns the count; `isEmpty` returns a boolean. `isEmpty` is a special case of `Size === 0` for collections, but also treats primitives as empty.

### Interview Takeaways

* Check `null`/`undefined` first.
* Different collections → different size property.
* Primitives → empty.
* Clean: return the boolean expression directly, no ternary.

### Related

- [[Type Checking]]
- [[Size]]
- [[Array]]
- [[Object]]
- [[Map]]
- [[Set]]