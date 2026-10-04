---
title: "Deep Omit"
aliases:
  - "deepOmit"
  - "Deep Omit"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Omit

> [!info] Problem
> Implement a function that removes specified keys and their corresponding values from an object, including nested objects or arrays

## Problem

## Deep Omit

Implement a function `deepOmit(obj, keys)` that removes specified keys and their corresponding values from an object, including nested objects or arrays. It works recursively to traverse the entire object structure, ensuring that all occurrences of the specified keys are removed at all levels. The function takes an object (`obj`) and an array of string keys (`keys`).

## Behavior guide

- Return a new object or array for every traversed object or array.
- Remove matching keys from plain objects at every depth.
- Recurse into arrays, but do not remove array elements just because their index appears in `keys`.
- Leave non-plain objects such as `Date`, `RegExp`, `Map`, and `Set` as-is.
- Ignore keys that do not exist.

## Examples

```javascript
deepOmit({ a: 1, b: 2, c: 3 }, ['b']); // { a: 1, c: 3 }
```

A more complicated example with nested objects:

```javascript
const obj = {
  a: 1,
  b: 2,
  c: {
    d: 3,
    e: 4,
  },
  f: [5, 6],
};
deepOmit(obj, ['b', 'c', 'e']); // { a: 1, f: [5, 6] }
```

## Hints

### Hint : Where do the keys apply?

## 🤔 Thought Process

- **Immediate Recognition:** Deep object pruning / blacklisting utility.
- **Core Requirements:**
  - Takes `obj` and `keys` (array of strings to omit).
  - Recursively removes matching keys from plain objects at all nesting levels.
  - Recurses into arrays without treating array indices as keys to omit.
  - Leaves non-plain objects (`Date`, `RegExp`, `Map`, `Set`) unmodified.
  - Returns brand new objects and arrays (pure function / immutable).
- **Optimization:** Convert `keys` array to a `Set` for $O(1)$ membership lookups.
- **Traversal Strategy:**
  - Base case: If primitive, `null`, or non-plain object/array -> return directly.
  - If array: `obj.map(item => deepOmit(item, keys))`.
  - If plain object: Iterate own keys; if `keySet.has(key)`, skip; otherwise `out[key] = deepOmit(obj[key], keys)`.

---

## 🧠 Mental Model

Think of a **Sanitization / Security Filter**:
- When preparing data for external logging or public API responses, sensitive fields (e.g. `password`, `ssn`, `auth_token`) must be stripped at every depth.
- `deepOmit` traverses the data structure, pruning matching keys while preserving the rest of the object hierarchy.

---

## 🔑 Key Concepts

- [[Recursion]]
- Set membership lookup ($O(1)$)
- Immutability / Pure functions
- Plain object detection vs built-in object instances

---

## ⚠️ Edge Cases / Traps

- **Array Indices:** If `keys = ['0', '1']`, an array `['a', 'b']` must NOT omit elements `'a'` or `'b'`. Keys only apply to object properties.
- **Inherited Prototype Properties:** Only own enumerable properties should be traversed (`Object.keys` or `Object.entries`).
- **Non-plain Objects:** Do not strip properties from instances of `Date`, `RegExp`, `Map`, or `Set`.
- **Empty Keys Array:** If `keys` is empty, return a fresh deep clone of the structure.

---

## ⭐ Interview Takeaway

- Convert `keys` to `const keySet = new Set(keys)` at the start to eliminate repetitive $O(K)$ array searches.
- Ensure strict separation between array recursion (recursing elements without checking keys) and object recursion (filtering keys against `keySet`).

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why convert the `keys` array into a `Set`?" (Reduces key lookup time from $O(K)$ to $O(1)$ for every property visited).
- "How do you ensure arrays don't accidentally drop items when a key is a number string like `'0'`?" (Do not check keys when traversing arrays; only map their elements).

### Follow-up Questions
- "How would you implement the inverse function `deepPick`?" (Whitelisting paths instead of blacklisting keys).
- "What if `keys` specified dot-notation paths like `user.profile.password` rather than global key names?"

### Conceptual Questions
- "Where is `deepOmit` commonly used in frontend web applications?" (Redux state serialization, API response sanitization, and stripping ephemeral UI state before POSTing payloads).

---

## 🔄 Variations

- **Lodash Omit:** Shallow key omission on top-level properties.
- **Deep Pick:** Retaining only specified keys across nested objects.
- **Deep Mask:** Replacing sensitive property values with masked placeholders instead of removing them.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
function isPlainObject(val) {
  if (val === null || typeof val !== 'object') return false;
  const proto = Object.getPrototypeOf(val);
  return proto === null || proto === Object.prototype;
}

