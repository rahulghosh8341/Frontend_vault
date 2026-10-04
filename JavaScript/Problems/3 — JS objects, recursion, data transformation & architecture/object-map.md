---
title: "Object Map"
aliases:
  - "objectMap"
  - "Object Map"
difficulty: "Easy"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Object Map

> [!info] Problem
> Implement a function to transform values within an object

## Problem

## Object Map

Implement a function `objectMap(obj, fn)` to return a new object containing the results of calling a provided function on every value in the object. The function `fn` is called with a single argument, the `value` that is being mapped or transformed.

Map only the input object's own enumerable properties, keep the same property names, and do not recurse into nested values. Invoke `fn` with the original input object as its `this` value, and leave the input object unchanged.

## Examples

```javascript
const double = (x) => x * 2;
objectMap({ foo: 1, bar: 2 }, double); // => { foo: 2, bar: 4}
```

## Hints

### Hint : Which parts of the object change?

## 🤔 Thought Process

- **Immediate Recognition:** Shallow dictionary transformation utility analogous to `Array.prototype.map`.
- **Core Requirements:**
  - Non-recursive: Only maps top-level own enumerable properties of the input object.
  - Leaves input object unchanged (pure function).
  - Invokes `fn` with a single argument: `fn.call(obj, value)`.
  - Context binding: `this` inside `fn` must be the original input object.
  - Returns a new object with identical keys and transformed values.
- **Implementation Strategy:**
  - Use `Object.entries(obj)` or `for...in` guarded by `Object.hasOwn`.
  - Initialize empty result `{}`.
  - For each `[key, val]`: `result[key] = fn.call(obj, val)`.
  - Return `result`.

---

## 🧠 Mental Model

Think of an **Array Map for Objects**:
```
Input Object:   { a: 1, b: 2 }
Transformation: (x) => x * 10  (with this === Input Object)
Output Object:  { a: 10, b: 20 }
```
Transforms values while strictly preserving the key structure and object shape.

---

## 🔑 Key Concepts

- [[Higher Order Mapping]]
- [[Closure]]
- Shallow vs Deep transformation
- Context binding via `Function.prototype.call(thisArg, arg)`
- Own enumerable property traversal (`Object.entries`)

---

## ⚠️ Edge Cases / Traps

- **Inherited Prototype Properties:** If using `for...in`, inherited properties on the prototype chain must be skipped using `Object.hasOwn(obj, key)`.
- **`this` Binding Context:** Using `fn(val)` instead of `fn.call(obj, val)` will lose the required `this` binding.
- **Empty Object:** `objectMap({}, fn)` should return `{}` without invoking `fn`.
- **Non-string Keys / Symbols:** Problem specifies standard own enumerable properties; `Object.keys()` / `Object.entries()` handles standard string keys.

---

## ⭐ Interview Takeaway

- Standard 3-line implementation:
  ```javascript
  export default function objectMap(obj, fn) {
    const result = {};
    for (const [k, v] of Object.entries(obj)) {
      result[k] = fn.call(obj, v);
    }
    return result;
  }
  ```
- Always remember that callback functions in utility libraries often expect `this` to point to the source collection.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does JavaScript have `Array.prototype.map` built-in but no `Object.map`?" (Objects are key-value maps with arbitrary structures, prototypes, and symbol keys; `Object.entries()` + `Object.fromEntries()` is the standard modern idiom).
- "How do you achieve this using `Object.fromEntries`?" (`Object.fromEntries(Object.entries(obj).map(([k, v]) => [k, fn.call(obj, v)]))`).

### Follow-up Questions
- "How would you implement `deepMap` that recursively maps nested objects and arrays?"
- "How would you map both keys and values (`objectMapEntries`)?"

### Conceptual Questions
- "What happens if `fn` is an arrow function?" (Arrow functions have lexical `this` that cannot be overridden by `.call(thisArg)`).

---

## 🔄 Variations

- **Deep Map:** Recursively transforming leaf values.
- **Object Filter:** Filtering key-value pairs based on a predicate.
- **Camel Case Keys:** Mapping keys rather than values.

---

## 📝 Revision Notes

- Idiomatic implementation:
```javascript
export default function objectMap(obj, fn) {
  const res = {};
  for (const [key, value] of Object.entries(obj)) {
    res[key] = fn.call(obj, value);
  }
  return res;
}
```

---

## Official Solution

## Object Map ( Official solution )

Premium
Languages

## Solution

`objectMap()` is the object equivalent of `Array.prototype.map()`, but objects add two rules arrays do not have: key ownership and callback receiver. Keep the same own property names, transform each top-level value, bind the callback to the original object, and return a fresh object. It does not recurse into nested objects. Nested values are just values passed to the callback.

It is tempting to focus only on "map the values" and accidentally change the returned object shape. A correct answer must preserve key membership, skip inherited enumerable keys, and call the callback with the original object as `this`.

Map each own property in three steps:

- Iterate over the input object's own enumerable keys.
- Call `fn` for each value with the input object as `this`.
- Store each returned value under the same key in a new result object.

Using `fn.call(obj, obj[key])` keeps the callback semantics explicit. The callback receives the value as its argument, and code inside the callback can access the original object through `this`.

For `{ foo: 2, bar: 3 }` and a doubling callback, the loop keeps the keys fixed while replacing only the values:

| Key | Callback input | Stored output |
| --- | --- | --- |
| `foo` | `2` | `4` |
| `bar` | `3` | `6` |

During both callback calls, `this` is the original object. That fixed-key rule is also why approaches based on mapping only `Object.values(obj)` are incomplete: they discard the property names that must be used to rebuild the result, and plain `fn(value)` drops the receiver rule.

```jsx
/**
 * @param {Record<string, unknown>} obj
 * @param {(value: unknown) => unknown} fn
 * @returns {Record<string, unknown>}
 */
export default function objectMap(obj, fn) {
  const result = {};

  for (const key in obj) {
    if (Object.prototype.hasOwnProperty.call(obj, key)) {
      // Skip inherited keys and preserve `obj` as the callback's `this` value.
      result[key] = fn.call(obj, obj[key]);
    }
  }

  return result;
}
```

The same traversal can also be expressed with `Object.entries()` and `Object.fromEntries()`. It is shorter and reads like "turn the object into pairs, transform the values, then rebuild the object":

```jsx
export default function objectMap<V, R>(
  obj: Record<string, V>,
  fn: (val: V) => R,
): Record<string, R> {
  // Rebuild the object from transformed [key, value] pairs while preserving callback `this`.
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, fn.call(obj, value as V)]),
  ) as Record<string, R>;
}
```

## Common pitfalls

- **Recursing into nested objects:** The function only operates on the top-level keys and values of the object. If a property value is itself an object, that object is passed to `fn` as a single value.
- **Losing the callback `this` value:** Calling `fn(obj[key])` transforms the value, but it does not bind `this`. Use `fn.call(obj, obj[key])` so the callback can read the original input object through `this`.
- **Copying inherited keys:** A `for...in` loop sees inherited enumerable properties too. Guarding with `Object.prototype.hasOwnProperty.call(obj, key)` keeps the result limited to the input object's own keys.

## Notes

- The callback is called with a single `value` argument.
- The returned object has the same own keys as the input object.
- Nested objects are returned as whatever the callback produces for that property.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
An optimization maps an object's values in place and returns that same object:

```javascript
for (const key of Object.keys(obj)) {
  obj[key] = fn.call(obj, obj[key]);
}
return obj;
```

For `obj = { factor: 2, amount: 3 }` and `fn = function (value) { return value * this.factor; }`, explain why this changes more than the returned object's identity.

Your notes (optional)
