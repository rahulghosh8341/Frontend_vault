---
title: "Deep Clone II"
aliases:
  - "deepCloneII"
  - "Deep Clone II"
difficulty: "Hard"
source: GreatFrontEnd
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Clone II

> [!info] Problem
> Implement a function that performs a deep copy of a value, but also handles circular references

## Problem

## Deep Clone II

Zhenghao He
Engineering Manager, Robinhood
**Note:** This is an advanced version of the [Deep Clone](/questions/javascript/deep-clone) question, which you should complete before attempting this question.

It is not realistic to expect candidates to produce a complete deep clone solution in typical interview settings, though the interviewer might ask you for a simpler version like [Deep Clone](/questions/javascript/deep-clone) and discuss the edge cases and how you would handle them.

Implement a `deepClone` function that performs a deep clone as thoroughly as possible, while also handling the following:

- The input object can contain any data type.
- If the input object is cyclic, clone its circular references as well.

## Behavior guide

Use this as the interview scope for the clone:

| Value kind | Expected behavior |
| --- | --- |
| Primitives and `null` | Return the value directly |
| Functions | Return the same function reference |
| Arrays | Return new arrays and recursively clone their contents |
| Plain objects and other ordinary object instances | Return new objects with the same prototype and recursively clone their own properties |
| `Date` and `RegExp` | Return equivalent new instances |
| `Map` and `Set` | Return new containers; clone `Set` entries and `Map` values while preserving `Map` keys |
| Symbol keys on objects | Preserve the symbol-keyed properties |
| Circular object references | Preserve the cycle in the cloned object graph |

You do not need to fully support every browser or platform object. Values such as DOM nodes, promises, weak collections, errors, and typed arrays are outside the intended interview scope.

## Examples

```javascript
const obj1 = {
  num: 0,
  str: '',
  boolean: true,
  unf: undefined,
  nul: null,
  obj: { name: 'foo', id: 1 },
  arr: [0, 1, 2],
  date: new Date(),
  reg: new RegExp('/bar/ig'),
  [Symbol('s')]: 'baz',
};

const clonedObj1 = deepClone(obj1);
clonedObj1.arr.push(3);
obj1.arr; // Should still be [0, 1, 2]

const obj2 = { a: {} };
obj2.a.b = obj2; // Circular reference

const clonedObj2 = deepClone(obj2); // Should not recurse infinitely or cause a stack overflow.

clonedObj2.a.b = 'something new';

obj2.a.b === obj2; // This should still be true
```

## Hints

### Hint 1 : Where does the simpler clone stop working?

### Hint 2 : Which object details are easy to lose?

### Hint 3 : What happens when the same object appears again?

## Asked at these companies

ByteDance
Tiktok

## 🤔 Thought Process

- **Immediate Recognition:** Production-grade deep clone handling arbitrary types, prototype preservation, and cyclic references.
- **Core Challenge: Circular References:**
  - An object pointing to itself (`a.self = a`) causes infinite recursion without a visited cache.
  - Solution: Use a `WeakMap` or `Map` to cache `[originalObject -> clonedObject]` *before* traversing child properties.
- **Type Differentiation:**
  - Primitives and functions: Return directly.
  - Dates: `new Date(val.getTime())`.
  - RegExps: `new RegExp(val.source, val.flags)`.
  - Sets: Create new `Set`, iterate and recursively clone entries.
  - Maps: Create new `Map`, iterate `[k, v]` and recursively clone values (preserving keys).
  - Plain / Prototype-bearing Objects: `Object.create(Object.getPrototypeOf(val))`.
- **Property Traversal:** Use `Reflect.ownKeys()` to capture both string keys and `Symbol` keys.

---

## 🧠 Mental Model

Think of **Graph Traversal with Cycle Detection & Object Memoization**:
- Deep cloning a cyclic object graph is identical to traversing a directed graph.
- Every node visited must be recorded in a `WeakMap` memo cache as soon as the empty shell is instantiated, *before* descending into its edges, so any back-edge immediately resolves to the memoized replica.

---

## 🔑 Key Concepts

- `WeakMap` for cycle tracking and memory-safe caching
- `Reflect.ownKeys()` for string and Symbol keys
- `Object.getPrototypeOf()` and `Object.create()` for prototype preservation
- Specialized constructor cloning (`new Date()`, `new RegExp()`)
- Container traversal (`Map`, `Set`)

---

## ⚠️ Edge Cases / Traps

