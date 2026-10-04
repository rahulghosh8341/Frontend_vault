---
title: "Deep Clone"
aliases:
  - "deepClone"
  - "Deep Clone"
difficulty: "Medium"
source: GreatFrontEnd
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Clone

> [!info] Problem
> Implement a function that performs a deep copy of a JSON-serializable value

## Problem

## Deep Clone

Zhenghao He
Engineering Manager, Robinhood
A deep clone makes a copy of a JavaScript value such that the copy has no shared references to nested arrays or objects in the original. Mutating the cloned value should never affect the original.

## How to deep clone an object in JavaScript

In production code, use the built-in [`structuredClone()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone):

```javascript
const original = { user: { name: 'Ada', tags: ['admin'] } };
const copy = structuredClone(original);

copy.user.tags.push('owner');

console.log(original.user.tags); // ['admin']
console.log(copy.user.tags); // ['admin', 'owner']
```

`structuredClone` is part of the Web Platform spec. It is available in all evergreen browsers, Node.js 17+, and Deno. It correctly handles plain objects and arrays as well as `Date`, `RegExp`, `Map`, `Set`, `ArrayBuffer`, typed arrays, and circular references. None of those work with the older `JSON.parse(JSON.stringify(value))` trick.

### When structuredClone isn't enough

`structuredClone` covers most cases, but it throws a `DataCloneError` for values that cannot be structured-cloned and silently flattens others:

| Input | Behavior |
| --- | --- |
| Functions / methods | Throws `DataCloneError`. |
| DOM nodes | Throws (except via the special `transfer` option for transferable types). |
| Symbol values | Throws. |
| Class instances | Cloned as plain objects. The prototype is **not** preserved. |
| Getters and setters | Flattened to data properties on the clone. |
| Property descriptors | `enumerable`, `writable`, `configurable` flags are not preserved. |

If you need to clone any of those, or you need full control over what gets shared versus copied, you'll write your own `deepClone`. Implementing it from scratch is also one of the most common JavaScript interview questions, because it tests recursion, type detection, and `Object` traversal in a small surface area.

> Looking for the conceptual explanation of "shallow vs deep copy"? See the dedicated quiz page: [Explain the difference between shallow copy and deep copy](/questions/quiz/explain-the-difference-between-shallow-copy-and-deep-copy).

Implement `deepClone(value)` so it returns a deep copy of a JSON-serializable value. The input may be `null`, booleans, numbers, strings, arrays, or plain objects. It will not contain cycles or special objects like `Date`, `RegExp`, `Map`, or `Set`. Primitive values can be returned as-is.

## Arguments

1. `value` *(*)*: The value to clone.

## Returns

*(*)*: Returns a deep copy of `value`.

## Examples

```javascript
const obj1 = { user: { role: 'admin' } };
const clonedObj1 = deepClone(obj1);

clonedObj1.user.role = 'guest'; // Change the cloned user's role to 'guest'.
clonedObj1.user.role; // 'guest'
obj1.user.role; // Should still be 'admin'.

const obj2 = { foo: [{ bar: 'baz' }] };
const clonedObj2 = deepClone(obj2);

obj2.foo[0].bar = 'bax'; // Modify the original object.
obj2.foo[0].bar; // 'bax'
clonedObj2.foo[0].bar; // Should still be 'baz'.
```

## Hints

### Hint : Which values need a new identity?

## Asked at these companies

ByteDance
Amazon
Tiktok
Canva
Ramp
Adobe
PayPal

## 🤔 Thought Process

- **Immediate Recognition:** Classic deep copy problem for JSON-serializable structures (plain objects and arrays).
- **Core Problem:** Creating a new copy of a data structure such that mutating any nested property in the copy never affects the original value.
- **Base Case vs Recursive Step:**
  - Primitives and `null`: Return directly (immutable by nature).
  - Arrays: Map over elements and recursively clone each item.
  - Plain Objects: Iterate over keys, recursively cloning each value onto a new `{}`.
- **Prototype Consideration:** Only plain objects and arrays are required in this variant; preserve `{}` prototype.
- **Traps to Avoid:**
  - `typeof null === 'object'`: Must check `value === null` before treating as an object.
  - `typeof [] === 'object'`: Must use `Array.isArray(value)` to distinguish arrays from plain objects.

---

## 🧠 Mental Model

Think of **Branching a File Directory Tree**:
- When duplicating a folder, you don't just create a new shortcut to the same files.
- You create a brand new physical folder at every sub-level, copying raw files (primitives) across while ensuring all directory containers (arrays/objects) are newly instantiated.

---

## 🔑 Key Concepts

- [[Recursion]]
- Primitive vs Reference types in JavaScript
- Shallow copy vs Deep copy
- `structuredClone()` vs `JSON.parse(JSON.stringify())`

---

## ⚠️ Edge Cases / Traps

- **`null` Trap:** `typeof null === 'object'`. If you check `if (typeof val === 'object')` without guarding `val === null`, you will attempt `Object.keys(null)` and throw an error.
- **Array vs Object Discrepancy:** `typeof [] === 'object'`. Must distinguish using `Array.isArray(val)` so arrays aren't cloned into plain objects with numeric string keys.
- **`NaN`, `Infinity`, `-Infinity`:** Primitives; should be returned as-is (unlike `JSON.stringify`, which corrupts them to `null`).
- **Inherited Prototype Properties:** Use `Object.prototype.hasOwnProperty.call(val, key)` or `Object.keys(val)` to avoid copying properties from the prototype chain.

---

## ⭐ Interview Takeaway

- **Standard 3-way check:**
  1. If `val === null || typeof val !== 'object'`, return `val`.
  2. If `Array.isArray(val)`, return `val.map(deepClone)`.
  3. Otherwise, create `{}` and copy own keys with `deepClone(val[k])`.
- Explain why `JSON.parse(JSON.stringify(x))` is disallowed: drops `undefined`, functions, symbols, converts `NaN`/`Infinity` to `null`, and breaks on circular references.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How does shallow copy differ from deep copy?"
- "Why can't we just use `Object.assign()` or the spread operator `...`?"
- "What built-in browser API natively does deep cloning?" (`structuredClone`)

### Follow-up Questions
- "What happens if the object has circular references?" (Requires `WeakMap` cache; leads to Deep Clone II).
- "How would you handle non-JSON types like `Date`, `RegExp`, `Map`, `Set`?"
- "How do you preserve the object's prototype?" (`Object.create(Object.getPrototypeOf(val))`).

### Conceptual Questions
- "Why does `structuredClone()` throw a `DataCloneError` on functions?" (Functions carry closures/lexical scopes that cannot be serialized safely).

---

## 🔄 Variations

- **Deep Clone II:** Adding circular reference detection via `WeakMap` and special objects (`Date`, `RegExp`, `Map`, `Set`).
- **Shallow Clone:** Implementing `Object.assign()` or spread emulation.
- **Deep Merge / Object Assign Deep:** Recursively combining two objects instead of just cloning one.

---

## 📝 Revision Notes

- Fast pattern:
```javascript
function deepClone(val) {
  if (val === null || typeof val !== 'object') return val;
  if (Array.isArray(val)) return val.map(deepClone);
  const out = {};
  for (const k of Object.keys(val)) {
    out[k] = deepClone(val[k]);
  }
  return out;
}
```

---

## Official Solution

## Deep Clone ( Official solution )

Zhenghao He
Engineering Manager, Robinhood
Languages
Writing out a complete deep clone solution from scratch is almost impossible under typical interview constraints. The scope is usually fairly limited, and interviewers are more interested in data-type detection and using built-in APIs and `Object` methods to traverse a given object.

View deep cloning as rebuilding the supported container tree. Primitive leaves can be returned as-is, but every supported container node needs a new container whose children are recursively cloned.

## Solution

For interview purposes, learn Approach 2 first. The JSON version is useful as a baseline to discuss, but it only works for JSON-safe data and avoids the actual cloning logic.

### Approach 1: JSON.stringify

The tempting but flawed shortcut for deep-copying an object in JavaScript is to serialize it and then deserialize it with `JSON.stringify` and `JSON.parse`.

```javascript
export default function deepClone(value) {
  return JSON.parse(JSON.stringify(value));
}
```

Although this approach is acceptable when the input object only contains `null`, `boolean`, `number`, and `string` values, it has important downsides:

- Only non-symbol-keyed properties whose values are supported by JSON can be copied. Unsupported data types are simply ignored.
- `JSON.stringify` also has a few other surprising behaviors such as converting `Date` objects to ISO timestamp strings, and turning `NaN` and `Infinity` into `null`.

Most interviewers disallow this shortcut.

### Approach 2: Recursion

This is the recommended solution. Traverse the value recursively, cloning arrays element-by-element and plain objects property-by-property.

The recursive pieces are:

1. Stop at primitives and `null`, because they do not need cloning.
2. For arrays, create a new array and recursively clone each element.
3. For objects, enumerate the object's own enumerable string keys, recursively clone each value, and rebuild a new object from those entries.

```jsx
/**
 * @template T
 * @param {T} value
 * @return {T}
 */
export default function deepClone(value) {
  if (typeof value !== 'object' || value === null) {
    // Primitives can be returned directly because they are already immutable values.
    return value;
  }

  if (Array.isArray(value)) {
    // Clone each slot so nested arrays do not share references with the original.
    return value.map((item) => deepClone(item));
  }

  // Rebuild the object with recursively cloned property values.
  return Object.fromEntries(
    Object.entries(value).map(([key, value]) => [key, deepClone(value)]),
  );
}
```

There are generally two ways to traverse an object:

- Loop through the keys with the traditional `for ... in` statement.
- Convert the object into an array of keys with `Object.keys()`, or an array of key-value tuples with `Object.entries()`.

With the `for ... in` statement, inherited enumerable properties are processed as well. On the other hand, `Object.keys()` and `Object.entries()` only include the properties directly defined on the object, and this is usually the intended behavior.

### Approach 3: structuredClone (the modern one-liner)

For production code outside an interview, the right answer is the built-in [`structuredClone`](https://developer.mozilla.org/en-US/docs/Web/API/Window/structuredClone). It is available in all evergreen browsers, Node.js 17+, and Deno.

```javascript
const clonedObj = structuredClone(obj);
```

Cases that `structuredClone` handles correctly and `JSON.parse(JSON.stringify(value))` does not:

- Circular references.
- `Date`, `RegExp`, `Map`, `Set`.
- `ArrayBuffer`, typed arrays, `Blob`, `File`, `FileList`, `ImageData`.
- Most error types.

What it still does **not** handle:

| Input | Behavior |
| --- | --- |
| Functions / methods | Throws `DataCloneError`. |
| DOM nodes | Throws. Transferable types use the `transfer` option instead. |
| `Symbol` values | Throws. |
| Class instances | Cloned as plain objects. The prototype is **not** preserved. |
| Getters and setters | Flattened to data properties on the clone. |
| Property descriptors (`enumerable`, `writable`, `configurable`) | Not preserved. |

See ["Deep-copying in JavaScript using structuredClone" on web.dev](https://web.dev/structured-clone/) for the full reference.

## Choosing an approach in real code

For a deep clone, work down this short decision list:

1. **Plain JSON-serializable data only?** `JSON.parse(JSON.stringify(value))` is fastest for that narrow case. Watch for the `Date` and `NaN`/`Infinity` pitfalls noted in Approach 1.
2. **Anything more complex, but no functions, DOM, or class identity?** Use `structuredClone(value)`. This is the right default for most apps.
3. **Need to preserve prototypes, functions, or property descriptors?** Write a custom recursive clone (Approach 2), or use a well-tested library like Lodash's `cloneDeep`.
4. **Updating React state?** Deep cloning state on every change is almost always the wrong tool. Reach for the spread operator, [Immer](https://immerjs.github.io/immer/), or a `useReducer` with structural updates instead.

## Common pitfalls

- Forgetting the `value === null` check before recursing. `typeof null` is `'object'`, but it is not traversable.
- Using `for ... in` without an own-property guard, which pulls in inherited enumerable properties.
- Expecting `Date`, `RegExp`, `Map`, `Set`, functions, DOM nodes, or class instances to clone correctly in this first recursive version.
- Expecting circular references to work without a visited-object cache.
- Assuming a cloned object also preserves prototypes, getters, setters, or property descriptors.

## Notes

- [Non-enumerable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty#description) and [symbol-keyed](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) properties are ignored.
- [Property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors) are not respected or copied into the cloned object.
- If the object has circular references, the current solution will recurse indefinitely and may cause a stack overflow.
- Prototypes are not copied.

These edge cases are addressed in [Deep Clone II](/questions/javascript/deep-clone-ii).

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A proposed clone returns a new root but uses `slice()` for arrays without cloning their elements. Which test isolates the remaining bug?
