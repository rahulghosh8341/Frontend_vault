---
title: "Deep Set II"
aliases:
  - "deepSetII"
  - "Deep Set II"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Set II

> [!info] Problem
> Extend a deep setter to merge object and array values at the leaf

## Problem

## Deep Set II

This is a follow-up to [Deep Set](/questions/javascript/deep-set).

In the first question, `deepSet(obj, path, value)` always overwrote the value at the target path. In this question, keep the same API but merge object and array values at the leaf when possible.

## Merge rules

When `deepSet(obj, path, value)` reaches the final segment:

- If the existing value and `value` are both plain objects, recursively merge them by key.
- If the existing value and `value` are both arrays, recursively merge them by index.
- Otherwise, overwrite the existing value with `value`.

These merge rules apply recursively within the leaf value. When nested values have compatible container types, merge them using the same rules. Otherwise, overwrite the nested existing value.

Intermediate traversal rules stay the same as the previous question:

- Missing containers should be created automatically.
- Numeric next segments should create arrays.
- Primitive intermediate branches may be replaced if traversal must continue.

## Examples

Merge plain objects at the leaf.

```javascript
const object = {
  settings: {
    theme: {
      mode: 'light',
      contrast: 'normal',
    },
  },
};

deepSet(object, 'settings.theme', { mode: 'dark' });

// {
//   settings: {
//     theme: {
//       mode: 'dark',
//       contrast: 'normal',
//     },
//   },
// }
```

Merge arrays by index.

```javascript
const object = {
  items: [{ done: false }, { done: false }, { done: true }],
};

deepSet(object, 'items', [{ done: true }]);

// {
//   items: [{ done: true }, { done: false }, { done: true }],
// }
```

Nested values inside merged arrays should also merge recursively.

```javascript
const object = {
  items: [{ info: { count: 1, label: 'a' } }],
};

deepSet(object, 'items', [{ info: { count: 2 } }]);

// {
//   items: [{ info: { count: 2, label: 'a' } }],
// }
```

If the values cannot be merged, overwrite as usual.

```javascript
const object = {
  settings: {
    theme: { mode: 'light' },
  },
};

deepSet(object, 'settings.theme', 'dark');

// {
//   settings: {
//     theme: 'dark',
//   },
// }
```

## Arguments

The API stays the same as the previous question:

`deepSet(obj, path, value)`

- `obj` (`object | Array<unknown>`): The object to mutate.
- `path` (`string | Array<string | number>`): The non-empty path where the value should be written. Array indices may be non-negative integers or canonical numeric strings such as `'0'` and `'12'`.
- `value` (`unknown`): The value to write or merge at the target path.

## Notes

- This question only changes the behavior of the final write.
- Paths are guaranteed to be non-empty.
- Array merging is by index, not concatenation.
- This is still an interview-sized subset:
  - Bracket syntax is out of scope.
  - Numeric strings with leading zeroes, such as `'01'`, are out of scope.
  - Immutable writes are out of scope.
  - Special object types and cycle handling are out of scope.

## Hints

### Hint 1 : Where does part II diverge from part I?

### Hint 2 : What should drive a recursive leaf merge?

## 🤔 Thought Process

- **Immediate Recognition:** Advanced variant of `deepSet` that performs recursive merging at the leaf instead of simple overwriting.
- **Core Problem:**
  - Intermediate traversal: Exactly like `deepSet` (auto-vivify `{}` or `[]` along path).
  - Destination leaf: When arriving at `current[targetKey]`, instead of unconditionally overwriting, apply merge rules:
    - If `target` and `value` are both plain objects: Recursively merge by key.
    - If `target` and `value` are both arrays: Recursively merge by index.
    - Otherwise: Overwrite target with `value`.
- **Recursive Leaf Merging Helper:**
  - `mergeValues(target, source)`:
    - If both plain objects: For each key in `source`, `target[k] = mergeValues(target[k], source[k])`.
    - If both arrays: For each index in `source`, `target[i] = mergeValues(target[i], source[i])`.
    - Otherwise: Return `source`.

---

## 🧠 Mental Model

Think of **Git Merge at a Target Subdirectory**:
- Navigate down the directory tree to the target location (`path`).
- Instead of wiping out the destination folder with the new files, perform a recursive three-way / union merge:
  - If both are folders (objects), merge file-by-file.
  - If both are lists (arrays), merge element-by-element by index.
  - If types conflict (folder vs file), overwrite with the incoming version.

---

## 🔑 Key Concepts

- In-place mutation with recursive leaf merging
- Compatible container matching (`isPlainObject` vs `Array.isArray`)
- Merge by index vs merge by key
- Path navigation and cursor tracking