- **Premature vs Delayed Memoization:** Caching must happen immediately *after* creating the container shell, but *before* recursing into children. If cached after recursing, circular back-references will re-trigger recursive calls infinitely.
- **Symbol Keys:** `Object.keys()` only returns enumerable string keys. `Reflect.ownKeys()` is required to catch Symbol properties.
- **Prototype Chain Loss:** Using `{}` drops custom class prototypes. Always use `Object.create(Object.getPrototypeOf(val))` or `new val.constructor()`.
- **Functions:** Generally returned by reference because cloning closures is not supported in JS without `eval`.

---

## ⭐ Interview Takeaway

- **The Memo Cache Pattern:**
  `function clone(val, cache = new WeakMap()) { ... }`
  `if (cache.has(val)) return cache.get(val);`
  `const copy = Object.create(Object.getPrototypeOf(val)); cache.set(val, copy);`
- Always mention `structuredClone()` as the modern native solution, and explicitly explain what `structuredClone` handles (cyclic graphs, TypedArrays, Date, RegExp, Map, Set) vs what throws (`Function`, DOM nodes, Symbols).

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why use `WeakMap` instead of `Map` for the circular reference cache?" (Allows garbage collection of temporary objects; avoids memory leaks if cache persists).
- "How do you preserve symbol-keyed properties during a clone?" (Use `Reflect.ownKeys(val)`).

### Follow-up Questions
- "How would you handle non-enumerable properties and property descriptors?" (Use `Object.getOwnPropertyDescriptors(val)` + `Object.defineProperties()`).
- "What if an object has getters and setters?" (Using property descriptors preserves getters/setters instead of executing them and flattening to static values).

### Conceptual Questions
- "Why does `JSON.stringify` throw on circular objects?" (It performs a tree serialization; cyclic graphs cause infinite call stacks without a cycle table).

---

## 🔄 Variations

- **Deep Clone I:** JSON-serializable subset only.
- **Deep Equal:** Comparing two cyclic objects for deep structural equality.
- **JSON Patch:** Generating diffs across complex nested structures.

---

## 📝 Revision Notes

- Type dispatch blueprint:
```javascript
function deepClone(val, cache = new WeakMap()) {
  if (val === null || typeof val !== 'object') return val;
  if (val instanceof Date) return new Date(val);
  if (val instanceof RegExp) return new RegExp(val.source, val.flags);
  if (cache.has(val)) return cache.get(val);

  if (val instanceof Set) {
    const copy = new Set();
    cache.set(val, copy);
    val.forEach(item => copy.add(deepClone(item, cache)));
    return copy;
  }
  if (val instanceof Map) {
    const copy = new Map();
    cache.set(val, copy);
    val.forEach((v, k) => copy.set(k, deepClone(v, cache)));
    return copy;
  }

  const copy = Array.isArray(val) ? [] : Object.create(Object.getPrototypeOf(val));
  cache.set(val, copy);
  Reflect.ownKeys(val).forEach(key => {
    copy[key] = deepClone(val[key], cache);
  });
  return copy;
}
```

---

## Official Solution

## Deep Clone II ( Official solution )

Premium
Zhenghao He
Engineering Manager, Robinhood
Languages
**Note:** This is an advanced version of the [Deep Clone](/questions/javascript/deep-clone) question. Complete that first before attempting this question.

This follow-up is intentionally much broader than the original Deep Clone question and touches more obscure corners of the language.

It is not realistic to expect a fully complete deep-clone implementation in a typical interview. The value of this exercise is learning how to detect runtime types, preserve prototypes, recurse through nested containers, and avoid infinite loops from circular references.

## Solution

The main idea is still recursive cloning, but now each object-like value needs type dispatch and identity tracking. The code treats primitives and functions as terminal values, then handles the remaining values based on their runtime tag.

For every supported object-like value, the clone logic needs to:

1. Detect its runtime type accurately.
2. Create the right empty container for that type.
3. Register object shells in a cache before recursing so circular object references can point back to the clone.
4. Recurse into nested values only where that type can actually contain children.

Before cloning those types, use a more precise type check than raw `typeof`. Here, `Object.prototype.toString()` gives stable tags like `array`, `date`, `map`, and `set`.

Here are the main moving parts:

- `Reflect.ownKeys()` lets ordinary object cloning see both string keys and symbol keys, including non-enumerable own properties.
- `Object.create(Object.getPrototypeOf(value))` preserves the original prototype chain for ordinary object instances.
- A `Map` cache records ordinary object clones before their properties are visited, which breaks direct object cycles and preserves repeated object references.
- `Map`, `Set`, `Date`, `RegExp`, and arrays each need type-specific cloning logic instead of a one-size-fits-all object loop.

The dispatch table is:

