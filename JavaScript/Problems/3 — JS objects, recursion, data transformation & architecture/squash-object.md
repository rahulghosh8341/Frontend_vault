---
title: "Squash Object"
aliases:
  - "squashObject"
  - "Squash Object"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Squash Object

> [!info] Problem
> Implement a function that returns a new object after squashing the input object into a single level of depth

## Problem

## Squash Object

Zhenghao He
Engineering Manager, Robinhood
Implement a function that returns a new object with the input object squashed to a single level, where nested keys are joined with a period delimiter (`.`).

## Examples

```javascript
const object = {
  a: 5,
  b: 6,
  c: {
    f: 9,
    g: {
      m: 17,
      n: 3,
    },
  },
};

squashObject(object); // { a: 5, b: 6, 'c.f': 9, 'c.g.m': 17, 'c.g.n': 3 }
```

Any keys with nullish values (`null` and `undefined`) are still included in the returned object.

```javascript
const object = {
  a: { b: null, c: undefined },
};
squashObject(object); // { 'a.b': null, 'a.c': undefined }
```

It should also work with properties that have arrays as the value:

```javascript
const object = { a: { b: [1, 2, 3], c: ['foo'] } };
squashObject(object); // { 'a.b.0': 1, 'a.b.1': 2, 'a.b.2': 3, 'a.c.0': 'foo' }
```

Empty keys should be treated as if that "layer" does not exist.

```javascript
const object = {
  foo: {
    '': { '': 1, bar: 2 },
  },
};
squashObject(object); // { foo: 1, 'foo.bar': 2 }
```

## Hints

### Hint 1 : What path reaches each leaf?

### Hint 2 : Does an empty key erase its descendants?

## 🤔 Thought Process

- **Immediate Recognition:** Deep object flattening / path collapse utility (inverse of `unsquashObject`).
- **Core Requirements:**
  - Returns a new single-level object where nested keys are joined by period (`.`).
  - Arrays are flattened using their numeric indices as path segments (`'a.b.0'`, `'a.b.1'`).
  - Nullish values (`null` and `undefined`) are preserved as terminal values.
  - Special Empty Key Rule:
    - Empty keys `""` in intermediate or leaf positions should be treated as if that "layer" does not exist!
    - E.g. `{ foo: { '': { '': 1, bar: 2 } } }` squashes to `{ foo: 1, 'foo.bar': 2 }`.
  - Empty objects / empty arrays:
    - If a property is an empty `{}` or `[]`, preserve `{}` or `[]` as the leaf value.
- **Recursive Strategy:**
  - Maintain an accumulated `prefix` string or array of path segments.
  - Base case: Primitives, `null`, `undefined`, functions, empty `{}` / `[]`.
    - Key joining logic: Filter out empty string segments `""` before joining with `'.'`.
    - Store `result[joinedKey] = val`.
  - Recursive case:
    - Non-empty plain objects and arrays: Iterate over keys/indices and recurse with new path segment.

---

## 🧠 Mental Model

Think of **Flattening a Directory Hierarchy into Full File Paths**:
```
folder/
  subfolder/
    file.txt (10)
    data.json (20)

Flattened Paths:
  "folder.subfolder.file.txt": 10
  "folder.subfolder.data.json": 20
```

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Object Path Traversal]]
- [[Recursion]]
- Tree flattening & path prefix accumulation
- Empty key normalization
- Handling empty containers (`{}` and `[]`)

---

## ⚠️ Edge Cases / Traps

- **Empty Key Stripping:** Keys that are empty strings `""` must not introduce double dots (`'foo..bar'`) or trailing dots. They should collapse as if that nesting layer does not exist.
- **`null` and `undefined`:** `typeof null === 'object'`. Do NOT treat `null` as an object to recurse into; `null` is a leaf!
- **Empty Objects `{}` and Arrays `[]`:** Must not disappear! An empty container `{ a: {} }` must produce `{ a: {} }`.
- **Arrays with Numeric Keys:** Array indices must be converted to strings and appended to the path (`'arr.0'`).