---

## ⚠️ Edge Cases / Traps

- **Type Incompatibility at Leaf:** If the existing leaf is an `Array` and incoming value is an `Object` (or vice versa), do NOT attempt merging; overwrite completely.
- **Array Index Merging:** In an array merge, if existing array is `[1, 2, 3]` and new value is `[9]`, the merged result is `[9, 2, 3]` (index 0 merged, remaining indices preserved).
- **Nested Structures inside Leaf:** Merging rules must apply recursively throughout any nested objects/arrays inside the incoming value.
- **Unset Target Property:** If `targetKey` is not yet set on parent, assign directly without attempting merge.

---

## ⭐ Interview Takeaway

- Decouple navigation from merging:
  1. Use `deepSet` navigation loop to reach the immediate parent of the target key.
  2. If target property already exists, call recursive `deepMerge(current[lastKey], value)`.
  3. Otherwise, directly assign `current[lastKey] = value`.
- Array merge rule in this problem is **by index**, not concatenation or append.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How does Deep Set II differ from Deep Set I?" (Deep Set I overwrites the leaf value; Deep Set II recursively merges compatible object/array containers).
- "How are arrays merged under these rules?" (Element-by-element by index, preserving non-overlapping elements from the target array).

### Follow-up Questions
- "How would you support array append or union merging instead of index-replacement?" (Allow an options object specifying array merge strategies: `concat`, `union`, or `replace`).
- "What if the incoming leaf has circular references?"

### Conceptual Questions
- "How does this compare to `lodash.merge` vs `Object.assign`?" (`Object.assign` is shallow; `lodash.merge` behaves identically to this recursive leaf merge).

---

## 🔄 Variations

- **Deep Set I:** Pure overwrite without merging.
- **Lodash Merge:** Full deep merge of two root objects.
- **Immutable Deep Merge:** Performing structural sharing merge without in-place mutation.

---

## 📝 Revision Notes

- Reference implementation:
```javascript
function isPlainObject(val) {
  if (val === null || typeof val !== 'object') return false;
  const proto = Object.getPrototypeOf(val);
  return proto === null || proto === Object.prototype;
}

function deepMerge(target, source) {
  if (isPlainObject(target) && isPlainObject(source)) {
    for (const key of Object.keys(source)) {
      if (key in target) {
        target[key] = deepMerge(target[key], source[key]);
      } else {
        target[key] = source[key];
      }
    }
    return target;
  }

  if (Array.isArray(target) && Array.isArray(source)) {
    for (let i = 0; i < source.length; i++) {
      if (i in source) {
        target[i] = i in target ? deepMerge(target[i], source[i]) : source[i];
      }
    }
    return target;
  }

  return source;
}

export default function deepSet(obj, path, value) {
  if (obj === null || typeof obj !== 'object') return obj;

  const keys = Array.isArray(path) ? path : path.split('.');
  let current = obj;

  for (let i = 0; i < keys.length - 1; i++) {
    const key = keys[i];
    const nextKey = keys[i + 1];

    if (!(key in current) || current[key] === null || typeof current[key] !== 'object') {
      const isNextNumeric = !isNaN(Number(nextKey)) && !isNaN(parseInt(nextKey, 10));
      current[key] = isNextNumeric ? [] : {};
    }

    current = current[key];
  }

  const lastKey = keys[keys.length - 1];
  if (lastKey in current && current[lastKey] !== null && typeof current[lastKey] === 'object') {
    current[lastKey] = deepMerge(current[lastKey], value);
  } else {
    current[lastKey] = value;
  }

  return obj;
}
```

---

## Official Solution

## Deep Set II ( Official solution )

Premium
Languages
Part II reuses the same path traversal as `Deep Set`. The only new idea is what happens at the final segment: instead of always overwriting the existing value, merge arrays by index and plain objects by key when both sides have compatible container types.

## Solution

Reuse the original path walk and specialize only the leaf write:

1. Normalize the non-empty `path` into segments and walk until the parent of the last segment.
2. Create missing intermediate containers the same way as part I.
3. At the leaf, call a `mergeLeaf(existingValue, nextValue)` helper instead of assigning directly.

The traversal remains mutable. Missing or primitive intermediate branches may still be replaced with new containers so the function can reach the target leaf.

Keeping traversal and leaf merging separate is the design boundary. Numeric path segments still decide whether missing intermediate containers become arrays, while only the final segment gets the recursive merge behavior from this follow-up.

### Leaf merge rules

`mergeLeaf()` has three cases:

