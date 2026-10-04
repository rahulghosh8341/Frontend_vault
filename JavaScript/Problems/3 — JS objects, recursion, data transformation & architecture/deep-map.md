---
title: "Deep Map"
aliases:
  - "deepMap"
  - "Deep Map"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Map

> [!info] Problem
> Implement a function to recursively transform values

## Problem

## Deep Map

Implement a function `deepMap(value, fn)` that returns a new value containing the results of calling a provided function on every value that is not an `Array` or plain object, including values within nested arrays and plain objects. The function `fn` is called with a single argument: the value being mapped or transformed. For every callback invocation, `this` should be the original root `value` passed to `deepMap`.

## Examples

```javascript
const double = (x) => x * 2;

deepMap(2, double); // 4
deepMap([1, 2, 3], double); // [2, 4, 6]
deepMap({ a: 1, b: 2, c: 3 }, double); // { a: 2, b: 4, c: 6 }
deepMap(
  {
    foo: 1,
    bar: [2, 3, 4],
    qux: { a: 5, b: 6 },
  },
  double,
); // => { foo: 2, bar: [4, 6, 8], qux: { a: 10, b: 12 } }
```

## Hints

### Hint 1 : Which values are leaves?

### Hint 2 : How can every leaf see the same `this`?

## 🤔 Thought Process

- **Immediate Recognition:** Recursive tree visitor applying a transformation callback to every non-container leaf.
- **Container vs Leaf Definition:**
  - Containers: Plain objects (`{}`) and `Array` instances.
  - Leaves: Everything else (numbers, strings, booleans, `null`, `undefined`, `Date`, functions, etc.).
- **Crucial Requirement: `this` Context:**
  - The problem specifies: "For every callback invocation, `this` should be the original root `value` passed to `deepMap`."
  - Must invoke `fn.call(rootValue, leafValue)`.
- **Traversal Strategy:**
  - Capture `rootValue = value` in an outer scope.
  - Define recursive helper `mapLeaf(current)`.
  - If array: `current.map(mapLeaf)`.
  - If plain object: Create new object, copy keys with `out[k] = mapLeaf(current[k])`.
  - Otherwise (leaf): Return `fn.call(rootValue, current)`.

---

## 🧠 Mental Model

Think of **Leaves on a Tree**:
- Containers (objects and arrays) form the branches and twigs.
- Leaf values (primitives, dates, functions) are the actual leaves hanging on the branches.
- `deepMap` traverses the branching structure untouched and applies a color dye (`fn`) exclusively to every leaf, while keeping a thread connected back to the tree trunk (`this = root`).

---

## 🔑 Key Concepts

- [[Recursion]]
- Higher-Order Functions & Callbacks
- Explicit `this` binding (`Function.prototype.call`)
- Plain object detection vs leaf primitives

---

## ⚠️ Edge Cases / Traps

- **Preserving `this` Context:** Calling `fn(leaf)` instead of `fn.call(root, leaf)` will fail the context requirement if `fn` references `this`.
- **`null` Check:** `typeof null === 'object'`. Do not treat `null` as a container; `null` is a leaf and must be transformed by `fn`.
- **Non-plain Objects:** Instances of custom classes or built-ins like `Date` are not plain objects and should be treated as leaves.
- **Empty Containers:** Empty arrays `[]` and empty objects `{}` should return new empty containers without invoking `fn`.

---

## ⭐ Interview Takeaway

- Use an inner recursive function that closes over the original root value:
  ```javascript
  export default function deepMap(value, fn) {
    function traverse(val) {
      if (Array.isArray(val)) return val.map(traverse);
      if (isPlainObject(val)) {
        const res = {};
        for (const [k, v] of Object.entries(val)) res[k] = traverse(v);
        return res;
      }
      return fn.call(value, val);
    }
    return traverse(value);
  }
  ```
- Always check `isPlainObject` accurately using `Object.prototype.toString.call(val) === '[object Object]'` or prototype checking.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why do we use `fn.call(root, val)` instead of `fn(val)`?" (To bind `this` to the root object as required by the specification).
- "How do you distinguish between a plain object and special objects like Date or RegExp?" (Check `Object.prototype.toString.call(val)` or `val.constructor === Object`).

### Follow-up Questions
- "What if `fn` itself returns an object or array? Should that returned container be recursed into?" (No; leaf transformations are terminal).
- "How would you implement an asynchronous version (`deepMapAsync`) that awaits promises returned by `fn`?"

### Conceptual Questions
- "How does this pattern relate to the Visitor design pattern in AST compilers like Babel?"

