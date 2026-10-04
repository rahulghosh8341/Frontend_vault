---
title: "JSON.stringify II"
aliases:
  - "jsonStringifyII"
  - "JSON.stringify II"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# JSON.stringify II

> [!info] Problem
> Implement a function that converts a JavaScript value into a JSON string

## Problem

## JSON.stringify II

Zhenghao He
Engineering Manager, Robinhood
Implement a `jsonStringify` function that converts a JavaScript value into a JSON string, similar to [`JSON.stringify`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

- You may ignore the [second and third](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#parameters) optional parameters in the original API.
- The function should behave exactly like `JSON.stringify()` for any data type. Refer to the examples below.
- Other cases:
  - Cyclic references: throw `TypeError('Converting circular structure to JSON')`.
  - `BigInt`: throw `TypeError('Do not know how to serialize a BigInt')`.

## Behavior guide

The differences between top-level values, object properties, and array elements are easy to mix up. Use this table as the core behavior contract:

| Value kind | Top-level value | Object property value | Array element value |
| --- | --- | --- | --- |
| `undefined`, `Symbol`, or function | Return `undefined` | Omit the property | Serialize as `null` |
| `NaN`, `Infinity`, or `-Infinity` | Serialize as `null` | Serialize as `null` | Serialize as `null` |
| `Date` or custom `toJSON()` object | Serialize the `toJSON()` result | Serialize the `toJSON()` result | Serialize the `toJSON()` result |
| `BigInt` | Throw a `TypeError` | Throw a `TypeError` | Throw a `TypeError` |
| Cyclic object or array | Throw a `TypeError` | Throw a `TypeError` | Throw a `TypeError` |

## Examples

```javascript
jsonStringify({ foo: 'bar' }); // '{"foo":"bar"}'
jsonStringify({ foo: 'bar', bar: [1, 2, 3] }); // '{"foo":"bar","bar":[1,2,3]}'
```

Other types and their expected behavior:

```javascript
jsonStringify(); // undefined
jsonStringify(undefined); // undefined
jsonStringify(null); // 'null'
jsonStringify(true); // 'true'
jsonStringify(false); // 'false'
jsonStringify(1); // '1'
jsonStringify(Infinity); // 'null'
jsonStringify(NaN); // 'null'
jsonStringify('foo'); // '"foo"'
jsonStringify('"foo"') === '"\\"foo\\""'; // Double quotes present in the original input are escaped using backslashes
jsonStringify(Symbol('foo')); // undefined
jsonStringify(() => {}); // undefined
jsonStringify(['foo', 'bar']); // '["foo","bar"]'
jsonStringify(/foo/); // '{}'
jsonStringify(new Map()); // '{}'
jsonStringify(new Set()); // '{}'
```

## Hints

### Hint 1 : Who decides what an unsupported value means?

### Hint 2 : Which strings pass through the same escaping rule?

### Hint 3 : Is every repeated reference a cycle?

## Asked at these companies

- [[Snap]]
- [[Amazon]]
- [[Google]]
- [[Netflix]]

## 🤔 Thought Process

- **Immediate Recognition:** Full-spec implementation of `JSON.stringify` including cycle detection, type exclusions, `.toJSON()` hooks, and `BigInt` errors.
- **Crucial Spec Behaviors:**
  - `BigInt`: Throws `TypeError("Do not know how to serialize a BigInt")`.
  - Cyclic Objects: Throws `TypeError("Converting circular structure to JSON")`. Track ancestor chain using a `Set`.
  - `.toJSON()` Method: If an object (e.g. `Date`) has a `.toJSON()` method, invoke it first before serializing.
  - `undefined`, `Function`, `Symbol`:
    - At top-level: return `undefined` (raw, not string `"undefined"`).
    - Inside Array: convert to `"null"`.
    - Inside Object: omit the key entirely.
  - `NaN`, `Infinity`, `-Infinity`: Serialize as `"null"`.
  - Number, Boolean, String wrapper objects (`new Number(1)`): Unbox to primitive values via `.valueOf()`.

---

## 🧠 Mental Model

Think of **Ancestor-Chain Graph Traversal with Serialization Pipeline**:
```
Value
  │
  ├─► Has .toJSON()? ──────► Call it and replace value
  ├─► Is BigInt? ──────────► THROW TypeError
  ├─► In Ancestor Set? ────► THROW TypeError (Circular Reference)
  ├─► Primitive / Wrapper ─► Serialize directly or unbox
  ├─► Array ───────────────► Push to Set, map elements (undefined -> null), pop from Set
  └─► Plain Object ────────► Push to Set, filter omit-types, map entries, pop from Set
```

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Recursion]]
- [[Type Checking]]
- [[Set Lookup]]
- Cycle detection using active ancestor `Set` (backtracking)
- Spec-compliant type suppression (`undefined`, `Function`, `Symbol`)
- Object unboxing (`valueOf`) and `.toJSON()` protocol