- If both values are arrays, merge them index by index.
- If both values are plain objects, merge them key by key.
- Otherwise, return the incoming value so it overwrites the old one.

Because the function is intentionally mutable, the merge helper can update existing arrays and plain objects in place and return the same container. Nested values inside arrays and plain objects use the same `mergeLeaf()` rules recursively.

Array merging is not concatenation. Incoming index `0` updates existing index `0`; incoming index `1` updates existing index `1`; if the incoming array is shorter, untouched trailing items stay as they are.

For an existing leaf of `[{ count: 1, label: 'a' }, { done: false }]` and an incoming value of `[{ count: 2 }]`, only index `0` is visited. That object is compatible, so `count` is replaced, `label` is preserved, and index `1` remains untouched.

Here is the same leaf merge as a recursive trace:

| Existing value | Incoming value | Merge decision | Result |
| --- | --- | --- | --- |
| `items` array with two entries | array with one entry | merge by index | visit index `0`; leave index `1` alone |
| `{ info: { count: 1, label: 'a' } }` | `{ info: { count: 2 } }` | merge by key | visit key `info`; keep other keys |
| `{ count: 1, label: 'a' }` | `{ count: 2 }` | merge by key | visit key `count`; keep `label` |
| `1` | `2` | incompatible leaves | overwrite with `2` |

```jsx
/**
 * @typedef {string | number} PathSegment
 * @typedef {string | Array<PathSegment>} Path
 * @typedef {Record<string | number, unknown> | Array<unknown>} Container
 */
function normalizePath(path) {
  return Array.isArray(path) ? path : path.split('.');
}

function isContainer(value) {
  return value !== null && typeof value === 'object';
}

function isPlainObject(value) {
  if (!isContainer(value) || Array.isArray(value)) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
}

function isArrayIndex(segment) {
  if (typeof segment === 'number') {
    return Number.isInteger(segment) && segment >= 0;
  }

  return /^\d+$/.test(segment);
}

function createContainer(nextSegment) {
  return isArrayIndex(nextSegment) ? [] : {};
}

function mergeLeaf(existingValue, nextValue) {
  if (Array.isArray(existingValue) && Array.isArray(nextValue)) {
    // Merge arrays by index so shorter updates leave later items untouched.
    for (let i = 0; i < nextValue.length; i += 1) {
      existingValue[i] = mergeLeaf(existingValue[i], nextValue[i]);
    }

    return existingValue;
  }

  if (isPlainObject(existingValue) && isPlainObject(nextValue)) {
    // Only plain objects are merged recursively; other object types are replaced.
    for (const key of Object.keys(nextValue)) {
      existingValue[key] = mergeLeaf(existingValue[key], nextValue[key]);
    }

    return existingValue;
  }

  return nextValue;
}

/**
 * @param {Container} obj
 * @param {Path} path
 * @param {unknown} value
 * @returns {void}
 */
export default function deepSet(obj, path, value) {
  const segments = normalizePath(path);
  let current = obj;

  for (let i = 0; i < segments.length - 1; i += 1) {
    const segment = segments[i];
    const nextSegment = segments[i + 1];
    const existing = current[segment];

    if (!isContainer(existing)) {
      // Build missing containers exactly like the basic deep-set version.
      current[segment] = createContainer(nextSegment);
    }

    current = current[segment];
  }

  const lastSegment = segments[segments.length - 1];
  // Only the leaf gets merged; the path creation step above just ensures it is reachable.
  current[lastSegment] = mergeLeaf(current[lastSegment], value);
}
```

## Common pitfalls

- Merging every object encountered while walking the path. Only the final path segment uses the merge rules.
- Concatenating arrays instead of merging them by index.
- Treating special object types as mergeable. Only plain objects merge recursively; other object types are replaced when types do not match the supported cases.
- Returning a newly rebuilt root object. Like part I, this function mutates the provided object.
- Forgetting that incoming arrays can be longer than existing arrays. Extra incoming indices are still assigned because the merge loop visits every incoming index.
- Changing the path-walk behavior from part I. Array paths and numeric path segments still need to create the same missing containers before the leaf merge happens.

## Notes

- The key distinction from a generic deep merge problem is that only the value at the final path segment is merged.
- Paths are guaranteed to be non-empty.
- Array-by-index merging is different from concatenation: incoming positions replace or merge into existing positions, while untouched trailing items stay as they are.
- Plain object and array merges are recursive at the leaf, so nested compatible values can merge without changing the traversal behavior.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A form uses `deepSet(state, 'tags', [])` to clear an existing non-empty tags array. What follows from the leaf merge contract?
