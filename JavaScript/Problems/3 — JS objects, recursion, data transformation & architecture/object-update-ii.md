---
title: "Object Update II"
aliases:
  - "objectUpdateII"
  - "Object Update II"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Object Update II

> [!info] Problem
> Implement a nested immutable update utility with arrays, updater callbacks, and missing object creation

## Problem

## Object Update II

This is a follow-up to [Object Update](/questions/javascript/object-update).

In the first question, `path` was always a `.`-delimited string, every intermediate object already existed, and arrays were out of scope. In this question, implement the same core idea but with more practical path traversal.

Implement `objectUpdate(source, path, valueOrUpdater)`.

Additional behavior for this question:

- `path` may be either a non-empty dot-delimited string or a non-empty array of keys/indices.
- Numeric path segments can access existing array items.
- The final argument may be either a direct value or an updater function that receives the current leaf value.
- Missing or nullish intermediate object branches should be created automatically as plain objects.

Keep the problem scoped:

- `source` is still a plain object at the root.
- Arrays may already exist in the data, but you do not need to create new arrays.
- Array indices used in the path are guaranteed to point to existing items.
- Existing intermediate values are guaranteed to be objects, arrays, `null`, or `undefined`.
- Any function passed as `valueOrUpdater` is treated as an updater. Storing a function as the direct leaf value is out of scope.
- Inputs are guaranteed valid.
- Path escaping and validation are out of scope.

## Examples

```javascript
const state = {
  todos: [
    { text: 'Ship feature', done: false },
    { text: 'Write docs', done: false },
  ],
  filters: {
    showCompleted: true,
  },
};

const next = objectUpdate(state, 'todos.0.done', (done) => !done);

next.todos[0].done; // true
next.todos !== state.todos; // true
next.todos[0] !== state.todos[0]; // true
next.todos[1] === state.todos[1]; // true
next.filters === state.filters; // true
```

```javascript
const state = {
  draft: {
    title: 'Trip ideas',
  },
};

const next = objectUpdate(state, 'draft.address.city', 'Singapore');

next; // {
//   draft: {
//     title: 'Trip ideas',
//     address: {
//       city: 'Singapore',
//     },
//   },
// }
```

```javascript
const state = {
  form: {
    sections: [{ title: 'Profile' }],
  },
};

const next = objectUpdate(state, ['form', 'sections', 0, 'title'], 'Account');

next.form.sections[0].title; // 'Account'
```

## Arguments

`objectUpdate(source, path, valueOrUpdater)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `source` | `Object` | The root object to update. |
| `path` | `string \| Array<string \| number>` | The non-empty path to update. String paths use `.` as the delimiter. |
| `valueOrUpdater` | `unknown \| ((currentValue) => unknown)` | Either the next non-function value to store or an updater that derives it from the current leaf value. Any function is treated as an updater. |

## Returns

Returns a new object with the update applied.

## Notes

- Clone only the touched objects and arrays along the updated path.
- Untouched branches should keep the same references as the original input.
- Missing, `null`, or `undefined` intermediate branches are only created as plain objects.
- Storing a function as the direct leaf value is out of scope because every function is treated as an updater.
- You do not need to support invalid-path errors, sparse-array behavior, or escaped path tokens.

## Hints

### Hint 1 : Can both path forms drive the same traversal?

### Hint 2 : What shape should each copied container keep?

### Hint 3 : When should an updater run?

## 🤔 Thought Process

- **Immediate Recognition:** Production-grade immutable nested updater supporting arrays, updater callbacks, and auto-vivification.
- **New Requirements over Part I:**
  1. `path`: Can be dot-delimited string (`'a.b.c'`) OR array of keys/indices (`['a', 0, 'b']`).
  2. `valueOrUpdater`: Can be a direct value OR a callback function `(prevValue) => nextValue`.
  3. Array Support: If current container is an `Array` or next key is a numeric index, clone and manipulate as an array (`[...arr]`).
  4. Auto-vivification: If intermediate containers do not exist, create `{}` or `[]` depending on the next key.
- **Recursive Structural Sharing with Arrays:**
  - When cloning a container:
    - If `Array.isArray(container)`: shallow copy with `[...container]`.
    - Else: shallow copy with `{ ...container }`.
  - When reaching the leaf:
    - Resolve new value: `typeof valueOrUpdater === 'function' ? valueOrUpdater(currentVal) : valueOrUpdater`.

---

## 🧠 Mental Model

Think of **Functional Lenses / Immutable Path Updates**:
```
Path: ['users', 1, 'score']
Updater: (x) => x + 10

