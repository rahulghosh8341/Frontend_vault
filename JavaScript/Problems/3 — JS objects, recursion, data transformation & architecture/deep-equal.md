---
title: "Deep Equal"
aliases:
  - "deepEqual"
  - "Deep Equal"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Deep Equal

> [!info] Problem
> Implement a function that determines whether two values are equal

## Problem

## Deep Equal

Zhenghao He
Engineering Manager, Robinhood
Implement a function `deepEqual` that performs a deep comparison between two values. It returns `true` if the two input values are deemed equal and `false` otherwise.

- Inputs are limited to `undefined`, numbers, strings, booleans, `null`, arrays, and plain objects.
- Arrays may be sparse. Their length and present indices are part of the structure being compared.
- There will not be cyclic objects, i.e. objects with circular references.

## Examples

```javascript
deepEqual('foo', 'foo'); // true
deepEqual({ id: 1 }, { id: 1 }); // true
deepEqual([1, 2, 3], [1, 2, 3]); // true
deepEqual([{ id: '1' }], [{ id: '2' }]); // false
```

## Hints

### Hint 1 : When should comparison recurse?

### Hint 2 : What makes two container shapes different?

## Asked at these companies

Google

## 🤔 Thought Process

- **Immediate Recognition:** Recursive structural equality comparison between two arbitrary values without circular references.
- **Core Requirements:**
  - Primitives: Strict equality comparison, with special handling for `NaN` (`Object.is` or `Number.isNaN`).
  - Containers:
    - Arrays: Must have identical length and identical elements at every index (including handling sparse array holes).
    - Objects: Must have identical own enumerable keys and identical values for every key.
    - Type mismatch: Objects and arrays must NOT equal each other, even if both are empty.
- **Step-by-Step Approach:**
  1. Fast path: If `a === b`, return `true` (unless both are numbers where `+0` vs `-0` matters, but usually standard `===` is sufficient, with `NaN` fallback).
  2. If either is primitive or `null`, check `Number.isNaN(a) && Number.isNaN(b)`. If not both `NaN`, return `false`.
  3. Check container types:
     - `Array.isArray(a) !== Array.isArray(b)` -> `false`.
     - `(typeof a === 'object') !== (typeof b === 'object')` -> `false`.
  4. Compare arrays: Check `a.length === b.length`. For sparse arrays, verify `i in a === i in b`, then recurse `deepEqual(a[i], b[i])`.
  5. Compare objects: Check `Object.keys(a).length === Object.keys(b).length`. For each key in `a`, check `key in b` and `deepEqual(a[key], b[key])`.

---

## 🧠 Mental Model

Think of **Lockstep Tree Traversal**:
- Two trees are evaluated simultaneously from the root down.
- At every step, compare node signatures (type, key count). If signatures match, lockstep-recurse into each corresponding pair of child edges. Any mismatch halts and bubbles `false` up the stack.

---

## 🔑 Key Concepts

- Structural equality vs Referential equality (`===`)
- `Object.is()` / `NaN` equality mechanics
- Sparse array handling (`index in array`)
- Key counting via `Object.keys()`

---

## ⚠️ Edge Cases / Traps

- **`null` Trap:** `typeof null === 'object'`. Comparing `{}` and `null` without guarding against `null` will throw when calling `Object.keys(null)`.
- **`NaN` Identity:** In JS, `NaN === NaN` is `false`. A naive `===` check will report that `NaN` does not equal `NaN`.
- **Sparse Arrays:** An array with empty slots `[ , ]` vs `[undefined]`. Both have `length === 1`, but `0 in [ , ]` is `false` while `0 in [undefined]` is `true`.
- **Arrays vs Plain Objects:** Both report `typeof === 'object'`. Comparing `[]` and `{}` must return `false`.

---

## ⭐ Interview Takeaway

- Always start with the fast referential check: `if (a === b) return true;`
- Immediately handle `NaN`: `if (Number.isNaN(a) && Number.isNaN(b)) return true;`
- Handle `null` and primitive types before invoking `Object.keys()`.
- Compare lengths/key counts *before* looping through children to fail fast in $O(1)$.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does `[1, 2] === [1, 2]` evaluate to `false` in JavaScript?" (Compares heap memory references, not structural values).
- "How do you distinguish an array with an undefined element from a sparse array slot?" (Using the `in` operator: `index in arr`).

