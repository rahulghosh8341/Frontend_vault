---
title: Implement size(collection) so it returns the number of items in a supported collection
aliases:
  - Size
difficulty: Easy
time: 15 min
languages:
  - JavaScript
concepts:
  - "[[Type Checking]]"
  - "[[Object Property Enumeration]]"
solved: true
solvedDate: 2026-09-03
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Returns the number of items in a collection (Arrays/Strings use `.length`, Map/Set use `.size`, Objects use `Object.keys().length`).

## Problem

Implement `size(collection)` so it returns the number of items in a supported collection.

- Arrays and strings use `.length`.
- Plain objects use the number of own enumerable properties.
- `Map` and `Set` use `.size`.
- Return `0` for `null`, `undefined`, or unsupported values.

```js
size([1, 2, 3, 4, 5]);               // => 5
size({ a: 1, b: 2 });                // => 2
size('peanut');                      // => 6
size(new Set([1, 2, 3]));            // => 3
size(new Map([[1, 2], [3, 4]]));     // => 2
size(null);                          // => 0
size(undefined);                     // => 0
```

---

## 🤔 Thought Process

1. Handle `null` / `undefined` → return `0`.
2. Strings and arrays → use `.length`.
3. `Map` and `Set` → use `.size`.
4. Plain objects → use `Object.keys().length`.
5. Anything else → return `0`.

---

## 💻 Final Solution

```js
export default function size(collection) {
  if (typeof collection === 'undefined' || collection === null)
    return 0;

  if (typeof collection == 'string' || Array.isArray(collection))
    return collection.length;

  if (collection instanceof Map || collection instanceof Set)
    return collection.size;

  if (typeof collection === "object")
    return Object.keys(collection).length;
  
  return 0;
}
```

---

## 🤔 Why This Works

### `Map` / `Set` Before Object

Both are objects:

```js
typeof new Map() // "object"
typeof new Set() // "object"
```

So the `Map`/`Set` check must come **before**:

```js
typeof collection === 'object'
```

Otherwise `Object.keys(map).length` runs first.

### Different Collection APIs

```text
Array / String → length
Map / Set      → size
Object         → Object.keys().length
```

---

## 🐞 Bugs I Made

### `Map` / `Set` check was too late

Your code had:

```js
if (typeof collection === 'object')
  return Object.keys(collection).length;

if (collection instanceof Map || collection instanceof Set)
  return collection.size;
```

The generic object condition catches `Map` and `Set` first.

**Fix:** Check `Map`/`Set` before generic objects.

---

## Production Considerations

- In production code, Lodash `_.size(collection)` handles broader edge cases like Array-like objects (`arguments`, DOM NodeLists).
- Note that `Object.keys()` only counts **own enumerable string keys**. Symbol keys or non-enumerable properties are not included.
- For primitive numbers or booleans, `size(123)` correctly returns `0`.

---

## ⭐ Revision Notes

### 🔑 Key Facts

* `typeof null === 'object'` → explicitly handle `null`.
* `typeof new Map() === 'object'`.
* `typeof new Set() === 'object'`.
* `Map`/`Set` use `.size`.
* Arrays/strings use `.length`.
* Plain objects use `Object.keys(obj).length`.
* Condition order matters when one condition is more general than another.

### 🧠 Mental Model

```text
              collection
                   ↓
          null / undefined?
             ↓ yes → 0
                   ↓ no
          String / Array?
             ↓ yes → length
                   ↓ no
             Map / Set?
             ↓ yes → size
                   ↓ no
             Object?
             ↓ yes → Object.keys().length
                   ↓ no
                   0
```

### Common Interview Questions

- Why must `instanceof Map` / `instanceof Set` be checked before `typeof collection === 'object'`? → Because `typeof new Map()` returns `"object"`, so a generic object check would intercept Maps and Sets prematurely.
- How do you get the number of properties on a plain object? → `Object.keys(obj).length` (or `Reflect.ownKeys(obj).length` if symbols are included).
- What does `typeof null` return? → `"object"`, which is a historical JS bug.

### ⭐ Interview Takeaways

> **When multiple types are objects, check specific types (`Map`, `Set`) before the generic `typeof === 'object'` check.**

Your approach was otherwise correct.

### Related

- [[Type Checking]]
- [[Object Property Enumeration]]