---

## ⚠️ Edge Cases / Traps

- **Ancestors vs Seen Cache:** For cycle detection in serialization, use a `Set` tracking the *current call stack ancestors*, adding before recursing into children and removing after (`ancestors.add(val)` ... `ancestors.delete(val)`). If you don't delete on exit, diamond dependencies (`a` references `b` and `c`, and both reference `d`) will falsely trigger circular reference errors!
- **`undefined` / Function / Symbol in Arrays vs Objects:**
  `JSON.stringify([undefined, () => {}])` -> `'[null,null]'`
  `JSON.stringify({ a: undefined, b: () => {} })` -> `'{ }'`
- **Top-level `undefined`:** `JSON.stringify(undefined)` returns `undefined` (not string `""` or `"undefined"`).
- **String / Number / Boolean Object Wrappers:** `new Number(3)` has `typeof === 'object'`. Must unbox using `Number(val)` or `val.valueOf()`.

---

## ⭐ Interview Takeaway

- **Active Ancestor Set (Cycle Detection):**
  Add object to `ancestorSet` before visiting children, delete from `ancestorSet` after returning (backtracking).
- **Three-way behavior for `undefined` / `Function` / `Symbol`:**
  - Top level: returns `undefined`.
  - In array: serialized as `'null'`.
  - In object: key-value pair omitted completely.
- Check for `BigInt` explicitly: `if (typeof val === 'bigint') throw new TypeError(...)`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does `JSON.stringify` throw on `BigInt`?" (JavaScript's `JSON.stringify` does not have a standard representation for arbitrary-precision integers, and auto-converting to number risks precision loss).
- "Why must cycle detection use an ancestor stack instead of a global visited set?" (A global visited set would mistakenly reject valid DAGs / diamond references).

### Follow-up Questions
- "How does `Date.prototype.toJSON()` fit into the serialization pipeline?" (If `typeof val.toJSON === 'function'`, `val.toJSON()` is invoked and its return value is serialized instead).
- "How would you implement the `replacer` function parameter?"

### Conceptual Questions
- "Why does `JSON.stringify` drop object properties whose values are `undefined` or functions?" (JSON is a data-interchange format designed for language interoperability; functions and `undefined` are JavaScript-specific runtime concepts).

---

## 🔄 Variations

- **JSON.stringify I:** Simplified variant without cycles or special type suppression.
- **Deep Clone II:** Cycle handling via `WeakMap` cache (preserving references rather than throwing).
- **Safe Stringify:** Cycle-safe serializer that replaces circular references with `'[Circular]'` instead of throwing.

---

## 📝 Revision Notes