State: { users: [ { id: 1, score: 5 }, { id: 2, score: 20 } ] }
           │
           ▼
[Cloned Root] ──► users: [Cloned Array]
                            ├── index 0: [Reused Original Reference]
                            └── index 1: [Cloned Object with score: 30]
```

---

## 🔑 Key Concepts

- [[Object Path Traversal]]
- [[DFS Recursion]]
- [[Recursion]]
- Immutable Array Cloning (`[...arr]`) vs Object Cloning (`{ ...obj }`)
- Auto-vivification in immutable pipelines
- Updater function resolution (`prev => next`)

---

## ⚠️ Edge Cases / Traps

- **Updater Function Calling:** If `valueOrUpdater` is a function, invoke with the current target value: `fn(target)`. If the target doesn't exist, pass `undefined`.
- **Array Immutability:** When updating an array index, use `const next = [...current]; next[idx] = ...`, NEVER `current[idx] = ...`.
- **Numeric vs String Keys:** Key `'0'` in array paths must update an array without converting the array into a plain object.
- **Lookahead Auto-vivification:** If `source` is empty `{}` and path is `'a.0.b'`, `a` must become an `Array` `[]`, not an object `{}`.

---

## ⭐ Interview Takeaway

- Distinguish container types when cloning:
  `const clone = Array.isArray(container) ? [...container] : { ...container };`
- Lookahead for container creation:
  `const nextContainer = typeof nextKey === 'number' || /^\d+$/.test(nextKey) ? [] : {};`
- Updater invocation:
  `const nextVal = typeof updater === 'function' ? updater(prevVal) : updater;`

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you distinguish whether an auto-vivified container should be an Array or an Object?" (Check if the upcoming path segment is a non-negative integer).
- "Why does `valueOrUpdater` accept a callback function?" (Enables atomic state calculations based on previous values without race conditions, exactly like React `setCount(prev => prev + 1)`).

### Follow-up Questions
- "How would you handle negative array indices (e.g. `-1` for the last element)?"
- "What if the path specifies deleting a key immutably?"

### Conceptual Questions
- "How does this compare to Immutable.js or Immer?" (Immer uses proxies to let you write mutating code that produces structurally shared copies; this functional utility achieves structural sharing directly).

---

## 🔄 Variations

- **Object Update I:** Simpler object-only version.
- **Deep Set:** In-place mutable version.
- **JSON Patch:** Standard RFC delta application.

---

## 📝 Revision Notes

- Production implementation:
```javascript
export default function objectUpdate(source, path, valueOrUpdater) {
  const keys = Array.isArray(path) ? path : path.split('.');

  function update(current, index) {
    const key = keys[index];
    const isLast = index === keys.length - 1;

    if (isLast) {
      const prevVal = current !== null && typeof current === 'object' ? current[key] : undefined;
      const nextVal = typeof valueOrUpdater === 'function' ? valueOrUpdater(prevVal) : valueOrUpdater;

      if (Array.isArray(current)) {
        const copy = [...current];
        copy[key] = nextVal;
        return copy;
      }
      return { ...current, [key]: nextVal };
    }

    const nextKey = keys[index + 1];
    const isNextIndex = typeof nextKey === 'number' || /^\d+$/.test(nextKey);

    let child = current !== null && typeof current === 'object' ? current[key] : undefined;
    if (child === null || typeof child !== 'object') {
      child = isNextIndex ? [] : {};
    }

    const updatedChild = update(child, index + 1);

    if (Array.isArray(current)) {
      const copy = [...current];
      copy[key] = updatedChild;
      return copy;
    }
    return { ...current, [key]: updatedChild };
  }

  const root = source !== null && typeof source === 'object'
    ? source
    : (typeof keys[0] === 'number' || /^\d+$/.test(keys[0]) ? [] : {});

  return update(root, 0);
}
```

---

## Official Solution

## Object Update II ( Official solution )

Premium
Languages

## Solution

Part 1 still applies: clone only the touched path and reuse everything else. The new pieces are array indices, missing object branches, and updater functions at the leaf.

Each recursive step handles one path segment and returns a new version of the current container. The "container" may be an object or an array, so the clone operation depends on the value being traversed:

- Objects are cloned with `{ ...object }`.
- Arrays are cloned with `[...array]`.

The recursive update has five jobs:

1. Normalize the incoming path into an array of keys.
2. Recursively walk the source value one segment at a time.
3. Clone the current container before changing it:
  - use `{ ...object }` for plain objects
  - use `[...array]` for arrays

4. If an intermediate object key is missing, create `{}` and keep recursing.
5. At the final segment, either assign the direct value or call the updater function with the current leaf value.

Because each recursive call creates at most one new container, only the touched ancestors become new references. Updating `todos.0.done` clones the root object, the `todos` array, and the first todo object. The second todo object keeps the same reference.

| Path segment | Container cloned | Reference behavior |
| --- | --- | --- |
| `todos` | root object | other root fields are reused |
| `0` | `todos` array | untouched array items are reused |
| `done` | first todo object | sibling fields on that todo are copied |
| final write | leaf value | updater receives the previous `done` value |

Path normalization is where string and array paths become the same problem. A dot-delimited path is split into segments, and digit-only string segments become numbers so `todos.0.done` can index into an existing array. Array paths are shallow-copied before use so the caller's path array is not mutated.

Missing branch creation is intentionally object-only. If the current value is `null` or `undefined` while traversing an object key, the solution creates `{}` and continues. Existing arrays keep their shape, and this question does not require creating new arrays.

Updater callbacks are leaf-only. They receive the current leaf value, not the parent object or array, and their return value becomes the new leaf. Every function passed as `valueOrUpdater` is treated as an updater, so storing a function as the direct leaf value is out of scope.

```jsx
function isUpdater(value) {
  return typeof value === 'function';
}

