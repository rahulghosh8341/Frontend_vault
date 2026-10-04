---
title: Superjson
aliases:
  - Superjson
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/superjson"
companies:
  - "[[Anthropic]]"
  - "[[OpenAI]]"
pattern:
  - "[[Recursion]]"
concepts:
  - "[[Recursion]]"
  - "[[JSON.stringify]]"
  - "[[Data Serialization]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Superjson

> [!info] Problem
> Implement serialize and deserialize functions that can stringify values JSON.stringify() cannot

## Problem

## Superjson

`JSON.stringify()` works well for plain JSON data, but it loses information for values like `undefined`, `Date`, and `RegExp`.

Popular libraries like [SuperJSON](https://github.com/flightcontrolhq/superjson) and [devalue](https://github.com/sveltejs/devalue) were built to solve exactly this kind of problem.

Implement two functions:

- `serialize(value)`: returns a string.
- `deserialize(serialized)`: returns the original value represented by that string.

The exact serialized format is up to you.

For this question, support:

- All normal JSON values
- `undefined`
- `NaN`, `Infinity`, `-Infinity`
- `BigInt`
- `Date`
- `RegExp`

Supported values can appear anywhere inside dense arrays and plain objects.

## Examples

```javascript
const value = {
  createdAt: new Date('2026-01-01T00:00:00.000Z'),
  retries: Infinity,
  missing: undefined,
  matcher: /user-\d+/gi,
};

const serialized = serialize(value);
// Any string format is fine as long as deserialize(serialized)
// recreates an equivalent value.

const restored = deserialize(serialized);

restored.createdAt instanceof Date; // true
restored.retries; // Infinity
restored.missing; // undefined
restored.matcher.test('USER-42'); // true
```

```javascript
const value = [undefined, BigInt(42), { score: NaN }];
const restored = deserialize(serialize(value));

restored[0]; // undefined
restored[1]; // 42n
Number.isNaN(restored[2].score); // true
```

## Notes

- You may assume `deserialize` only receives strings produced by `serialize`.
- `serialize` should throw a `TypeError` for functions, symbols, symbol keys, `Map`, `Set`, cyclic references, sparse arrays, and custom class instances.
- Malformed serialized strings are out of scope.

## Resources

- [SuperJSON](https://github.com/flightcontrolhq/superjson)
- [devalue](https://github.com/sveltejs/devalue)

## Hints

### Hint 1 : What information would JSON erase?

### Hint 2 : Can one codec work at every depth?

### Hint 3 : Is a shared reference necessarily cyclic?

## Asked at these companies

Anthropic
OpenAI

## 🤔 Thought Process

- **Immediate Recognition:** Custom serialization codec / data hydration engine designed to overcome the fundamental data loss of `JSON.stringify` (`undefined`, `Date`, `RegExp`, `BigInt`, `NaN`, `Infinity`).
- **Core Problem:**
  - Standard JSON erases type metadata: `NaN` -> `null`, `Infinity` -> `null`, `undefined` -> omitted / `null`, `BigInt` -> throws `TypeError`, `Date` -> ISO string, `RegExp` -> `{}`.
  - We need a two-way round-trip: `deserialize(serialize(val)) === equivalent val`.
- **Architecture (Tagged Tree / Envelope Pattern):**
  - Transform the original arbitrary data into an intermediate JSON-safe envelope structure carrying explicit type tags.
  - Example: `NaN` -> `["$NaN"]`, `undefined` -> `["$undefined"]`, `Date` -> `["$Date", date.toISOString()]`, `BigInt` -> `["$BigInt", bigint.toString()]`, `RegExp` -> `["$RegExp", regex.source, regex.flags]`.
- **Validation & Unsupported Types:**
  - `serialize` must throw `TypeError` on functions, symbols, cyclic references, custom class instances, `Map`, `Set`, and sparse arrays.
  - Track seen objects in a `Set` (or `WeakSet`) during recursion to detect cycles and throw `TypeError`.

---

## 🧠 Mental Model

Think of **Custom Envelope Packaging for Courier Transit**:
- Standard JSON is a flat postal envelope that only accepts basic text and numbers.
- If you hand it an exotic item (like a Date or BigInt), it either crushes it or rejects it.
- **Superjson acts as a specialized packer:**
  - Before shipping, it boxes exotic items into tagged standardized containers with packing slips: `[TYPE_TAG, PAYLOAD]`.
  - The postal service (JSON.stringify) transports the standardized boxes safely.
  - The receiver (deserialize) unboxes each tagged container, reads the packing slip, and reconstructs the authentic original object.

---

## 🔑 Key Concepts

- Data serialization / deserialization (codecs)
- Limitations and type loss of standard JSON
- Cycle detection via recursion tracking (`Set` / `WeakSet`)
- Envelope pattern / tagged union representation for runtime type restoration

---

## ⚠️ Edge Cases / Traps

- **Circular Reference Detection:** Objects referencing themselves or ancestors must throw `TypeError`, not recurse infinitely. Track visited objects in a `visitedSet` during the active traversal stack.
- **Sparse Arrays:** An array like `[1, , 3]` has holes (`!(1 in arr)`). The spec explicitly requires throwing `TypeError` on sparse arrays. Check `array.length` against `Object.keys(array).length`.
- **`BigInt` Serialization Crash:** Native `JSON.stringify` throws a `TypeError: Do not know how to serialize a BigInt`. Must convert BigInts before calling `JSON.stringify`.
- **`NaN` and Infinities in Objects:** `JSON.stringify({ a: NaN })` produces `{"a": null}`. Without pre-tagging, `NaN` collapses into `null` irreversibly.
- **Undefined in Arrays vs Objects:** In native JSON, `undefined` in an array becomes `null` (`[undefined]` -> `"[null]"`), while in an object the key is omitted (`{ a: undefined }` -> `"{}"`). A tagged tree preserves both explicitly.

---

## ⭐ Interview Takeaway

1. **Never Stringify Directly:** Don't attempt to serialize complex objects with custom replacer functions directly inside `JSON.stringify`. Transform the object graph into a clean, tagged intermediate representation first.
2. **Cycle Tracking via Stack:** Add the object to a `Set` when entering, remove it when leaving (or use a shared set if shared DAG references are permitted vs cycles).
3. **Real-World Impact:** This exact pattern powers modern Next.js/tRPC data loaders, Redux DevTools, and SvelteKit `devalue`.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why does `JSON.stringify` fail for `BigInt`, `undefined`, and `Date`?
- How do you detect circular references during object serialization?
- How do you detect whether an array is sparse?

### Follow-up Questions
- How would you extend this to support `Map` and `Set`? (See [[Superjson II]])
- How does `devalue` or `SuperJSON` preserve shared object references (same instance referenced multiple times without duplication)?
- How would you handle custom user classes with serialization hooks (`toJSON` / `fromJSON`)?

### Conceptual Questions
- What is the difference between structured clone algorithm (`structuredClone`) and JSON serialization?
- Why can't functions and symbols be safely serialized across process boundaries?

---

## 🔄 Variations

- **Superjson II:** Adding support for `Map` and `Set` collections with arbitrary keys.
- **Cyclic Graph Serializer (devalue):** Generating index-referenced graph nodes to serialize cyclic and shared structures without throwing.
- **Binary Codec (MessagePack / BSON):** Serializing data into compact binary buffers instead of JSON strings.

---

## 📝 Revision Notes

- **Core idea:** Recursively map exotic values to tagged tuples before `JSON.stringify`; unpack tagged tuples during deserialization.
- **Remember:** Throw `TypeError` on circular references, functions, symbols, and sparse arrays.
- **Watch out for:** Tag `NaN`, `Infinity`, `-Infinity`, and `undefined` before standard JSON erases them.
- **Complexity:** Time: $O(N)$ where $N$ is total object nodes; Space: $O(N)$ for intermediate tagged tree.

## Official Solution
## Superjson ( Official solution )

Premium
Languages
Here the important part is metadata round-tripping. A common trap is to call `JSON.stringify()` on raw values and hope to recover them later, but by then `undefined`, `NaN`, infinities, `Date`, and `RegExp` have already lost information. The solution first converts the value into a JSON-safe tagged tree that carries enough metadata to rebuild those runtime values later.

## Solution

`serialize()` and `deserialize()` are mirror-image tree walks:

1. `encode()` recursively walks the input value.
2. Normal JSON primitives stay unchanged.
3. Special values are replaced with plain objects containing a `$superjson` tag and any metadata needed for reconstruction.
4. `serialize()` calls `JSON.stringify()` on that encoded tree.
5. `deserialize()` calls `JSON.parse()`, then `decode()` recursively turns tagged objects back into JavaScript values.

Keep a strict codec boundary: `JSON.stringify()` only sees normal JSON data because supported non-JSON runtime values have already been replaced with tagged objects, while unsupported shapes throw during encoding.

Every non-JSON runtime value must map to a plain JSON shape with enough metadata to rebuild it later:

| Runtime value | Encoded metadata | Decoded value |
| --- | --- | --- |
| `undefined` | tag only | `undefined` |
| `NaN` | tag only | `NaN` |
| `Infinity` / `-Infinity` | tag only | the matching infinity |
| `BigInt` | decimal string | `BigInt(...)` |
| `Date` | ISO timestamp string | fresh `Date` instance |
| `RegExp` | `source` and `flags` | fresh `RegExp` instance |

Arrays and plain objects reuse the same recursive codec for their entries. That is what lets special values work at the top level, inside dense arrays, and inside nested objects without separate cases for each container.

Unsupported shapes are rejected during encoding. This implementation throws for functions, symbols, symbol keys, `Map`, `Set`, sparse arrays, cyclic references, and custom class instances. The active recursion `stack` is only used to detect cycles on the current path, so separate acyclic branches are allowed even if they contain equal-looking data.

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

  if (value instanceof Map || value instanceof Set) {
    throw new TypeError('Map and Set are not supported in this version');
  }

  if (typeof value !== 'object') {
    throw new TypeError('Unsupported value type');
  }

  // Track the active recursion path so cycles throw without rejecting separate branches.
  if (stack.has(value)) {
    throw new TypeError('Circular references are not supported');
  }

  stack.add(value);

  try {
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
    // Always pop this object off the path, even if a nested encode throws.
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

  // Tagged objects are the escape hatch for values JSON cannot represent directly.
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

- **Calling JSON.stringify too early:** `JSON.stringify()` transforms some values before they can be recovered: object properties with `undefined` disappear, array `undefined` becomes `null`, `NaN` and infinities become `null`, and top-level `undefined` does not produce a JSON string. Encode the tree first, then stringify the encoded tree.
- **Not storing enough metadata:** `Date` needs its timestamp, `RegExp` needs both `source` and `flags`, and `BigInt` should be stored as a string so large integers are not rounded through `number`.
- **Forgetting top-level values:** The recursive codec should handle `serialize(undefined)`, `serialize(NaN)`, and `serialize(/x/g)` the same way it handles those values inside an object. Do not rely on object traversal to discover special cases.

### Using a permanent seen set for cycles

Cycle detection should track the active recursion path and remove the value again when that branch finishes. Otherwise, shared but acyclic values can be rejected even though the prompt only excludes circular references.

## Scope

- `undefined` must survive in objects, arrays, and even at the top level.
- `NaN` and infinities need explicit tags because JSON would otherwise lose them.
- `BigInt` should be stored as a string and recreated during decoding.
- `Date` and `RegExp` need enough metadata to build a fresh instance later.
- Sparse arrays should throw instead of being silently converted to `null` values.
- Custom class instances are out of scope; only plain objects are encoded as objects.

## Notes

- The decoder creates fresh `Date` and `RegExp` instances. It recreates equivalent values, not object identity.
- `deserialize()` is expected to receive strings produced by `serialize()`, but unknown tags are still treated as invalid serialized values.
- This first version intentionally rejects `Map` and `Set`; that collection metadata is added in [Superjson II](/questions/javascript/superjson-ii).
- The active-path stack is a cycle detector, not a cache. Removing a value from the stack after its branch finishes lets separate acyclic branches be encoded normally.

## Techniques

- Recursion
- Tagged unions
- Serialization

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A BigInt tag stores `Number(value)` and decoding calls `BigInt` on that number. Small integer tests pass. Which test reveals why the payload must preserve exact decimal text?
