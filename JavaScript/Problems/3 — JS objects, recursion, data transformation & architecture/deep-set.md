---
title: "Deep Set"
aliases:
  - "deepSet"
  - "Deep Set"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Set

> [!info] Problem
> Implement a function to write a value to a deeply nested path within an object

## Problem

## Deep Set

[`dset`](https://github.com/lukeed/dset) is a tiny utility for writing values into deeply nested objects and arrays.

Implement a simplified `deepSet(obj, path, value)` function.

The function should mutate `obj` in place by writing `value` at `path`. If part of the path does not exist yet, create the missing objects or arrays along the way.

## Examples

Write into nested objects.

```javascript
const object = { user: { profile: { name: 'John' } } };

deepSet(object, 'user.profile.age', 30);

// {
//   user: {
//     profile: {
//       name: 'John',
//       age: 30,
//     },
//   },
// }
```

Write into arrays using numeric path segments.

```javascript
const object = { items: ['a', 'b', 'c'] };

deepSet(object, 'items.1', 'updated');

// { items: ['a', 'updated', 'c'] }
```

Create mixed object/array structures when needed.

```javascript
const object = {};

deepSet(object, 'a.0.b.1', 2);

// {
//   a: [
//     {
//       b: [undefined, 2],
//     },
//   ],
// }
```

If traversal needs to continue through a primitive value, replace that branch with a new container.

```javascript
const object = { config: true };

deepSet(object, 'config.theme.name', 'dark');

// {
//   config: {
//     theme: {
//       name: 'dark',
//     },
//   },
// }
```

## Arguments

`deepSet(obj, path, value)`

- `obj` (`object | Array<unknown>`): The object to mutate.
- `path` (`string | Array<string | number>`): The path where the value should be written.
  - A string path uses `.` as the separator, such as `'user.profile.name'`.
  - An array path can contain strings and numbers, such as `['user', 'profile', 'name']` or `['items', 0, 'label']`.

- `value` (`unknown`): The value to write at the target path.

## Notes

- Missing containers should be created automatically.
- Use an array when the next path segment is a numeric index. Otherwise, use an object.
- This question is intentionally limited:
  - Bracket syntax like `a[0].b` is out of scope.
  - The function should mutate the input object instead of returning a new one.
  - Production-hardening details like prototype-pollution protection are out of scope.

## Follow-up

Implement [Deep Set II](/questions/javascript/deep-set-ii) to merge object and array values at the leaf instead of always overwriting them.

## Hints

### Hint 1 : Where should traversal stop?

### Hint 2 : How do you choose a missing container?

## 🤔 Thought Process

- **Immediate Recognition:** In-place deep property setter with auto-vivification (similar to `lodash.set` or `dset`).
- **Core Requirements:**
  - Mutates `obj` in place.
  - Takes path as dot-separated string (`'a.b.c'`) or array of segment keys (`['a', 'b', 'c']`).
  - Writes `value` at the destination leaf.
  - Auto-vivification: If intermediate segments do not exist, create missing containers (`{}` or `[]`).
- **Container Selection Rule:**
  - Inspect the *next* segment in the path:
    - If the next segment is a non-negative integer / numeric string, create an `Array` (`[]`).
    - Otherwise, create an `Object` (`{}`).
- **Traversal Strategy:**
  - Normalize path into array of keys.
  - Loop from `0` to `keys.length - 2` with cursor `current = obj`.
  - If `current[key]` is not an object/array, replace it with `isNaN(keys[i + 1]) ? {} : []`.
  - Step cursor: `current = current[key]`.
  - At final key (`keys.length - 1`), assign `current[finalKey] = value`.
  - Return `obj`.

---

## 🧠 Mental Model

Think of **Filesystem Path Navigation with `mkdir -p`**:
- You are writing a file to `/var/log/app/output.json`.
- If `/var/log` or `/app` doesn't exist, create the directory first before touching the file.
- Lookahead determines folder type: if the next child looks like an index number, create an indexed list; otherwise create a directory dictionary.

---

## 🔑 Key Concepts

- Path normalization (splitting dot notation vs array paths)
- Auto-vivification (automatic creation of intermediate data structures)
- In-place object mutation
- Integer index detection (`String(Number(segment)) === segment` or regex `/^\d+$/`)

---

## ⚠️ Edge Cases / Traps

- **Replacing Primitives:** If intermediate property already exists as a primitive (e.g. `obj = { a: 'hello' }` and path is `'a.b'`), the string `'hello'` must be overwritten with an object/array.
- **Prototype Pollution:** Setting keys like `'__proto__'`, `'constructor'`, or `'prototype'` can corrupt the global prototype. Security-conscious implementations guard against these keys.
- **Numeric vs String Keys:** A path like `'items.0'` requires `items` to be an array, not a dictionary with key `'0'`.
- **Array Path Input:** Paths can be supplied as `['a', 'b', 'c']` or `'a.b.c'`. Normalize array vs string input upfront.

---

## ⭐ Interview Takeaway

- Normalize path first: `const segments = Array.isArray(path) ? path : path.split('.');`
- Always check the *next* segment (`segments[i + 1]`) to decide whether to auto-vivify an `Array` or an `Object`.
- Remember that `deepSet` mutates `obj` in place and returns `obj`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you determine if a path segment should create an Array or an Object?" (Check if the subsequent segment is a numeric integer string).
- "Why does `deepSet` mutate the object in place instead of returning a new one?" (Optimized for performance and memory when updating deep state without framework immutability requirements).

### Follow-up Questions
- "What is Prototype Pollution, and how can an attacker exploit `deepSet`?" (Passing `'__proto__.admin = true'` injects properties into all JS objects; prevent by banning dangerous keys).
- "How would you implement the immutable counterpart that returns a new object?" (Deep Set II / structural sharing).

### Conceptual Questions
- "How does Lodash's `_.set` handle bracket notation like `'a[0].b.c'`?" (Regex tokenizer parses brackets into distinct path keys).

---

## 🔄 Variations

- **Deep Set II:** Deep merging values at the leaf instead of overwriting.
- **Deep Get (`lodash.get`):** Safely reading nested properties without throwing `TypeError`.
- **Immutable Set (`Immer` / Structural Sharing):** Returning new references along the modified path while preserving untouched branches.

---

## 📝 Revision Notes

- Standard implementation:
```javascript
export default function deepSet(obj, path, value) {
  if (obj === null || typeof obj !== 'object') return obj;

  const keys = Array.isArray(path) ? path : path.split('.');
  let current = obj;

  for (let i = 0; i < keys.length - 1; i++) {
    const key = keys[i];
    const nextKey = keys[i + 1];

    if (
      !(key in current) ||
      current[key] === null ||
      typeof current[key] !== 'object'
    ) {
      const isNextNumeric = !isNaN(Number(nextKey)) && !isNaN(parseInt(nextKey, 10));
      current[key] = isNextNumeric ? [] : {};
    }

    current = current[key];
  }

  current[keys[keys.length - 1]] = value;
  return obj;
}
```

---

## Official Solution

## Deep Set ( Official solution )

Languages
Focus on mutable path traversal: walk to the parent of the target key, create any missing containers along the way, then write the value in place.

## Solution

Treat the path as a list of segments and walk the object until traversal reaches the parent of the final key. During the walk, `current` always points at the container targeted by the next write.

Start by normalizing the path. A string like `'a.0.b'` becomes a list of segments, and array paths can already contain strings or numbers. Once the path is normalized, the traversal logic is the same for both forms.

### Walking the path

At each step:

- Read the current child value.
- If it is already a non-null object, keep traversing into it.
- Otherwise, create a new container based on the next segment:
  - If the next segment is numeric, create an array.
  - Otherwise create an object.

Peeking at the next segment is the important interview detail. That is how the traversal knows whether a missing branch should become `[]` or `{}`.

### Writing the leaf

Once traversal reaches the parent container, assign the final segment to `value`.

This function intentionally mutates the original object instead of rebuilding a new structure. Replacing missing or primitive intermediate branches keeps the condition simple: after each loop iteration, `current` is a container and traversal can continue.

For `deepSet(object, 'a.0.b.1', 2)`, container creation is driven by the next segment:

| Current segment | Next segment | Container created at current segment |
| --- | --- | --- |
| `a` | `0` | `[]` because the next segment is numeric |
| `0` | `b` | `{}` because the next segment is a property name |
| `b` | `1` | `[]` because the next segment is numeric |
| `1` | N/A | Write the final value `2` |

Existing containers are reused instead of replaced. If `items` is already an array, the traversal writes into that array; if `config` is `true` and the path continues through `config.theme`, the primitive is replaced because it cannot hold the next segment.

```jsx
function normalizePath(path) {
  return Array.isArray(path) ? path : path.split('.');
}

function isContainer(value) {
  return value !== null && typeof value === 'object';
}

function isArrayIndex(segment) {
  if (typeof segment === 'number') {
    return Number.isInteger(segment) && segment >= 0;
  }

  return /^\d+$/.test(segment);
}

function createContainer(nextSegment) {
  // Look ahead to decide whether the missing branch should be an array or object.
  return isArrayIndex(nextSegment) ? [] : {};
}

/**
 * @param {Record<string | number, unknown> | Array<unknown>} obj
 * @param {string | Array<string | number>} path
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
      // Overwrite missing or primitive branches so traversal can keep going.
      current[segment] = createContainer(nextSegment);
    }

    current = current[segment];
  }

  // The last segment receives the final value without any extra merging logic.
  current[segments[segments.length - 1]] = value;
}
```

## Common pitfalls

- Choosing the new container from the current segment instead of the next segment. The next segment indicates whether the child must support array indexing.
- Stopping when an intermediate branch is primitive. The intended behavior is to replace that branch with a new container and continue.
- Rebuilding and returning a new object. This utility mutates `obj` in place.
- Adding leaf merge behavior here. The basic version always overwrites the final segment.

## Notes

- The key interview detail is peeking at the next segment so arrays and objects are created in the right places.
- Replacing primitive intermediate values keeps the traversal logic simple and matches the intended utility behavior.
- Bracket syntax, prototype-pollution hardening, and immutable writes are out of scope for this interview-sized version.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
After `deepSet(state, 'rows.0.name', 'Ada')` on `{}`, a test checks only `state.rows[0].name`. That test also passes when `rows` was incorrectly created as `{}`. Which additional assertion detects the wrong container choice?