### Follow-up Questions
- "How would you handle cyclic data structures?" (Track visited pairs in a `Set` or `Map` of seen references).
- "How would you compare `Date` and `RegExp` objects?" (Compare `a.getTime() === b.getTime()` and `a.source === b.source && a.flags === b.flags`).

### Conceptual Questions
- "How does React's `shallowEqual` differ from `deepEqual`?" (React only compares own keys at depth 1 for performance; deep equality on large trees causes frame drops).

---

## 🔄 Variations

- **Deep Equal with Cycles:** Cycle-tolerant deep comparison using visited pair sets.
- **Deep Clone:** Constructing a duplicate graph instead of verifying equality.
- **Shallow Equal:** Depth-1 equality check used in `React.memo`.

---

## 📝 Revision Notes

- Production-grade recursive implementation:
```javascript
export default function deepEqual(a, b) {
  if (Object.is(a, b)) return true;

  if (
    a === null ||
    b === null ||
    typeof a !== 'object' ||
    typeof b !== 'object'
  ) {
    return false;
  }

  if (Array.isArray(a) !== Array.isArray(b)) return false;

  if (Array.isArray(a)) {
    if (a.length !== b.length) return false;
    for (let i = 0; i < a.length; i++) {
      if (i in a !== i in b) return false;
      if (!deepEqual(a[i], b[i])) return false;
    }
    return true;
  }

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  if (keysA.length !== keysB.length) return false;

  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
    if (!deepEqual(a[key], b[key])) return false;
  }

  return true;
}
```

---

## Official Solution

## Deep Equal ( Official solution )

Zhenghao He
Engineering Manager, Robinhood
Languages
Deep equality only becomes tractable once the recursive scope is clear. The submitted comparison handles structure for plain objects and arrays, and treats everything else as a final value comparison.

## Solution

The recommended default approach is:

1. Detect whether both values are plain objects or arrays.
2. If they are, compare their entries recursively.
3. Otherwise fall back to `Object.is` for the final value comparison.

`Object.is` is a better primitive comparison than `===` here because it treats `NaN` as equal to itself and distinguishes `-0` from `+0`.

The important boundary is that only arrays and plain objects recurse. A `Date`, `RegExp`, `Map`, or `Set` is treated as a final value in this interview-sized version unless the prompt asks for richer built-in support.

There are two reasonable ways to order those checks. The first version is the one to internalize because it makes the recursion boundary explicit.

### Approach 1: Handling arrays and objects first

The main challenge is deciding when to recurse. `typeof` is not enough for that because `null`, arrays, and many built-in objects all report as `'object'`.

Instead, use `Object.prototype.toString` to classify the values. That keeps recursion limited to plain objects and arrays, while everything else can stay on the `Object.is` path.

A small helper such as `shouldDeepCompare()` makes that recursion check explicit:

```javascript
// Warning: Incomplete solution. See below.
function shouldDeepCompare(type) {
  return type === '[object Object]' || type === '[object Array]';
}

function getType(value) {
  return Object.prototype.toString.call(value);
}

export default function deepEqual(valueA, valueB) {
  const typeA = getType(valueA);
  const typeB = getType(valueB);

  if (typeA === typeB && shouldDeepCompare(typeA) && shouldDeepCompare(typeB)) {
    // When both values are objects or arrays, recurse into them.
  }

  return Object.is(valueA, valueB);
}
```

Once both values are confirmed to be arrays or plain objects, convert them to `Object.entries(...)` and compare:

1. The number of entries.
2. That every key in the first value exists in the second.
3. That each corresponding value is deeply equal.

Using `Object.entries` is convenient because it enumerates own enumerable string-keyed properties, ignores inherited properties, and gives an easy early exit when the entry counts differ. The `Object.hasOwn(...)` check then prevents inherited properties on the second value from satisfying a required key.

For `{ user: { id: 1 } }` compared with `{ user: { id: 1 } }`, the recursive trace is:

| Level | Values being compared | Check that must pass |
| --- | --- | --- |
| root | two plain objects | same type and same number of entries |
| key `user` | two plain objects | key exists on both sides |
| key `id` | `1` and `1` | `Object.is(1, 1)` |
| unwind | all nested comparisons returned `true` | root comparison returns `true` |

If any level has a missing key, different entry count, different runtime type, or failed primitive comparison, the recursion can stop with `false`.