---

## ⭐ Interview Takeaway

- Recursive helper with path accumulator:
  `function recurse(val, path = []) { ... }`
- Key joining helper that skips empty strings:
  `const buildKey = (segments) => segments.filter(Boolean).join('.');`
- Check for empty objects: `Object.keys(val).length === 0` to assign empty `{}` or `[]` directly.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you handle empty strings as object keys during squashing?" (Filter out empty string segments before joining keys with dots).
- "Why must `null` be explicitly checked when determining if a value is an object?" (`typeof null === 'object'`; forgetting `val !== null` causes infinite loop or runtime error).

### Follow-up Questions
- "How would you implement the reverse function `unsquashObject`?" (Split keys by `.` and reconstruct nested objects/arrays).
- "How would you handle cyclic object references?" (Track seen objects with a `WeakSet`).

### Conceptual Questions
- "Where is object squashing used in real applications?" (Converting nested configuration objects into flat environment variable dictionaries or query parameter strings).

---

## 🔄 Variations

- **Unsquash Object:** Expanding dot-notated keys back into nested structures.
- **Deep Map:** Applying a function across all leaves without flattening.
- **Flatten Array:** One-dimensional flattening of nested arrays.

---

## 📝 Revision Notes

- Production implementation:
```javascript
export default function squashObject(obj) {
  const result = {};

  function traverse(current, path) {
    if (
      current === null ||
      typeof current !== 'object' ||
      (Array.isArray(current) && current.length === 0) ||
      (!Array.isArray(current) && Object.keys(current).length === 0)
    ) {
      const key = path.filter(Boolean).join('.');
      result[key] = current;
      return;
    }

    if (Array.isArray(current)) {
      current.forEach((item, index) => {
        traverse(item, [...path, String(index)]);
      });
    } else {
      for (const [k, v] of Object.entries(current)) {
        traverse(v, [...path, k]);
      }
    }
  }

  traverse(obj, []);
  return result;
}
```

---

## Official Solution

## Squash Object ( Official solution )

Zhenghao He
Engineering Manager, Robinhood
Languages
This is a tricky recursion question because traversal is only half the work. The solution also has to change the shape of the object by gluing the keys on the current path into one flattened key when traversal reaches a leaf value.

The frame is:

- `path` stores the keys from the root to the current value
- object-like values keep the recursion going
- non-object values and `null` become entries in the output object

## Solution

There are generally two ways to traverse an object:

1. Loop through the keys with the `for ... in` statement.
2. Convert the object into an array of keys with `Object.keys()`, or an array of key-value pairs with `Object.entries()`.

With the `for ... in` statement, inherited enumerable properties are processed as well. Normally, add an `Object.hasOwn()` check to make sure the property is not inherited from its prototype. On the other hand, `Object.keys()` and `Object.entries()` only care about the properties directly defined on the object, and this is usually the intended behavior.

Here is how to visit each property. When the value of a given property is an object, repeat the process recursively.

```javascript
function squashObject(object) {
  for (const [key, value] of Object.entries(object)) {
    if (typeof value !== 'object' || value === null) {
      // Add props with glued/squashed keys.
    } else {
      // Recursion by calling squashObject.
    }
  }
}
```

Track the keys on the path to the current value so they can be squashed into the output object's new keys. Pass the keys down to the recursive call to do that.

The helper receives three pieces of state:

1. The current object being traversed.
2. `path`, an array of keys explored so far.
3. `output`, the object that receives flattened key-value pairs.

When the current value is a leaf, append the current key to `path`, remove empty key segments, join the remaining keys with `.`, and assign the value into `output`.

The solution below defines an inner recursive helper function that accepts the `path` and `output` parameters.

For `{ foo: { '': { bar: 2 } } }`, the path evolves like this:

