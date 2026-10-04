---
title: Unsquash Object
aliases:
  - Unsquash Object
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/unsquash-object"
pattern:
  - "[[Object Path Traversal]]"
concepts:
  - "[[Object Path Traversal]]"
  - "[[Recursion]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Unsquash Object

> [!info] Problem
> Implement a function that reconstructs a nested object from dot-delimited keys.

## Problem

## Unsquash Object

Implement a function that returns a new object after unsquashing a flat object where keys use period delimiters (`.`) to represent nesting.

This is the reverse operation of the **Squash Object** question.

## Examples

```javascript
const object = {
  a: 5,
  b: 6,
  'c.f': 9,
  'c.g.m': 17,
  'c.g.n': 3,
};

unsquashObject(object);
// {
//   a: 5,
//   b: 6,
//   c: {
//     f: 9,
//     g: {
//       m: 17,
//       n: 3,
//     },
//   },
// }
```

Any keys with nullish values (`null` and `undefined`) are still included in the returned object.

```javascript
const object = {
  'a.b': null,
  'a.c': undefined,
};

unsquashObject(object); // { a: { b: null, c: undefined } }
```

It should also work with properties that represent arrays:

```javascript
const object = {
  'a.b.0': 1,
  'a.b.1': 2,
  'a.b.2': 3,
  'a.c.0': 'foo',
};

unsquashObject(object); // { a: { b: [1, 2, 3], c: ['foo'] } }
```

Empty path segments should be treated as if that layer does not exist.

A key containing only empty segments, such as `''` or `'...'`, should be ignored.

```javascript
const object = {
  'foo..bar': 2,
};

unsquashObject(object); // { foo: { bar: 2 } }
```

## Hints

### Hint 1 : What does the next segment tell you?

### Hint 2 : How do shared paths meet?

## 🤔 Thought Process

- **Immediate Recognition:** Reverse operation of `squashObject` / object path expansion (Lodash `_.set` logic).
- **Core Problem:** Taking flat key-value pairs where keys contain dot-delimited paths (e.g. `'a.b.0': 1`) and reconstructing the nested hierarchy of objects and arrays.
- **Path Processing:**
  - Split key on `.` into segments.
  - Filter out empty segments (e.g. `'foo..bar'` -> `['foo', 'bar']`). Ignore keys with only empty segments (`'...'`).
- **Container Type Selection (Object vs Array):**
  - When traversing or creating intermediate containers, look ahead at the *next* segment.
  - If the next segment is a valid numeric string (e.g. `'0'`, `'1'`), initialize an array `[]`.
  - Otherwise, initialize a plain object `{}`.
- **Handling Nullish Values:**
  - Values that are `null` or `undefined` must be explicitly assigned, not skipped.

---

## 🧠 Mental Model

Think of **Building a Nested Folder Directory from File Paths**:
- You are given a list of full paths: `a/b/0.txt`, `a/b/1.txt`, `c/f.txt`.
- Start at the root directory `{}`.
- Step through each directory in the path:
  - Does the folder exist? If yes, enter it.
  - If not, check what kind of container to build: if the next token is an integer, build an array drawer; otherwise build a folder object.
- Place the final payload in the destination slot.

---

## 🔑 Key Concepts

- [[Object Path Traversal]]
- Dynamic property creation and deep object reconstruction
- Integer key heuristic for array instantiation (`String(Number(key)) === key`)
- Path tokenization and filtering empty segments

---

## ⚠️ Edge Cases / Traps

- **Consecutive Dots (`'foo..bar'`):** Empty segments created by splitting `..` must be ignored, treating the path as `foo.bar`.
- **Pure Empty / Dot Keys (`''`, `'...'`):** Keys that yield zero valid segments must be discarded completely without modifying the root object.
- **Preserving `null` and `undefined`:** Keys like `'a.b': null` must produce `{ a: { b: null } }`. Do not use truthiness checks (`if (!val)`) that drop nullish values.
- **Array vs Object Type Conflicts:** If multiple paths share prefixes, existing containers must be preserved rather than overwritten.
- **Numeric Property Detection:** Checking whether a segment represents an array index requires validating positive integer strings without negative signs or non-numeric characters.

---

## ⭐ Interview Takeaway

1. **Lookahead for Container Selection:** When building a missing level at segment `i`, look ahead to segment `i + 1`: if `/^\d+$/.test(segments[i + 1])`, create `[]`, else `{}`.
2. **Two-Pointer / Traversal Cursor:** Maintain a mutable `cursor` pointer pointing to the current nesting depth, updating `cursor = cursor[segment]` at each step.
3. **Filter Before Traversal:** Sanitizing `key.split('.').filter(Boolean)` simplifies traversal by eliminating empty segment checks inside the loop.

---

## 🎯 Common Interview Questions

### Direct Questions
- How do you determine whether an intermediate container should be an Array or an Object?
- Why must empty segments like `'foo..bar'` be filtered out?
- What happens if the input has conflicting paths like `'a': 1` and `'a.b': 2`?

