---
title: "Object Update"
aliases:
  - "objectUpdate"
  - "Object Update"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Object Update

> [!info] Problem
> Implement a function that updates a nested object path immutably with structural sharing

## Problem

## Object Update

Reducers and state containers often need to update a deeply nested field without mutating the existing state. Writing the full object-spread chain by hand works, but it quickly becomes noisy.

An easy way to update an object while leaving the original untouched would be to [deeply clone](/questions/javascript/deep-clone) it, but that uses much more memory because each update creates an entirely new object, which is wasteful when most values in the original object usually stay the same.

Implement a function `objectUpdate(source, path, value)` that updates a nested object path in an immutable fashion and returns a new object instance while sharing the original inner structure where possible.

This question is intentionally scoped:

- `source` is a plain JavaScript object.
- `path` is a non-empty dot-delimited string such as `'draft.user.name'`.
- Every intermediate object already exists.
- Arrays are out of scope.

Return a new object where the value at `path` has been replaced with `value`.

The important constraint is **structural sharing**:

- Clone only the objects along the touched path.
- Keep all untouched branches as the exact same references.
- Do not mutate the input object.

## Examples

```javascript
const state = {
  draft: {
    user: {
      name: 'Alice',
      role: 'admin',
    },
    meta: {
      saved: false,
    },
  },
  theme: {
    mode: 'dark',
  },
};

const next = objectUpdate(state, 'draft.user.name', 'Bob');

next.draft.user.name; // 'Bob'
next !== state; // true
next.draft !== state.draft; // true
next.draft.user !== state.draft.user; // true
next.draft.meta === state.draft.meta; // true
next.theme === state.theme; // true
```

```javascript
const state = {
  status: 'draft',
  draft: {
    title: 'Object Update',
  },
};

const next = objectUpdate(state, 'status', 'published');

console.log(next); // { status: 'published', draft: { title: 'Object Update' } }
console.log(state.status); // 'draft'
```

## Arguments

`objectUpdate(source, path, value)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `source` | `Object` | The plain object to update. |
| `path` | `string` | A non-empty dot-delimited path pointing to an existing nested property. |
| `value` | `unknown` | The new value to store at the final path segment. |

## Returns

Returns a new object with the updated value written at `path`.

## Notes

- The returned object should reuse untouched nested references from `source`.
- Only the objects on the updated path need to be shallow-cloned.
- You do not need to support updater callbacks, arrays, missing-branch creation, or path validation.

## Follow-up

Implement [Object Update II](/questions/javascript/object-update-ii) to support arrays, updater callbacks, and creating missing object branches.

## Hints

### Hint : Which references lie on the changed path?

## 🤔 Thought Process

- **Immediate Recognition:** Immutable nested update with structural sharing (the foundation of Redux, React state updates, and Immer).
- **Core Requirements:**
  - Updates value at nested path without mutating the original object.
  - Dot-delimited path: e.g. `'a.b.c'`.
  - Structural sharing: Only nodes along the modification path get newly instantiated; all unchanged sibling branches retain their original references.
- **Recursive Strategy:**
  - Base Case (path length 1):
    Return shallow clone of current object with updated key: `{ ...current, [key]: value }`.
  - Recursive Step:
    Return shallow clone of current object where `key` is set to the recursive update of `current[key]`:
    `{ ...current, [head]: objectUpdate(current[head], tail, value) }`.
- **Iterative vs Recursive:**
  - Recursion naturally unwinds from the bottom leaf up, reconstructing parent objects while sharing untouched sibling references.

---

## 🧠 Mental Model

Think of **Git Tree Branching / Persistent Data Structures**:
```
       Root A (new)  ────────┐
      /                      │ (shares untouched right branch)
  Node B (new)               ▼
    /                    Node D (identical reference)