function cloneContainer(value) {
  return Array.isArray(value) ? [...value] : { ...value };
}

function normalizePath(path) {
  if (Array.isArray(path)) {
    return [...path];
  }

  // Convert digit-only segments to numbers so dot-notation paths can index arrays.
  return path
    .split('.')
    .map((segment) => (/^\d+$/.test(segment) ? Number(segment) : segment));
}

function updateAtPath(source, keys, valueOrUpdater) {
  const [key, ...rest] = keys;
  const nextContainer = cloneContainer(source);

  if (rest.length === 0) {
    const currentValue = source[key];
    nextContainer[key] = isUpdater(valueOrUpdater)
      ? valueOrUpdater(currentValue)
      : valueOrUpdater;

    return nextContainer;
  }

  const currentValue = source[key];
  // Missing object branches are created on demand, but array branches keep their shape.
  const childValue =
    currentValue == null && !Array.isArray(source) ? {} : currentValue;

  nextContainer[key] = updateAtPath(childValue, rest, valueOrUpdater);
  return nextContainer;
}

/**
 * @param {Record<string, unknown>} source
 * @param {string | Array<string | number>} path
 * @param {unknown | ((currentValue: unknown) => unknown)} valueOrUpdater
 * @returns {Record<string, unknown>}
 */
export default function objectUpdate(source, path, valueOrUpdater) {
  return updateAtPath(source, normalizePath(path), valueOrUpdater);
}
```

## Common pitfalls

- **Cloning objects but mutating arrays:** Arrays on the updated path need the same immutable treatment as objects. Use a copied array before replacing an item, otherwise the original array is changed.
- **Forgetting to normalize numeric string segments:** The string path `'todos.0.done'` needs to behave like `['todos', 0, 'done']`. Converting digit-only segments to numbers lets the recursion index arrays consistently.

### Creating missing branches inside arrays

Missing branches are only created as plain objects for object keys. Array indices are guaranteed to point to existing items, and the problem does not require sparse-array behavior or automatic array creation.

### Calling updater functions too early

The updater function should run only at the final path segment, with the current leaf value. Calling it at an intermediate segment would replace an entire branch instead of the target value.

## Notes

- Dot-delimited string paths and array paths should both work.
- Paths are guaranteed to be non-empty.
- Numeric string path segments like `'todos.0.done'` should traverse arrays.
- Updating one array item should keep the other array items as the same references.
- Creating `draft.address.city` should only create the missing object branch, not replace unrelated parts of `draft`.
- The original source object should remain unchanged.
- Path escaping and invalid-path validation are out of scope.
- Existing arrays may be traversed and cloned, but new arrays do not need to be created.
- Any function passed as `valueOrUpdater` is treated as an updater; storing a function directly is out of scope.

## Techniques

- Recursion
- Path normalization
- Structural sharing
- Immutable updates

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A generic path copier uses `{ ...container }` at every level. Updating `'todos.0.done'` produces the right leaf value, but a component later calling `next.todos.map(...)` fails. Explain the missed invariant and why deep-cloning every todo is unnecessary.

Your notes (optional)