export default function deepOmit(obj, keys) {
  const keySet = new Set(keys);

  function omit(val) {
    if (Array.isArray(val)) {
      return val.map(omit);
    }
    if (isPlainObject(val)) {
      const result = {};
      for (const [k, v] of Object.entries(val)) {
        if (!keySet.has(k)) {
          result[k] = omit(v);
        }
      }
      return result;
    }
    return val;
  }

  return omit(obj);
}
```

---

## Official Solution

## Deep Omit ( Official solution )

Languages
`deepOmit()` is copy-on-write traversal: rebuild only visited arrays and plain objects, omit matching object keys, and return every primitive or special object unchanged. The common mistake is to treat arrays and non-plain objects like ordinary records.

## Solution

`deepOmit()` is a recursive copy problem: traverse arrays and plain objects, omit matching object keys, and leave everything else untouched. The original input should not be modified, so every traversed array or object is rebuilt.

Since the input can be deeply nested, recursion is the simplest way to walk it. Each call decides what kind of value it received:

1. **Arrays**: Use `Array.isArray()` to detect them, recurse into each item, and return a new array.
2. **Plain objects**: Use `isPlainObject()` so values like `Date` and `Set` are not treated like ordinary object records. Copy only the keys that are not listed in `keys`, and recurse into each retained value.
3. **Everything else**: Return the value as-is.

Those branches are deliberately narrow. Trying to clone every object-like value sounds more general, but it changes the behavior of built-ins such as `Date`, `RegExp`, `Map`, and `Set`. For this prompt, only array containers and plain object records participate in traversal.

The key distinction is between removing object properties and traversing array contents. Array indices are not object keys for this problem. Arrays keep their order and length, but any object inside an array still has matching keys omitted.

That also means an omitted key stops traversal for the value behind that key. If a whole property is removed, none of its descendants need to be inspected.

For `deepOmit({ a: 1, b: [{ c: 2, b: 3 }] }, ['b'])`, the traversal behaves like this:

| Value visited | Decision | Result fragment |
| --- | --- | --- |
| root object | omit key `b`, keep key `a` | `{ a: ... }` |
| `a: 1` | primitive | `1` |
| array under omitted `b` | not visited because the whole key was omitted | none |

If the key was `['c']` instead, the array would stay in place and the object inside it would become `{ b: 3 }`.

```jsx
/**
 * @param {any} val
 * @param {Array<string>} keys
 * @returns any
 */
export default function deepOmit(val, keys) {
  // Handle arrays.
  if (Array.isArray(val)) {
    return val.map((item) => deepOmit(item, keys));
  }

  // Handle objects.
  if (isPlainObject(val)) {
    const newObj = {};
    for (const key in val) {
      if (!keys.includes(key)) {
        newObj[key] = deepOmit(val[key], keys);
      }
    }

    return newObj;
  }

  // Other values can be returned directly.
  return val;
}

function isPlainObject(value) {
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
}
```

Both arrays and objects can also be traversed with a single `for...in` loop. That works here because the omitted keys are strings and will not accidentally match array indices. The shorter version below is valid, but it is harder to read and harder to type safely in TypeScript.

```jsx
export default function deepOmit(val: unknown, keys: Array<string>): unknown {
  if (!Array.isArray(val) && !isPlainObject(val)) {
    return val;
  }

  // Both arrays and objects can be traversed using `for...in` statements.
  const newObj: any = Array.isArray(val) ? [] : {};
  for (const key in val) {
    if (!keys.includes(key)) {
      newObj[key] = deepOmit((val as any)[key], keys);
    }
  }

  return newObj;
}

function isPlainObject(value: unknown): boolean {
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
}
```

## Common pitfalls

- **Deleting keys in place:** Do not call `delete` on the input object. Rebuild plain objects with only the retained keys so callers keep their original data unchanged.
- **Recursing into non-plain objects:** Values like `Date`, `RegExp`, `Map`, `Set`, and `Symbol` values are not ordinary object records in this question. Return them as-is instead of trying to copy or inspect their internals.
- **Removing array elements:** The `keys` array contains object property names, not values to filter from arrays. Arrays should keep their order and length while their nested contents are recursively processed.
- **Omitting only the first matching key:** The traversal must remove every matching object key at every depth, including objects nested inside arrays.

## Notes

- Arrays should keep their order and length; only nested object keys are removed from the copied contents.
- Non-plain objects such as `Date` or `RegExp` should be returned as-is.
- Missing keys should be ignored rather than treated as errors.
- Values like `Date`, `Symbol`, and `RegExp` may appear in the input.
- To keep the question simple, `Map`s and `Set`s do not need to be traversed. There are no test cases containing them, but adding support is possible.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A refactor applies the omit list to every key from `Object.entries`, including array indices. Which input catches a violation of this question's distinction between record properties and array positions?
