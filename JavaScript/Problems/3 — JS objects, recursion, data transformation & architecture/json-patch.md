---
title: "JSON Patch"
aliases:
  - "applyJsonPatch"
  - "JSON Patch"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# JSON Patch

> [!info] Problem
> Implement a function that applies add, remove, and replace JSON patch operations to an object

## Problem

## JSON Patch

JSON Patch is a format for describing updates to a JSON document as a list of operations.

For a quick overview, see [jsonpatch.com](https://jsonpatch.com/).

It is often used when systems want to send **incremental updates** instead of resending a full object every time. For example, a server can stream small patches over SSE or WebSockets to update a client-side document, chat state, or tool result in place. This pattern also shows up in streaming AI chat UIs, where the server may send structured deltas as new content arrives.

In this question, implement `applyJsonPatch(document, operations)` which applies a list of patch operations to a nested object and returns the patched result.

This question is intentionally simplified:

- Only `add`, `remove`, and `replace` operations need to be supported.
- The `document` only contains plain objects and JSON primitive values. Arrays are out of scope.
- Every `path` uses `/`-delimited object keys such as `/user/name`.
- Inputs are guaranteed valid.

## Examples

```javascript
applyJsonPatch(
  {
    user: {
      name: 'Alice',
      role: 'admin',
    },
  },
  [
    { op: 'replace', path: '/user/name', value: 'Bob' },
    { op: 'add', path: '/user/active', value: true },
  ],
);
// {
//   user: {
//     name: 'Bob',
//     role: 'admin',
//     active: true,
//   },
// }
```

```javascript
applyJsonPatch(
  {
    settings: {
      theme: 'dark',
      locale: 'en-US',
    },
  },
  [{ op: 'remove', path: '/settings/locale' }],
);
// {
//   settings: {
//     theme: 'dark',
//   },
// }
```

## Arguments

`applyJsonPatch(document, operations)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `document` | `Object` | A nested object containing JSON primitive values or other plain objects. |
| `operations` | `Array` | A list of patch operations to apply in order. |

Each operation has one of the following shapes:

```javascript
{ op: 'add', path: string, value: unknown }
{ op: 'remove', path: string }
{ op: 'replace', path: string, value: unknown }
```

## Returns

Returns a new object with all operations applied in order.

## Notes

- `add` creates or overwrites the final property at `path`.
- `remove` deletes the final property at `path`.
- `replace` overwrites the existing property at `path`.
- Paths in this question are always non-empty and always point to properties inside the object.
- Do not mutate `document` or `operations`.
- The returned object should not share nested object references with `document` or with values supplied by `operations`.

## Resources

- [RFC 6902: JSON Patch](https://datatracker.ietf.org/doc/html/rfc6902)
- [RFC 6901: JSON Pointer](https://datatracker.ietf.org/doc/html/rfc6901)

## Hints

### Hint 1 : Which document does the next operation see?

### Hint 2 : Where does a path operation act?

### Hint 3 : Which inputs could still share a nested reference?

## 🤔 Thought Process

- **Immediate Recognition:** Implementation of RFC 6902 (JSON Patch) standard operations: `add`, `remove`, and `replace`.
- **Core Operations:**
  - `add`: Inserts value at path. If target is array, inserts at specified index (or appends if `-`). If object, sets property.
  - `remove`: Removes element at index from array or deletes property from object.
  - `replace`: Replaces existing value at path (equivalent to `remove` then `add`).
- **JSON Pointer Path Parsing:**
  - Paths are formatted as `/foo/bar/0`.
  - Splitting `/` yields segment array (skipping initial empty string).
  - Special escape sequences in RFC 6902: `~1` represents `/`, `~0` represents `~`.
- **Traversal to Target Parent:**
  - Navigate down to the second-to-last segment to find the parent container.
  - The last segment is the target key or array index.
- **Array Operations:**
  - `add` on array: `splice(index, 0, value)`. If index is `'-'`, push to end.
  - `remove` on array: `splice(index, 1)`.

---

## 🧠 Mental Model

Think of **DOM / File Mutator Operations on a Document Tree**:
```
Document Tree: { users: [ 'Alice', 'Bob' ] }

Patch: { op: "add", path: "/users/1", value: "Charlie" }
          │
          ▼
Find Parent: document.users
Target Index: 1
Action: splice(1, 0, 'Charlie')
          │
          ▼
Updated: { users: [ 'Alice', 'Charlie', 'Bob' ] }
```

---

## 🔑 Key Concepts

- [[Object Path Traversal]]
- [[Recursion]]
- [[Type Checking]]
- RFC 6902 JSON Pointer specification
- Array splicing (`splice`) vs Object property assignment/deletion
- In-place mutation vs Immutable document patching

---

## ⚠️ Edge Cases / Traps

- **Array Index Insertion:** Adding to `/arr/1` on `['a', 'b']` must SHIFT elements right (`splice(1, 0, val)`), NOT overwrite index 1! Overwriting is `replace`, not `add`.
- **The `'-'` End-of-Array Token:** In JSON patch, path `/arr/-` with `add` means append to the end of the array (`push`).
- **Root Path (`""`):** If path is `""` or `"/"`, operations target the root document itself.
- **Numeric String Keys in Objects:** Check whether the parent is `Array.isArray(parent)` or a plain object. A key like `'0'` on an object is an object property, not an array index.

---

## ⭐ Interview Takeaway

- **JSON Pointer Tokenizer:** `path.split('/').slice(1).map(s => s.replace(/~1/g, '/').replace(/~0/g, '~'))`.
- Crucial distinction:
  - `add` on Array = `splice(idx, 0, val)`
  - `add` on Object = `parent[key] = val`
  - `replace` on Array/Object = `parent[key] = val`
  - `remove` on Array = `splice(idx, 1)`
  - `remove` on Object = `delete parent[key]`

---

## 🎯 Common Interview Questions

### Direct Questions
- "What is JSON Patch (RFC 6902) and where is it used?" (Standardized delta format for incremental document synchronization over WebSockets, SSE, and REST `PATCH`).
- "How does `add` differ from `replace` on an array?" (`add` inserts an element and shifts subsequent elements; `replace` overwrites the item at that index).

### Follow-up Questions
- "How do you handle JSON pointer escape sequences (`~0` and `~1`)?" (Decode `~1` to `/` and `~0` to `~`).
- "How would you implement the other RFC 6902 operations: `move`, `copy`, and `test`?" (`test` asserts target matches expected value; `move` = remove + add).

### Conceptual Questions
- "How does JSON Patch compare to JSON Merge Patch (RFC 7396)?" (Merge Patch is simpler and looks like the target document, but cannot express array insertions or removing keys with null values).

---

## 🔄 Variations

- **Deep Set:** Writing a value at a dot-path.
- **Undo Redo Manager:** Generating inverse patches for history rollback.
- **Collaborative CRDT / OT:** Conflict-free operational transformations on tree documents.

---

## 📝 Revision Notes

- Dispatch blueprint:
```javascript
export default function applyJsonPatch(doc, ops) {
  for (const op of ops) {
    const tokens = op.path.split('/').slice(1);
    let target = doc;
    for (let i = 0; i < tokens.length - 1; i++) {
      target = target[tokens[i]];
    }
    const key = tokens[tokens.length - 1];

    if (op.op === 'add') {
      if (Array.isArray(target)) {
        if (key === '-') target.push(op.value);
        else target.splice(Number(key), 0, op.value);
      } else {
        target[key] = op.value;
      }
    } else if (op.op === 'replace') {
      target[key] = op.value;
    } else if (op.op === 'remove') {
      if (Array.isArray(target)) {
        target.splice(Number(key), 1);
      } else {
        delete target[key];
      }
    }
  }
  return doc;
}
```

---

## Official Solution

## JSON Patch ( Official solution )

Languages

## Solution

This interview version is mostly about walking nested object paths while preserving immutability. Because the inputs are guaranteed valid and arrays are excluded, the code can stay small and focused.

There are two important correctness rules:

- Patch operations run sequentially, so later operations should see the results of earlier ones.
- The original `document` must stay untouched.

Create one working copy, then apply each operation to that copy in order:

1. Deep-clone the input document once so the original data stays untouched.
2. For each operation, split its `path` into object keys.
3. Walk to the parent object of the target key.
4. Apply the final `add`, `remove`, or `replace` directly on the cloned structure.

Path handling has one job in this simplified version: convert a path like `/user/name` into `['user', 'name']`. The walk stops at the parent container (`user` in this example), because the final segment decides which key to add, remove, or replace.

Cloning once up front is enough because every operation can then mutate that cloned structure in place, and the next operation will naturally observe the updated result. Inserted and replaced values are cloned too, so the result does not share nested references with the input document or the operations payload.

For these operations:

```javascript
[
  { op: 'add', path: '/settings/layout', value: 'grid' },
  { op: 'replace', path: '/settings/theme', value: 'light' },
  { op: 'remove', path: '/settings/layout' },
];
```

the working copy changes sequentially:

| Operation | Parent path walked to | Final key | Working copy effect |
| --- | --- | --- | --- |
| `add /settings/layout` | `/settings` | `layout` | creates `layout: 'grid'` |
| `replace /settings/theme` | `/settings` | `theme` | changes `dark` to `light` |
| `remove /settings/layout` | `/settings` | `layout` | deletes only `layout` |

```jsx
function cloneValue(value) {
  if (typeof value !== 'object' || value === null) {
    return value;
  }

  // Clone inserted and replaced subtrees too so the result never shares nested
  // references with the input document or operations payload.
  return Object.fromEntries(
    Object.entries(value).map(([key, nestedValue]) => [
      key,
      cloneValue(nestedValue),
    ]),
  );
}

function getSegments(path) {
  return path.slice(1).split('/');
}

/**
 * @param {Record<string, unknown>} document
 * @param {Array<
 *   | { op: 'add', path: string, value: unknown }
 *   | { op: 'remove', path: string }
 *   | { op: 'replace', path: string, value: unknown }
 * >} operations
 * @returns {Record<string, unknown>}
 */
export default function applyJsonPatch(document, operations) {
  const result = cloneValue(document);

  operations.forEach((operation) => {
    const segments = getSegments(operation.path);
    let current = result;

    // Stop at the parent container; the final segment decides which key to edit.
    for (let index = 0; index < segments.length - 1; index += 1) {
      current = current[segments[index]];
    }

    const finalKey = segments[segments.length - 1];

    switch (operation.op) {
      case 'add':
      case 'replace':
        current[finalKey] = cloneValue(operation.value);
        break;
      case 'remove':
        delete current[finalKey];
        break;
    }
  });

  return result;
}
```

## Common pitfalls

- **Applying operations to the original document:** The operations are allowed to mutate the working copy, but not the input `document`. Clone first, then patch the clone.
- **Resolving every operation against the original document:** Operations are sequential. If one operation adds `/user/active`, a later operation should be able to replace or remove `/user/active` from the updated working copy.
- **Walking all the way to the final value before editing:** For `add`, `remove`, and `replace`, the final path segment is the property being edited. Walk only to the parent object, then apply the operation at the final key.
- **Forgetting to clone operation values:** When an `add` or `replace` operation inserts an object value, clone that value before storing it so the result does not share nested references with `operations`.

## Notes

- Multiple operations can touch the same branch of the object.
- A later operation can replace a value that was added by an earlier operation.
- Removing a property should only affect that final key, not its sibling properties.
- The original `document` and `operations` should remain unchanged.
- `add` creates or overwrites the final object property at `path`.
- `remove` deletes only the final object property at `path`.
- `replace` overwrites the final object property at `path`.
- Arrays, invalid paths, escaped JSON Pointer tokens, and root replacement are out of scope for this simplified version.

## Techniques

- Recursion
- Path parsing
- Immutable updates

## Resources

- [RFC 6902: JSON Patch](https://datatracker.ietf.org/doc/html/rfc6902)
- [RFC 6901: JSON Pointer](https://datatracker.ietf.org/doc/html/rfc6901)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
An optimization clones the document, resolves every patch path's parent object once, and then applies all operations to those cached parents.

```javascript
const document = { settings: { theme: 'dark' } };
const operations = [
  { op: 'replace', path: '/settings', value: { theme: 'light' } },
  { op: 'replace', path: '/settings/theme', value: 'blue' },
];
```

Why is caching all parent objects before applying the first operation incorrect, even though both paths initially exist?

Your notes (optional)