- Complete cycle-safe implementation:
```javascript
export default function jsonStringify(value, ancestors = new Set()) {
  if (typeof value === 'bigint') {
    throw new TypeError('Do not know how to serialize a BigInt');
  }

  if (value !== null && typeof value === 'object' && typeof value.toJSON === 'function') {
    value = value.toJSON();
  }

  if (value === null) return 'null';
  if (typeof value === 'boolean') return String(value);
  if (typeof value === 'number') return Number.isFinite(value) ? String(value) : 'null';
  if (typeof value === 'string') return `"${value.replace(/\\/g, '\\\\').replace(/"/g, '\\"')}"`;

  if (typeof value === 'function' || typeof value === 'undefined' || typeof value === 'symbol') {
    return undefined;
  }

  if (value instanceof Number || value instanceof Boolean || value instanceof String) {
    return jsonStringify(value.valueOf(), ancestors);
  }

  if (ancestors.has(value)) {
    throw new TypeError('Converting circular structure to JSON');
  }

  ancestors.add(value);

  try {
    if (Array.isArray(value)) {
      const items = value.map(item => {
        const res = jsonStringify(item, ancestors);
        return res === undefined ? 'null' : res;
      });
      return `[${items.join(',')}]`;
    }

    if (typeof value === 'object') {
      const entries = [];
      for (const [key, val] of Object.entries(value)) {
        if (typeof val !== 'function' && typeof val !== 'undefined' && typeof val !== 'symbol') {
          const serializedVal = jsonStringify(val, ancestors);
          if (serializedVal !== undefined) {
            entries.push(`"${key}":${serializedVal}`);
          }
        }
      }
      return `{${entries.join(',')}}`;
    }
  } finally {
    ancestors.delete(value);
  }

  return undefined;
}
```

---

## Official Solution

## JSON.stringify II ( Official solution )

Premium
Zhenghao He
Engineering Manager, Robinhood
Languages

## Solution

This follow-up is mostly about mirroring `JSON.stringify`'s branching rules. The hard part is not the recursion itself, but knowing how the same value changes behavior depending on where it appears: top-level value, object property, or array element.

The serializer has one recursive property: `stringifyValue(value, stack)` either returns a valid JSON fragment string, returns `undefined` when the current value should disappear, or throws for invalid input such as `BigInt` or a cycle. Callers then interpret `undefined` according to their context.

### Handling data types

Start by classifying the current value and applying the same rules as `JSON.stringify`.

When the top-level value is `undefined`, a `Symbol`, or a function, `JSON.stringify` returns `undefined` rather than the string `'undefined'`:

```javascript
JSON.stringify(undefined); // undefined
JSON.stringify(Symbol('foo')); // undefined
JSON.stringify(() => {}); // undefined
```

For most other built-in object types such as `Map`, `Set`, `WeakMap`, `WeakSet`, and `RegExp`, `JSON.stringify` falls back to an empty object literal because they do not expose enumerable own properties:

```javascript
JSON.stringify(/foo/); // '{}'
JSON.stringify(new Map()); // '{}'
JSON.stringify(new Set()); // '{}'
```

`NaN` and `Infinity` become `null`. `Date` values serialize to ISO strings because `Date.prototype.toJSON()` runs first, and the same applies to any custom `toJSON()` method on the input.

### Cyclic references

The other major rule is that cyclic input should throw instead of recursing forever:

```javascript
const foo = {};
foo.a = foo;

