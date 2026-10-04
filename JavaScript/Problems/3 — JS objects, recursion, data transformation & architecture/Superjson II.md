---
title: Superjson II
aliases:
  - Superjson II
difficulty: Hard
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/superjson-ii"
companies:
  - "[[Anthropic]]"
  - "[[OpenAI]]"
pattern:
  - "[[Recursion]]"
concepts:
  - "[[Recursion]]"
  - "[[Map]]"
  - "[[Set Lookup]]"
  - "[[Data Serialization]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Superjson II

> [!info] Problem
> Extend the serialize and deserialize APIs to support Map and Set while preserving insertion order

## Problem

## Superjson II

This is a follow-up to [Superjson](/questions/javascript/superjson).

`JSON.stringify()` works well for plain JSON data, but it loses information for values like `undefined`, `Date`, and `RegExp`.

Popular libraries like [SuperJSON](https://github.com/flightcontrolhq/superjson) and [devalue](https://github.com/sveltejs/devalue) were built to solve exactly this kind of problem.

Implement the same two functions:

- `serialize(value)`: returns a string.
- `deserialize(serialized)`: returns the original value represented by that string.

This question extends the behavior implemented in [Superjson](/questions/javascript/superjson) and adds support for:

- `Map`
- `Set`

Collection values can contain any previously supported value, as well as nested `Map` and `Set`. `Map` keys should also round-trip correctly, including object keys. Insertion order should be preserved for both collections.

As before, the exact serialized format is up to you.

## Examples

```javascript
const value = new Map([
  ['createdAt', new Date('2026-01-01T00:00:00.000Z')],
  ['flags', new Set([undefined, /user-\d+/gi])],
]);

const serialized = serialize(value);
const restored = deserialize(serialized);

restored instanceof Map; // true
restored.get('createdAt') instanceof Date; // true
restored.get('flags') instanceof Set; // true
```

```javascript
const key = { id: 1 };
const value = new Map([[key, { retries: Infinity }]]);
const restored = deserialize(serialize(value));

const [[restoredKey, restoredValue]] = restored.entries();
restoredKey; // { id: 1 }
restoredValue.retries; // Infinity
```

## Notes

- You may assume `deserialize` only receives strings produced by `serialize`.
- `serialize` should still throw a `TypeError` for functions, symbols, symbol keys, cyclic references, sparse arrays, and custom class instances.
- Malformed serialized strings are still out of scope.

## Resources

- [SuperJSON](https://github.com/flightcontrolhq/superjson)
- [devalue](https://github.com/sveltejs/devalue)

## Hints

### Hint 1 : Why can't a map become a plain object?

### Hint 2 : Do collections participate in cycles?

## Asked at these companies

Anthropic
OpenAI

## 🤔 Thought Process

- **Immediate Recognition:** Extension of [[Superjson]] supporting ES6 collections: `Map` and `Set`.
- **Core Problem:**
  - `Map` supports arbitrary keys (including object references, functions, numbers, dates), and preserves insertion order.
  - `Set` stores unique values of any type and preserves insertion order.
  - Standard JSON cannot represent either (plain objects only support string/symbol keys).
- **Encoding Collections:**
  - `Map`: Tag as `["$Map", Array.from(map.entries()).map(([k, v]) => [encode(k), encode(v)])]`.
  - `Set`: Tag as `["$Set", Array.from(set.values()).map(v => encode(v))]`.
- **Cyclic Reference Detection with Collections:**
  - `Map` and `Set` instances can participate in circular references (e.g. `map.set('self', map)`).
  - Must include `Map` and `Set` references in the active cycle-detection `visitedSet` during recursion.

---

## 🧠 Mental Model

Think of **Preserving Ordered Registries in Transit**:
- Standard JSON only knows unordered string-to-value keypads (`{}`).
- A `Map` is an ordered ledger where both the left column (key) and right column (value) can be rich objects.
- **Superjson II serializes the ledger into an ordered array of rows**:
  - `Map` becomes an ordered list of `[encodedKey, encodedValue]` pairs.
  - `Set` becomes an ordered list of `encodedValue` items.
  - Deserialization instantiates `new Map(pairs)` and `new Set(items)`, guaranteeing exact entry insertion order and object key restoration.

---

## 🔑 Key Concepts

- ES6 `Map` and `Set` data model
- Object-as-key serialization in hash maps
- Collection iteration and order preservation (`Map.prototype.entries()`, `Set.prototype.values()`)
- Cycle detection across heterogenous collection boundaries

---

## ⚠️ Edge Cases / Traps

- **Object Keys in `Map`:** A map key can be an object: `map.set({ id: 1 }, 'admin')`. The key itself must be recursively encoded and decoded, not stringified into `"[object Object]"`.
- **Cycles inside Maps / Sets:** A `Map` containing itself as a key or value (`map.set('self', map)`) must trigger a `TypeError` for cyclic references. Add the collection to the cycle detector before visiting its entries!
- **Insertion Order Invariant:** `new Map()` and `new Set()` guarantee iteration matches insertion order. The serialized intermediate format must preserve this linear ordering (e.g., using Arrays).
- **Nested Collections:** A `Map` containing a `Set` containing a `Map` must recurse cleanly across all layers.

---

## ⭐ Interview Takeaway

1. **Pairs Array for Map Serialization:** Always serialize a `Map` as an array of `[key, value]` entry tuples. This preserves both non-string keys and insertion order.
2. **Cycle Detection on All Containers:** Every container type (Plain Object, Array, Map, Set) must participate in cycle detection tracking.
3. **Round-Trip Fidelity:** In full-stack TypeScript (tRPC, Next.js Server Actions), native `Map` and `Set` support eliminates manual conversion boilerplate between server and client.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why can't `Map` be serialized directly as a plain JSON object?
- How do you preserve object references used as `Map` keys across serialization?
- How do you detect circular references inside a `Set`?

### Follow-up Questions
- How does `WeakMap` differ from `Map` in terms of serialization feasibility? (WeakMaps are non-enumerable and cannot be serialized).
- How would you handle NaN keys in `Map`? (`Map` uses SameValueZero equality, treating all `NaN` keys as identical).
- If two different map keys refer to the same object in memory, how would you ensure they remain the same reference after deserialization?

### Conceptual Questions
- What is the difference between `Map` key equality (SameValueZero) and `===`?
- Why does JSON.stringify drop `Map` and `Set` completely by default?

---

## 🔄 Variations

- **Superjson I:** Primitives, Date, RegExp, BigInt without Map/Set.
- **Reference-Preserving Serializer:** Restoring identical memory references for shared object keys.
- **Streaming Collection Serializer:** Emitting NDJSON / chunks for streaming large Maps/Sets.

---

## 📝 Revision Notes

- **Core idea:** Represent `Map` as `["$Map", [[encK, encV], ...]]` and `Set` as `["$Set", [encV, ...]]`.
- **Remember:** Map keys can be objects; recurse on both keys and values.
- **Watch out for:** Add Maps and Sets to the cycle-detection set to prevent infinite loops.
- **Complexity:** Time: $O(N)$ where $N$ is total elements across all nested collections; Space: $O(N)$.

## Official Solution
## Superjson II ( Official solution )

Premium
Languages
Part II extends the same tagged-tree codec from [Superjson](/questions/javascript/superjson), but now collection structure is part of the metadata that must round-trip. `Map` and `Set` are not plain objects, so the encoded form has to record their ordered contents explicitly.

## Solution

The codec still has two phases:

1. Convert supported values into a JSON-safe intermediate tree.
2. Tag special values such as `undefined`, `Date`, `RegExp`, `Map`, and `Set`.
3. For `Map`, store an ordered list of encoded `[key, value]` pairs.
4. For `Set`, store an ordered list of encoded values.
5. After `JSON.parse()`, decode the tagged objects back into fresh JavaScript values.

All of the part I tags still work the same way. The new tags only add collection metadata:

| Runtime value | Encoded metadata | Decoded value |
| --- | --- | --- |
| `Map` | ordered `entries` array of encoded `[key, value]` pairs | fresh `Map` |
| `Set` | ordered `values` array of encoded values | fresh `Set` |

The important detail is that `Map` keys are real values too. A key can be an object, array, date, regex, or nested collection, so the key must go through the same recursive encode/decode path as the value. Encoding a map as a plain object would stringify keys and lose that information.

The traversal order also matters. JavaScript `Map` and `Set` preserve insertion order, so the encoded arrays should be emitted in iteration order and decoded in the same order. That lets collection order round-trip without adding extra indexes.

Cycle detection uses the same active recursion `stack` as part I. A collection is added before its entries are encoded and removed in a `finally` block, which rejects circular structures without leaving stale state after an error.

```jsx
const TAG_KEY = '$superjson';

function isPlainObject(value) {
  if (typeof value !== 'object' || value === null) {
    return false;
  }

  const prototype = Object.getPrototypeOf(value);
  return prototype === Object.prototype || prototype === null;
}

function makeTag(type, extra = {}) {
  return {
    [TAG_KEY]: type,
    ...extra,
  };
}

function encode(value, stack = new Set()) {
  if (
    value === null ||
    typeof value === 'boolean' ||
    typeof value === 'string'
  ) {
    return value;
  }

  if (typeof value === 'number') {
    if (Number.isNaN(value)) {
      return makeTag('NaN');
    }

    if (value === Infinity) {
      return makeTag('Infinity');
    }

    if (value === -Infinity) {
      return makeTag('-Infinity');
    }

    return value;
  }

  if (value === undefined) {
    return makeTag('undefined');
  }

  if (typeof value === 'bigint') {
    return makeTag('BigInt', { value: String(value) });
  }

  if (typeof value === 'function' || typeof value === 'symbol') {
    throw new TypeError('Unsupported value type');
  }

  if (value instanceof Date) {
    return makeTag('Date', { value: value.toISOString() });
  }

  if (value instanceof RegExp) {
    return makeTag('RegExp', {
      source: value.source,
      flags: value.flags,
    });
  }

  if (typeof value !== 'object') {
    throw new TypeError('Unsupported value type');
  }

  if (stack.has(value)) {
    throw new TypeError('Circular references are not supported');
  }

  // Track only the active recursion chain so shared references are allowed,
  // but cycles are rejected.
  stack.add(value);

  try {
    if (value instanceof Map) {
      const entries = [];

      // Map keys go through the same codec, so object and Date keys round-trip.
      for (const [key, entryValue] of value.entries()) {
        entries.push([encode(key, stack), encode(entryValue, stack)]);
      }

      return makeTag('Map', { entries });
    }

    if (value instanceof Set) {
      const values = [];

      for (const entryValue of value.values()) {
        values.push(encode(entryValue, stack));
      }

      return makeTag('Set', { values });
    }

    if (Array.isArray(value)) {
      return value.map((item, index) => {
        if (!(index in value)) {
          throw new TypeError('Sparse arrays are not supported');
        }

        return encode(item, stack);
      });
    }

    if (!isPlainObject(value)) {
      throw new TypeError('Only plain objects are supported');
    }

    if (Object.getOwnPropertySymbols(value).length > 0) {
      throw new TypeError('Symbol keys are not supported');
    }

    const encodedObject = {};

    for (const [key, entry] of Object.entries(value)) {
      encodedObject[key] = encode(entry, stack);
    }

    return encodedObject;
  } finally {
    stack.delete(value);
  }
}

function decode(value) {
  if (
    value === null ||
    typeof value === 'boolean' ||
    typeof value === 'number' ||
    typeof value === 'string'
  ) {
    return value;
  }

  if (Array.isArray(value)) {
    return value.map((item) => decode(item));
  }

  // Tagged objects are the escape hatch for values JSON cannot represent natively.
  const tag = value[TAG_KEY];

  if (typeof tag === 'string') {
    switch (tag) {
      case 'undefined':
        return undefined;
      case 'NaN':
        return NaN;
      case 'Infinity':
        return Infinity;
      case '-Infinity':
        return -Infinity;
      case 'BigInt':
        return BigInt(String(value.value));
      case 'Date':
        return new Date(String(value.value));
      case 'RegExp':
        return new RegExp(String(value.source), String(value.flags));
      case 'Map': {
        const encodedEntries = value.entries;

        if (!Array.isArray(encodedEntries)) {
          throw new TypeError('Invalid serialized value');
        }

        const map = new Map();

        for (const encodedEntry of encodedEntries) {
          if (!Array.isArray(encodedEntry) || encodedEntry.length !== 2) {
            throw new TypeError('Invalid serialized value');
          }

          map.set(decode(encodedEntry[0]), decode(encodedEntry[1]));
        }

        return map;
      }
      case 'Set': {
        const encodedValues = value.values;

        if (!Array.isArray(encodedValues)) {
          throw new TypeError('Invalid serialized value');
        }

        const set = new Set();

        for (const encodedEntry of encodedValues) {
          set.add(decode(encodedEntry));
        }

        return set;
      }
      default:
        throw new TypeError('Invalid serialized value');
    }
  }

  const decodedObject = {};

  for (const [key, entry] of Object.entries(value)) {
    decodedObject[key] = decode(entry);
  }

  return decodedObject;
}

/**
 * @param {unknown} value
 * @returns {string}
 */
export function serialize(value) {
  return JSON.stringify(encode(value));
}

/**
 * @param {string} serialized
 * @returns {unknown}
 */
export function deserialize(serialized) {
  return decode(JSON.parse(serialized));
}
```

## Common pitfalls

- **Encoding Map as an object:** `Object.fromEntries(map)` loses non-string key identity and cannot distinguish some keys after string coercion. Store an ordered entries array and encode each key recursively.
- **Encoding only Map values:** Map keys can be supported values too, including objects, dates, regexes, and nested collections. Decode both sides of each `[key, value]` pair before calling `map.set()`.
- **Losing insertion order:** Do not sort entries or values. Use the collection's normal iteration order when encoding, and insert decoded entries back in that same order.
- **Treating Set as an array:** A set's encoded `values` array is metadata for reconstruction, not the final type. The decoder should create a fresh `Set` and add decoded values into it.
- **Leaving values in the recursion stack:** Remove each object or collection from the active stack when its branch finishes. That keeps cycle detection accurate after successful branches and after thrown errors.

## Scope

- `Map` keys can be objects, arrays, dates, regexes, or other supported values.
- `Set` should preserve insertion order.
- Nested `Map` and `Set` values should reuse the same recursive codec as plain objects and arrays.
- Collection instances, keys, and nested objects should all be recreated as fresh values during decoding.
- All part I values such as `undefined`, `NaN`, infinities, `BigInt`, `Date`, and `RegExp` should still round-trip inside collections.
- Sparse arrays, cyclic references, symbol keys, functions, symbols, and custom class instances are still rejected.

## Notes

- Decoding creates equivalent fresh collections and entries. The serialized format does not preserve object identity across multiple references.
- The decoder validates that `Map` entries are two-item arrays and that `Set` values are arrays before reconstructing the collections.
- The exact serialized string is not important. The round-trip guarantee is that `deserialize(serialize(value))` recreates the supported value structure.

## Techniques

- Recursion
- Tagged unions
- Collection serialization

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A candidate uses `Object.fromEntries(map)` as its intermediate representation. Which map proves that this can lose information even with only primitive keys?