### Follow-up Questions
- How would you handle escaped dots (e.g. `'a\\.b.c'` where `'a.b'` is a literal single property name)?
- How would you implement the reverse function `squashObject`?
- How does Lodash's `_.set` differ from `unsquashObject` when setting sparse array indices?

### Conceptual Questions
- How are numeric object keys handled in JavaScript V8 engine (elements vs properties)?
- Why are array indices in JavaScript technically string properties?

---

## 🔄 Variations

- **Squash Object:** The inverse operation flattening nested structures into dot-separated paths.
- **Deep Set / Lodash `_.set`:** Setting a single deep path on an existing object.
- **Bracket Notation Path Parsing:** Supporting paths like `'a[0].b.c'`.

---

## 📝 Revision Notes

- **Core idea:** Split path on `.`, filter out empty segments, look ahead to decide between `{}` and `[]`, traverse with a cursor.
- **Remember:** If next segment is a numeric integer string, initialize container as `[]`, else `{}`.
- **Watch out for:** Preserve explicit `null` and `undefined` values; discard keys containing only dots.
- **Complexity:** Time: $O(K \times L)$ where $K$ is number of keys and $L$ is max path length; Space: $O(N)$ total reconstructed nodes.

## Official Solution
## Unsquash Object ( Official solution )

Premium
Languages
This problem is the reverse of **Squash Object**. Instead of traversing nested structures and joining path segments, the input contains joined path segments, and the solution rebuilds the nested structure in a fresh output object.

## Solution

Walk each path: split every flattened key into segments, then rebuild the missing containers on the way down.

For each key-value pair in the input object:

1. Split the key by `.` to get path segments.
2. Ignore empty segments (so `foo..bar` behaves like `foo.bar`).
3. Walk the output object and create containers (`{}` or `[]`) as needed.
4. Assign the value at the final segment.

The traversal condition is the same as in a deep setter: `current` points at the container that should receive the next segment. Existing object or array containers are reused. Missing, `null`, or primitive branches are replaced so the walk can continue.

### Creating containers

The only extra detail is deciding whether a new container should be an object or an array. Choose from the next segment:

- If the *next* segment is numeric (`"0"`, `"1"`, ...), initialize with `[]`.
- Otherwise initialize with `{}`.

This lets a path like `'a.b.0.foo'` create `a` as an object, `b` as an array, and index `0` as an object before assigning `foo`.

Trace for `'b.e.f.0.foo': 123`:

| Current segment | Next segment | Missing container created |
| --- | --- | --- |
| `b` | `e` | `{}` |
| `e` | `f` | `{}` |
| `f` | `0` | `[]` |
| `0` | `foo` | `{}` |
| `foo` | none | assign `123` |

The decision uses the next segment because the current segment names the slot, while the next segment indicates what kind of container has to live in that slot.

### Ambiguous paths

Conflicting flattened keys can describe incompatible structures, such as both `a` and `a.b`. This implementation follows JavaScript object insertion order while processing entries. Later assignments may overwrite earlier primitive values or structures when the path requires a different container shape.

```jsx
function isArrayIndex(segment) {
  return /^\d+$/.test(segment);
}

/**
 * @param {Object} obj
 * @return {Object}
 */
export default function unsquashObject(obj) {
  const output = {};

  for (const [rawKey, value] of Object.entries(obj)) {
    const path = rawKey.split('.').filter(Boolean);

    if (path.length === 0) {
      continue;
    }

    let current = output;

    for (let i = 0; i < path.length - 1; i += 1) {
      const segment = path[i];
      const nextSegment = path[i + 1];

      // The upcoming path segment indicates whether this missing container
      // should behave like an object branch or an array slot holder.
      if (current[segment] == null || typeof current[segment] !== 'object') {
        current[segment] = isArrayIndex(nextSegment) ? [] : {};
      }

      current = current[segment];
    }

    current[path[path.length - 1]] = value;
  }

  return output;
}
```

## Common pitfalls

- Creating the container from the current segment instead of the next segment.
- Treating empty path segments as real keys. The code filters them out, so `foo..bar` behaves like `foo.bar`.
- Dropping `null` or `undefined` values. They are valid values at the final segment.
- Expecting conflicting paths to have one universally correct result. The input order decides which assignment wins.

## Notes

- An empty input object should return an empty object.
- Keys with no delimiters should stay at the top level.
- Empty path segments, such as `foo..bar`, should be ignored.
- Nullish values (`null` and `undefined`) should be preserved.
- Array-like paths (`a.0`, `a.1`) should produce arrays.
- Conflicting keys (for example both `a` and `a.b`) are ambiguous. In JavaScript object insertion order, later assignments can overwrite earlier structures.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A reconstruction allocates a new container every time it sees a non-final path segment. It can build `'team.members.0.name'`, but processing `'team.members.0.role'` erases the name.

Why does looking ahead to the next segment not solve this bug by itself? State when an existing container should be reused.

Your notes (optional)