JSON.stringify(foo); // ❌ Uncaught TypeError: Converting circular structure to JSON
```

One workable approach is to keep a `Set` of visited objects while traversing. As soon as the same object appears again along the traversal, the input has a cycle.

The stack should represent the current recursion path, not every object ever seen. Remove an object from the stack when its array or object serialization finishes, so the same object reference can appear in two separate branches without being mistaken for a circular parent-child link.

Once those type rules and cycle detection are in place, the remaining serializer is a straightforward recursive case split:

| Context | Child value | Recursive result | Caller behavior |
| --- | --- | --- | --- |
| top level | `undefined` | `undefined` | return `undefined` |
| object property | function | `undefined` | omit the key |
| array element | `Symbol('x')` | `undefined` | emit `null` to preserve length |
| object property | `NaN` | `'null'` | keep key with `null` |

That caller-specific handling is why arrays and objects cannot share one generic "skip falsy serialization results" path.

```jsx
const ESCAPE_REGEX = /[\u0000-\u001f"\\]/g;
const ESCAPE_SEQUENCES = {
  '"': '\\"',
  '\\': '\\\\',
  '\b': '\\b',
  '\f': '\\f',
  '\n': '\\n',
  '\r': '\\r',
  '\t': '\\t',
};

function escapeString(value) {
  return value.replace(ESCAPE_REGEX, (character) => {
    return (
      ESCAPE_SEQUENCES[character] ??
      `\\u${character.charCodeAt(0).toString(16).padStart(4, '0')}`
    );
  });
}

/**
 * @param {unknown} value
 * @param {Set<object>} stack
 * @return {string | undefined}
 */
function stringifyValue(value, stack) {
  if (typeof value === 'bigint') {
    throw new TypeError('Do not know how to serialize a BigInt');
  }

  if (value === null) {
    return 'null';
  }

  const type = typeof value;

  if (type === 'number') {
    if (Number.isNaN(value) || !Number.isFinite(value)) {
      return 'null';
    }

    return String(value);
  }

  if (type === 'boolean') {
    return String(value);
  }

  if (type === 'function' || type === 'undefined' || type === 'symbol') {
    return undefined;
  }

  if (type === 'string') {
    return `"${escapeString(value)}"`;
  }

  // Built-ins like Date and custom serializers get first crack at producing JSON-safe output.
  if (typeof value.toJSON === 'function') {
    return stringifyValue(value.toJSON(), stack);
  }

  if (
    value instanceof Number ||
    value instanceof String ||
    value instanceof Boolean
  ) {
    return stringifyValue(value.valueOf(), stack);
  }

  if (stack.has(value)) {
    throw new TypeError('Converting circular structure to JSON');
  }

  stack.add(value);

  try {
    if (Array.isArray(value)) {
      const arrayValues = [];

      // Arrays keep their length; unsupported entries become null rather than disappearing.
      for (let index = 0; index < value.length; index += 1) {
        arrayValues.push(stringifyValue(value[index], stack) ?? 'null');
      }

      return `[${arrayValues.join(',')}]`;
    }

    // Objects omit keys whose values serialize to undefined.
    const objectEntries = Object.entries(value)
      .map(([key, entryValue]) => {
        const serializedValue = stringifyValue(entryValue, stack);

        if (serializedValue === undefined) {
          return undefined;
        }

        return `"${escapeString(key)}":${serializedValue}`;
      })
      .filter((entry) => entry !== undefined);

    return `{${objectEntries.join(',')}}`;
  } finally {
    stack.delete(value);
  }
}

/**
 * @param {unknown} value
 * @return {string | undefined}
 */
export default function jsonStringify(value) {
  return stringifyValue(value, new Set());
}
```

## Edge cases

- `BigInt` should throw instead of being coerced.
- `Date` and custom `toJSON()` values get serialized through their returned value.
- Object properties whose values serialize to `undefined` are omitted, while array entries in the same category become `null`.
- Cycles should throw, but the same object reused in two sibling branches is not a cycle after the recursion stack unwinds.
- String escaping has to cover quotes, backslashes, common control characters, and other control codes via `\uXXXX`.

## Notes

- This question still ignores `JSON.stringify`'s optional `replacer` and `space` parameters.
- One possible follow-up is to make it faster. The current implementation involves frequent runtime type checks due to JavaScript's dynamic typing. One way to make this implementation of `JSON.stringify` faster is to have the user provide a schema of the object (e.g. using [JSON Schema](https://json-schema.org/)) so the object structure is known before serialization. This can avoid a lot of runtime guesswork. In fact, many `JSON.stringify`-alternative libraries are implemented this way to make serialization faster. One example is [fast-json-stringify](https://github.com/fastify/fast-json-stringify).

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A serializer removes every child whose recursive result is `undefined` from both arrays and objects. For `{ absent: undefined, items: [undefined, 7] }`, explain which removal is valid and which changes the meaning of the array. What should the two parent containers emit?

Your notes (optional)