Node C (new val)
```
Only nodes directly on the path from root to leaf are cloned. All off-path sub-trees are reused by reference.

---

## 🔑 Key Concepts

- [[Object Path Traversal]]
- [[Recursion]]
- [[DFS Recursion]]
- Structural Sharing (Persistent Data Structures)
- Referential Equality (`===`) in React change detection
- Immutable state updates vs Deep cloning

---

## ⚠️ Edge Cases / Traps

- **Full Deep Clone Pitfall:** Cloning the entire object is an anti-pattern: it destroys referential identity on untouched sub-trees, causing unnecessary re-renders in React `memo` components.
- **Path of Length 1:** Path like `'a'` must return a new root object `{ ...source, a: value }`.
- **Primitives as Intermediate Nodes:** The problem states intermediate objects exist in this first version, but defensive code should ensure safe traversal.
- **Mutating Source Object:** Never do `source[key] = value`. Always create fresh container `{ ...source }`.

---

## ⭐ Interview Takeaway

- Structural sharing preserves untouched references:
  `copy[key] = update(copy[key])`
- Why structural sharing matters in React:
  `oldState.user === newState.user` evaluates to `true` if only `state.settings` changed, preventing re-rendering of User profile components.
- Clean recursive formulation:
  ```javascript
  export default function objectUpdate(source, path, value) {
    const keys = Array.isArray(path) ? path : path.split('.');
    const [head, ...tail] = keys;
    if (tail.length === 0) {
      return { ...source, [head]: value };
    }
    return {
      ...source,
      [head]: objectUpdate(source[head], tail, value),
    };
  }
  ```

---

## 🎯 Common Interview Questions

### Direct Questions
- "What is structural sharing and why is it preferred over deep cloning in frontend state management?" (Reuses memory and preserves referential equality of unchanged branches, enabling $O(1)$ memoization checks).
- "How does `objectUpdate` compare to the object spread syntax `{ ...state, user: { ...state.user, age: 30 } }`?" (Provides dynamic path access without hardcoded nested spread boilerplate).

### Follow-up Questions
- "How would you handle arrays in the path (e.g. `'users.0.name'`)?" (Answered in Object Update II).
- "How would you support an updater function `(prev) => prev + 1`?" (Answered in Object Update II).

### Conceptual Questions
- "How does Immer use ES6 `Proxy` to automate structural sharing?" (Tracks property writes on proxies and constructs a structurally shared copy only for mutated branches on finalize).

---

## 🔄 Variations

- **Object Update II:** Array support, updater functions, and auto-vivification.
- **Deep Set:** Mutating in-place instead of structural sharing.
- **Deep Clone:** Full duplication without reference sharing.

---

## 📝 Revision Notes

- Recursive pattern:
```javascript
export default function objectUpdate(source, path, value) {
  const keys = typeof path === 'string' ? path.split('.') : path;
  const [head, ...tail] = keys;

  if (tail.length === 0) {
    return { ...source, [head]: value };
  }

  return {
    ...source,
    [head]: objectUpdate(source[head], tail, value),
  };
}
```

---

## Official Solution

## Object Update ( Official solution )

Languages

## Solution

This exercise centers on structural sharing: create new objects only for the touched path, and reuse every other reference. The path is a dot-delimited string, every intermediate object already exists, and arrays are out of scope.

View the update as a chain of shallow clones. Each recursive call receives the current object and the remaining path segments, then returns the new version of that current object.

The update path has four jobs:

1. Split the path into segments by the `.` delimiter.
2. At each level, shallow-clone the current object.
3. If this is the last key, assign the new value.
4. Otherwise recurse into the next nested object and store that cloned child on the current clone.

Each level creates exactly one new object for the path being updated. Untouched siblings are never cloned or rewritten, so they keep pointing at the original values.

For example, updating `draft.user.name` clones `state`, `state.draft`, and `state.draft.user`. It does not clone `state.draft.meta` or `state.theme`, because those branches are not on the path.

Reference trace:

| Path level | Object cloned? | Why |
| --- | --- | --- |
| root `state` | yes | it contains the `draft` child being replaced |
| `state.draft` | yes | it contains the `user` child being replaced |
| `state.draft.user` | yes | it contains the final `name` property |
| `state.draft.meta` | no | sibling branch is untouched |
| `state.theme` | no | sibling branch is untouched |

That trace is the key difference from a deep clone: only ancestors of the edited property receive new references.

```jsx
function updateAtPath(source, keys, value) {
  const [key, ...rest] = keys;
  const nextObject = { ...source };

  // Clone each container on the path so the update stays immutable.
  nextObject[key] =
    rest.length === 0 ? value : updateAtPath(source[key], rest, value);

  return nextObject;
}

/**
 * @param {Record<string, unknown>} source
 * @param {string} path
 * @param {unknown} value
 * @returns {Record<string, unknown>}
 */
export default function objectUpdate(source, path, value) {
  return updateAtPath(source, path.split('.'), value);
}
```

## Common pitfalls

- **Deep-cloning the whole object:** A deep clone keeps the input immutable, but it breaks structural sharing by creating new references for untouched branches. The expected behavior is to clone only the root and the ancestors of the updated property.
- **Mutating the nested object before returning:** Assigning directly into `source[key]` or one of its descendants changes the original input. Always assign into the shallow clone created for the current recursion level.
- **Treating top-level updates as a special mutation case:** A path with one segment still needs to return a new root object. The recursion handles this naturally by cloning the current object before assigning the final key.

## Notes

- Updating a top-level key should still return a new root object.
- Untouched siblings should keep the same references as the original object.
- The original input object should remain unchanged.
- Replacing a nested value with `null` should work the same way as any other value.
- This version assumes every intermediate object already exists and arrays are out of scope.
- Updater callbacks, missing-branch creation, and path validation are intentionally left for the follow-up question.

## Techniques

- Recursion
- Path parsing
- Structural sharing
- Immutable updates

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A developer copies the root with `{ ...source }`, then assigns `copy.draft.info.title = 'Final'`. Why does the original title change too? Explain which copies are needed for this path without copying an untouched `source.theme` branch.

Your notes (optional)