---

## 🔄 Variations

- **Object Map:** Shallow mapping over object values.
- **Deep Map Keys:** Transforming object keys rather than values (e.g. `camelCaseKeys`).
- **Deep Filter:** Pruning leaves that fail a predicate function.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
function isPlainObject(val) {
  if (val === null || typeof val !== 'object') return false;
  const proto = Object.getPrototypeOf(val);
  return proto === null || proto === Object.prototype;
}

export default function deepMap(value, fn) {
  function traverse(current) {
    if (Array.isArray(current)) {
      return current.map(traverse);
    }
    if (isPlainObject(current)) {
      const result = {};
      for (const [key, val] of Object.entries(current)) {
        result[key] = traverse(val);
      }
      return result;
    }
    return fn.call(value, current);
  }

  return traverse(value);
}
```

---

## Official Solution

## Deep Map ( Official solution )

Premium
Languages
Recursively rebuild arrays and plain objects, and only call `fn` at leaf values. The extra wrinkle is that every callback should see the original input as `this`, not the current nested value.

## Clarification questions

- Should `Map` and `Set` values be traversed?
  - To keep the question simple, no. There are no test cases containing `Map`s or `Set`s, but support can be added if needed.

- What should the value of `this` be within the callback function?
  - The input `value`.

## Solution

The traversal rebuilds the tree:

- Arrays become new arrays whose items are recursively mapped.
- Plain objects become new plain objects whose property values are recursively mapped.
- Everything else is a leaf, so it is passed to `fn`.

That means the original object or array structure is not mutated. Only the non-container leaves are transformed.

The recursive helper's condition is: return the mapped version of the current element while preserving container shape. For containers, that means rebuilding and recurring; for leaves, that means invoking the callback exactly once.

### Recursion cases

1. Arrays: map over items and recurse into each one.
2. Plain objects: rebuild the object entry by entry and recurse into each property value.
3. Everything else: this is the base case, so call `fn`.

Using a plain-object check matters here. `typeof value === 'object'` is too broad because values like `Date`, `Set`, and `null` should be treated as leaves for this question.

### Callback context

The callback context is the other important detail. The code uses a helper that receives both the current element and the original root value. That extra `original` argument is what lets every leaf call use `fn.call(original, element)`, even deep inside the structure.

For `deepMap({ foo: 1, bar: [2, { baz: 3 }] }, double)`, the traversal decisions are:

| Current value | Case | Result |
| --- | --- | --- |
| root object | rebuild entries | `{ foo: ..., bar: ... }` |
| `1` | leaf | `double(1)` |
| `[2, { baz: 3 }]` | array | map each item recursively |
| `2` | leaf | `double(2)` |
| `{ baz: 3 }` | plain object | rebuild entries |
| `3` | leaf | `double(3)` |

No callback is run for the array or plain objects themselves, only for the leaves.

```jsx
/**
 * @param {unknown} value
 * @param {(value: unknown) => unknown} fn
 * @returns {unknown}
 */
export default function deepMap(value, fn) {
  return mapHelper(value, fn, value);
}

function isPlainObject(value) {
  if (value == null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === null || prototype === Object.prototype;
}

function mapHelper(element, fn, original) {
  // Handle arrays.
  if (Array.isArray(element)) {
    return element.map((item) => mapHelper(item, fn, original));
  }

  // Handle plain objects.
  if (isPlainObject(element)) {
    return Object.fromEntries(
      Object.entries(element).map(([key, value]) => [
        key,
        mapHelper(value, fn, original),
      ]),
    );
  }

  // Handle other types.
  return fn.call(original, element);
}
```

## Common pitfalls

- Treating every object-like value as a container. `null`, `Date`, `RegExp`, `Map`, `Set`, and functions should be leaf values for this question.
- Calling `fn` for arrays or plain objects themselves instead of only for leaves.
- Mutating the input structure while walking it. The intended approach rebuilds arrays and plain objects.
- Letting `this` inside `fn` become the current nested object instead of the original root input.

## Scope

- Accessing `this` within the callback function.
- Values such as `null`, `Date`, `Symbol`, etc.

## Notes

- This is not a generic clone utility. Special object types are preserved as leaf values unless `fn` returns something else for them.
- The callback receives only the leaf value as its argument. If the caller needs path information such as `bar.baz`, that is outside this API.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
After this code runs, which reference comparisons are `true`? Select all that apply.

```javascript
const stamp = new Date(0);
const input = {
  nested: { score: 2 },
  list: [3],
  stamp,
};

const output = deepMap(input, (value) => value);
```