```jsx
function shouldDeepCompare(type) {
  return type === '[object Object]' || type === '[object Array]';
}

function getType(value) {
  return Object.prototype.toString.call(value);
}

/**
 * @param {unknown} valueA
 * @param {unknown} valueB
 * @returns {boolean}
 */
export default function deepEqual(valueA, valueB) {
  // Check for arrays/objects equality.
  const typeA = getType(valueA);
  const typeB = getType(valueB);

  // Only compare the contents if they're both arrays or both objects.
  // If typeA === typeB, shouldDeepCompare(typeA) implies shouldDeepCompare(typeB).
  if (typeA === typeB && shouldDeepCompare(typeA)) {
    // Sparse arrays can have a length that exceeds Object.entries() output
    // (holes are not own properties), so compare array length directly.
    if (Array.isArray(valueA) && valueA.length !== valueB.length) {
      return false;
    }

    const entriesA = Object.entries(valueA);
    const entriesB = Object.entries(valueB);

    if (entriesA.length !== entriesB.length) {
      return false;
    }

    return entriesA.every(
      // Make sure the other object has the same properties defined.
      ([k, v]) => Object.hasOwn(valueB, k) && deepEqual(v, valueB[k]),
    );
  }

  // Check for primitives + type equality.
  return Object.is(valueA, valueB);
}
```

### Approach 2: Handling primitives first

The order can also be reversed: first try `Object.is`, then recurse only if both values are traversable containers. This is a valid alternative for eliminating the non-recursive cases early.

For plain objects and arrays, the recursive responsibility is the same:

1. Check that both objects have the same keys:
  1. Both objects have the same number of keys.
  2. All of the first object's keys exist in the other object.

2. Recursively check that each key's value is the same.

```jsx
export default function deepEqual(valueA: unknown, valueB: unknown): boolean {
  // Check primitives for equality.
  if (Object.is(valueA, valueB)) {
    return true;
  }

  const bothObjects =
    Object.prototype.toString.call(valueA) === '[object Object]' &&
    Object.prototype.toString.call(valueB) === '[object Object]';
  const bothArrays = Array.isArray(valueA) && Array.isArray(valueB);

  // At this point, they can still be primitives but of different types.
  // If they had the same value, they would have been handled earlier in Object.is().
  // So if they're not both objects or both arrays, they're definitely not equal.
  if (!bothObjects && !bothArrays) {
    return false;
  }

  const comparableA = valueA as Record<string, unknown> | Array<unknown>;
  const comparableB = valueB as Record<string, unknown> | Array<unknown>;

  // Sparse arrays can have a length that exceeds Object.keys() output, so
  // compare array length directly before falling back to the keys check.
  if (
    bothArrays &&
    (valueA as Array<unknown>).length !== (valueB as Array<unknown>).length
  ) {
    return false;
  }

  // Compare the keys of arrays and objects.
  if (Object.keys(comparableA).length !== Object.keys(comparableB).length) {
    return false;
  }

  for (const key in comparableA) {
    // Make sure the other side actually has the key. Otherwise `{ foo: undefined }`
    // would compare equal to `{ bar: 1 }` because `comparableB.foo` is also undefined.
    if (
      !Object.hasOwn(comparableB, key) ||
      !deepEqual(comparableA[key], comparableB[key])
    ) {
      return false;
    }
  }

  // All checks passed, the arrays/objects are equal.
  return true;
}
```

## Common pitfalls

- Recursing into every value where `typeof value === 'object'`. `null`, `Date`, `Map`, `Set`, `RegExp`, and class instances are outside this structural comparison scope.
- Using `===` for the final comparison and accidentally treating `NaN` as unequal to itself.
- Comparing values at matching positions without first checking that both containers have the same number of keys.
- Checking `valueB[k]` without confirming that `k` is an own property of `valueB`.
- Trying to compare cyclic objects without tracking visited object pairs.

## Notes

- `Object.is` treats `NaN` as equal to itself and distinguishes `+0` from `-0`.
- `null`, plain objects, and arrays are handled explicitly by the type checks.
- Cyclic objects, i.e. objects with circular references, are not handled.
- [Property descriptors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptors) are not taken into account when comparing properties.
- Non-enumerable properties and symbol-keyed properties are not compared.
- Prototypes are not compared. Two plain-object-shaped values with different prototypes can still compare equal if their enumerable entries match.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
What does this code produce?

```javascript
const first = deepEqual(
  { status: undefined, id: 1 },
  { state: undefined, id: 1 },
);

const second = deepEqual(
  { status: undefined, id: 1 },
  { status: undefined, id: 1 },
);

const result = [first, second];
```