| Runtime value | Clone strategy | Scope boundary |
| --- | --- | --- |
| Primitive or function | Return as-is | no nested clone needed |
| `Date` / `RegExp` | Construct a new instance | extra own properties on those instances are not copied |
| `Array` / `Set` | Clone contained values | container identity changes |
| `Map` | Clone values, keep keys | key cloning is outside this implementation |
| Ordinary object | Preserve prototype, copy own keys | descriptors/private fields are not preserved |

For an object cycle like `obj.a.b = obj`, the important order is:

| Step | Action | Why it matters |
| --- | --- | --- |
| Visit `obj` | Create an empty clone with the same prototype | A target exists before children are cloned |
| Before visiting properties | Store `cache.set(obj, clonedObj)` | Self-references can resolve to the clone |
| Visit `a` | Clone nested object and cache it | Nested object gets its own identity |
| Visit `a.b` | Find original `obj` in the cache | Assign `clonedObj`, not the original and not a new clone |

Caching the shell before recursion is the difference between preserving the graph shape and recursing forever.

```jsx
function isPrimitiveTypeOrFunction(value) {
  return (
    typeof value !== 'object' || typeof value === 'function' || value === null
  );
}

function getType(value) {
  const type = typeof value;
  if (type !== 'object') {
    return type;
  }

  // `toString` distinguishes built-ins like Array, Map, Set, Date, and RegExp.
  return Object.prototype.toString
    .call(value)
    .replace(/^\[object (\S+)\]$/, '$1')
    .toLowerCase();
}

function deepCloneWithCache(value, cache) {
  if (isPrimitiveTypeOrFunction(value)) {
    return value;
  }

  const type = getType(value);

  if (type === 'set') {
    const cloned = new Set();
    value.forEach((item) => {
      cloned.add(deepCloneWithCache(item, cache));
    });
    return cloned;
  }

  if (type === 'map') {
    const cloned = new Map();
    value.forEach((value_, key) => {
      cloned.set(key, deepCloneWithCache(value_, cache));
    });
    return cloned;
  }

  if (type === 'array') {
    return value.map((item) => deepCloneWithCache(item));
  }

  if (type === 'date') {
    return new Date(value);
  }

  if (type === 'regexp') {
    return new RegExp(value);
  }

  if (cache.has(value)) {
    // Reuse the clone to break cycles and preserve repeated references.
    return cache.get(value);
  }

  // Preserve the original prototype instead of always falling back to a plain object.
  const cloned = Object.create(Object.getPrototypeOf(value));

  cache.set(value, cloned);
  for (const key of Reflect.ownKeys(value)) {
    // `Reflect.ownKeys()` includes symbol keys too.
    const item = value[key];
    cloned[key] = isPrimitiveTypeOrFunction(item)
      ? item
      : deepCloneWithCache(item, cache);
  }

  return cloned;
}

/**
 * @template T
 * @param {T} value
 * @return {T}
 */
export default function deepClone(value) {
  return deepCloneWithCache(value, new Map());
}
```

## Common pitfalls

- Treating `typeof value === 'object'` as enough type information. `Array`, `Date`, `RegExp`, `Map`, `Set`, and `null` all need different behavior.
- Adding the object to the cycle cache after recursing into its properties. The shell has to be cached first so self-references can resolve.
- Assuming `Reflect.ownKeys()` also preserves descriptors. It only finds keys; assigning `cloned[key] = ...` still creates normal data properties.
- Forgetting that functions are returned by reference, not cloned.
- Expecting this implementation to cover every platform object. DOM nodes, promises, weak collections, typed arrays, errors, and many host objects are outside the scope of this interview solution.

## Notes

- [Property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors) are not copied.
- Getters are invoked when `value[key]` is read, and the clone receives the resulting value as a data property.
- `Map` values are cloned, but `Map` keys are kept by their original reference in this implementation. Clone keys too only if the question explicitly requires that behavior.
- `Set` entries are cloned before being added to the new set.
- The cache in this implementation is used for ordinary object cloning. Fully general cycle support for arrays, maps, and sets requires registering those container shells before cloning their contents as well.
- Preserving prototypes for ordinary objects does not make this a complete clone of class instances; constructor internals, descriptors, and private fields remain outside the interview scope.

## One-liner solution

As of writing, all major browsers have native support for performing a deep clone via the `structuredClone` API. See ["Deep-copying in JavaScript using structuredClone" on web.dev](https://web.dev/structured-clone/) for more on `structuredClone`'s features and limitations.

```javascript
const clonedObj = structuredClone(obj);
```

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
An ordinary-object clone adds an entry to its cache only after recursively cloning all properties. It passes tests on trees but overflows on `const value = {}; value.self = value`. Why does a cache alone not solve this, and what must be stored before visiting `self`?

Your notes (optional)