| Current key | Raw path before key | Path used for child/leaf | Output action |
| --- | --- | --- | --- |
| `foo` | `[]` | `['foo']` | recurse |
| `''` | `['foo']` | `['foo', '']` | recurse |
| `bar` | `['foo', '']` | `['foo', '', 'bar']` | write key `'foo.bar'` |

The code filters empty path segments only when writing the flattened key, so empty-key layers disappear without preventing traversal through their children.

```jsx
/**
 * @param {Object} obj
 * @return {Object}
 */
export default function squashObject(obj) {
  function squashImpl(obj_, path, output) {
    for (const [key, value] of Object.entries(obj_)) {
      if (typeof value !== 'object' || value === null) {
        // Build the dotted key from the recursion path; empty segments are skipped.
        output[path.concat(key).filter(Boolean).join('.')] = value;
      } else {
        squashImpl(value, path.concat(key), output);
      }
    }
  }

  const out = {};
  squashImpl(obj, [], out);
  return out;
}
```

Every recursive call owns one path prefix. Children receive a longer path, while sibling branches continue from the same prefix they started with.

### Alternative approach

This question asks for a new object based on the current object but with a different shape. The previous solution does that by recursively passing down the `output` object and assigning the new key directly to the output object when the value is a primitive.

Another technique for processing objects is to convert the object into an array of key-value tuples with `Object.entries`, transform those tuples with array methods such as `Array.prototype.map`, and then convert the result back to an object with `Object.fromEntries`.

Suppose the input is:

```javascript
const object = {
  a: 5,
  c: {
    f: 9,
  },
};
```

`Object.entries(object)` would produce `[['a', 5], ['c', { f: 9 }]]`. To get the object with squashed keys, for example `{ a: 5, 'c.f': 9 }`, transform the array `[['a', 5], ['c', { f: 9 }]]` into `[['a', 5], ['c.f', 9]]` and pass it to `Object.fromEntries`.

Here is a second solution that may be easier to understand than the previous solution.

```javascript
function chunk(array, size = 2) {
  // Helper function that groups two adjacent items in an array into one subarray.
  const chunkedArray = [];
  while (array.length) {
    chunkedArray.push(array.splice(0, size));
  }
  return chunkedArray;
}

function traverse(object, path = []) {
  if (typeof object !== 'object' || object === null) {
    return [path.join('.'), object];
  }

  return Object.entries(object).flatMap(([key, value]) => {
    const newPath = key === '' ? [...path] : [...path, key];
    return traverse(value, newPath);
  });
}

export default function squashObject(object) {
  const flattened = traverse(object);
  return Object.fromEntries(chunk(flattened));
}
```

## Common pitfalls

- The input has to be an object, not a primitive value.
- Forgetting to carry the path through recursion, which makes it impossible to reconstruct the flattened key at a leaf.
- Joining the path before adding the current key, which drops the leaf key from the output.
- Using `for ... in` without guarding against inherited properties.
- Recursing into `null`.
- Expecting this implementation to preserve multiple values that flatten to the same key.

## Notes

- Symbol-keyed properties and non-enumerable properties are ignored.
- Cyclic objects are not handled properly.
- Conflicting keys with different values. JavaScript keys can contain `.` too, and existing keys that contain `.` may conflict with a resulting key. For example, `{ a: { b: 1 }, 'a.b': 2 }`.
  - Point this case out during interviews, but it usually does not need to be handled.

- Keys that are empty strings are skipped when building the dotted path.
- Arrays are object-like and can be traversed through their enumerable indexes, but the main scope is plain objects.
- Built-ins such as `Date`, `Map`, `Set`, and `RegExp` are not meaningfully flattened by this object-entry traversal.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A traversal uses one shared `path` array and pushes each key before recursing, but never pops it afterward. It passes tests containing a single nested leaf.

Give a small branching input that exposes the bug. Explain the condition the path must satisfy before each sibling is visited.

Your notes (optional)
