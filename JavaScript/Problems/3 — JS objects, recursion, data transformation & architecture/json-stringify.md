---
title: "JSON.stringify"
aliases:
  - "jsonStringify"
  - "JSON.stringify"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# JSON.stringify

> [!info] Problem
> Implement a function that converts a JavaScript value into a JSON string

## Problem

## JSON.stringify

Implement a `jsonStringify` function that converts a JavaScript value into a JSON string, similar to [`JSON.stringify`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify).

- Only JSON-serializable values (i.e. boolean, number, `null`, array, object) will be present in the input value.
- Ignore the [second and third](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#parameters) optional parameters in the original API.

## Examples

```javascript
jsonStringify({ foo: 'bar' }); // '{"foo":"bar"}'
jsonStringify({ foo: 'bar', bar: [1, 2, 3] }); // '{"foo":"bar","bar":[1,2,3]}'
jsonStringify({ foo: true, bar: false }); // '{"foo":true,"bar":false}'
```

Other types:

```javascript
jsonStringify(null); // 'null'
jsonStringify(true); // 'true'
jsonStringify(false); // 'false'
jsonStringify(1); // '1'
jsonStringify('foo'); // '"foo"'
```

## Hints

### Hint : What should one recursive call return?

## Asked at these companies

- [[Snap]]
- [[Amazon]]
- [[Google]]
- [[Meta]]
- [[Netflix]]
- [[Snowflake]]

## 🤔 Thought Process

- **Immediate Recognition:** Basic serialization of JSON-safe data types into an RFC 8259 JSON string.
- **Supported Types in Scope:**
  - Strings: Enclosed in double quotes with character escaping (`"hello"`).
  - Numbers: Stringified (`42`). `NaN` and `Infinity` serialize to `"null"`.
  - Booleans: `"true"` or `"false"`.
  - `null`: `"null"`.
  - Arrays: Serialized elements enclosed in brackets `[ ... ]` joined by commas.
  - Objects: Serialized key-value pairs `"key":value` enclosed in braces `{ ... }` joined by commas.
- **Base Case vs Recursive Step:**
  - Primitives: Direct serialization.
  - Containers: Map over elements (arrays) or entries (objects) and join with `","`.
- **String Escaping:** Must wrap strings in double quotes `"` and escape internal double quotes and control characters.

---

## 🧠 Mental Model

Think of **Recursive Abstract Syntax Tree Serialization**:
- A JSON document is a tree where internal nodes are `Array` or `Object` containers and leaf nodes are primitive values.
- Serialize from the leaves up: convert leaf values to their literal JSON text representation, then wrap child representations into container delimiters (`[]` or `{}`).

---

## 🔑 Key Concepts

- [[Recursion]]
- [[DFS Recursion]]
- [[Type Checking]]
- String serialization & escaping rules
- `NaN` and `Infinity` coercion to `null`

---

## ⚠️ Edge Cases / Traps

- **`NaN`, `Infinity`, `-Infinity`:** In JavaScript JSON serialization, non-finite numbers must be stringified as `"null"`, NOT `"NaN"`.
- **`null` vs Object Check:** `typeof null === 'object'`. Check for `null` before checking for object properties.
- **Strings with Quotes:** Strings must be enclosed in double quotes. Any internal `"` or `\` characters must be properly escaped (`"`, `\\`).
- **Empty Containers:** Empty array serializes to `"[]"`; empty object serializes to `"{}"`.

---

## ⭐ Interview Takeaway

- Distinguish primitives from containers immediately:
  - Strings: `JSON.stringify` or `"` + escaped + `"`
  - Numbers: `Number.isFinite(val) ? String(val) : "null"`
  - Booleans: `String(val)`
  - Arrays: `'[' + val.map(jsonStringify).join(',') + ']'`
  - Objects: `'{' + Object.keys(val).map(k => '"' + k + '":' + jsonStringify(val[k])).join(',') + '}'`

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does `JSON.stringify(NaN)` return `'null'` instead of `'NaN'`?" (The JSON standard RFC 8259 does not support `NaN` or `Infinity`; it only supports finite decimal numbers).
- "How are strings formatted in valid JSON?" (Strings must be delimited by double quotes `"`, never single quotes).

### Follow-up Questions
- "What happens when the object has circular references?" (Throws a `TypeError: Converting circular structure to JSON`; handled in JSON.stringify II).
- "How does `JSON.stringify` handle functions, `undefined`, and `Symbol`?" (Omitted in objects; converted to `null` in arrays).

### Conceptual Questions
- "What are the second (`replacer`) and third (`space`) arguments in native `JSON.stringify`?" (`replacer` filters or transforms values; `space` controls indentation for pretty-printing).

---

## 🔄 Variations

- **JSON.stringify II:** Full implementation with cycle detection, `BigInt` handling, and function/undefined filtering.
- **JSON.parse:** Implementing the deserializer parser.
- **Superjson:** Serializing non-standard types like `Date`, `RegExp`, and `Set`.

---

## 📝 Revision Notes

- Fast, clean implementation:
```javascript
export default function jsonStringify(value) {
  if (value === null) return 'null';
  if (typeof value === 'boolean') return String(value);
  if (typeof value === 'number') return Number.isFinite(value) ? String(value) : 'null';
  if (typeof value === 'string') return `"${value.replace(/\\/g, '\\\\').replace(/"/g, '\\"')}"`;

  if (Array.isArray(value)) {
    return `[${value.map(item => jsonStringify(item)).join(',')}]`;
  }

  if (typeof value === 'object') {
    const entries = Object.keys(value).map(
      key => `"${key}":${jsonStringify(value[key])}`
    );
    return `{${entries.join(',')}}`;
  }

  return undefined;
}
```

---

## Official Solution

## JSON.stringify ( Official solution )

Premium
Languages

## Solution

Because non-primitive values can contain both primitive and non-primitive values, use a recursive solution. Each recursive call returns the complete JSON text for one value, so parent arrays and objects only need to join already-stringified children.

That "child returns a JSON fragment" rule is what prevents comma and brace logic from leaking across recursion levels.

The values form a tree. Primitive values are nodes that do not have children, and array/object values are nodes that have children of any value type. Convert each node into a string by converting each of its children into strings. Work upward from the "leaf" nodes, which in this case are primitive values because they cannot contain children, and build the string up from the leaf nodes all the way to the root.

Define how to stringify each value type:

- **`null`**: Directly convert it into the string `null`
- **Boolean**: Directly convert `true`/`false` into a string via `String()`
- **Numbers**: Directly convert into a string via `String()`
- **Strings**: Wrap the value with double quotes because strings use double quotes
- **Arrays**: Recursively stringify each child item, then concatenate them with a comma, and wrap them in square brackets: `[` and `]`
- **Objects**: Convert each key/value pair (also called an entry) into a `"key":{stringifiedValue}` format by recursively stringifying the values, concatenate them with a comma, and wrap them in braces: `{` and `}`

After defining how to stringify each value, determine the value type. Since `null`, booleans, and numbers can produce the desired string just by using `String()`, handle them together as the default case at the bottom. To determine the types:

- **Arrays**: Use `Array.isArray()`
- **Objects**: Use `typeof value === 'object' && value !== null`. The check for `!== null` is important because `typeof null` is `'object'` but `null` values must be handled differently from objects
- **Strings**: Use `typeof value === 'string'`

Here's the code that determines the type of a value and stringifies each value type appropriately.

For `{ foo: true, bar: [1, 'x'] }`, the recursion works from leaves back to the root:

| Value | Stringified result |
| --- | --- |
| `true` | `true` |
| `1` | `1` |
| `'x'` | `"x"` |
| `[1, 'x']` | `[1,"x"]` |
| whole object | `{"foo":true,"bar":[1,"x"]}` |

```jsx
/**
 * @param {unknown} value
 * @returns {string}
 */
export default function jsonStringify(value) {
  if (Array.isArray(value)) {
    const arrayValues = value.map((item) => jsonStringify(item));
    return `[${arrayValues.join(',')}]`;
  }

  if (typeof value === 'object' && value !== null) {
    // Object.entries preserves key insertion order for plain objects.
    const objectEntries = Object.entries(value).map(
      ([key, value]) => `"${key}":${jsonStringify(value)}`,
    );
    return `{${objectEntries.join(',')}}`;
  }

  if (typeof value === 'string') {
    return `"${value}"`;
  }

  return String(value);
}
```

Here's an alternative that does the type checking in a cleaner fashion by using `switch` cases:

```jsx
function getType(value: unknown): string {
  if (value === null) {
    return 'null';
  }

  // Split arrays out before falling back to typeof's coarser buckets.
  if (Array.isArray(value)) {
    return 'array';
  }

  return typeof value;
}

export default function jsonStringify(value: unknown): string {
  const type = getType(value);

  switch (type) {
    case 'array': {
      const arrayValues = (value as Array<unknown>)
        .map((item) => jsonStringify(item))
        .join(',');
      return `[${arrayValues}]`;
    }
    case 'object': {
      // Object.entries preserves key insertion order for plain objects.
      const objectValues = Object.entries(value as Record<string, unknown>)
        .map(([key, value]) => `"${key}":${jsonStringify(value)}`)
        .join(',');
      return `{${objectValues}}`;
    }
    case 'string':
      return `"${value as string}"`;
    default:
      // Handles null, boolean, numbers.
      return String(value);
  }
}
```

## Edge cases

- Empty arrays and empty objects should serialize to `[]` and `{}` without extra commas.
- Array order and object entry order should be preserved while recursively stringifying nested values.
- Mixed nesting works because arrays and objects reuse the same recursive rules for their children.
- Keys are quoted at the object-entry level, while values are delegated to the recursive function. That keeps key formatting separate from value formatting.

## Limitations

The code above is a simplified version of `JSON.stringify` and does not handle many other JavaScript types or support the API's replacer and formatting options. Other unsupported cases include:

- Cyclic references within objects.
- Types such as `undefined`, `Function`, `Map`, `Set`, `Symbol`, `RegExp`, `Date`, and more.
- Double quotes and other special characters, such as backslashes and tabs, within strings should be escaped.

To practice handling such cases, try out [`JSON.stringify` II](/questions/javascript/json-stringify-ii).

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
The simplified serializer wraps a string in double quotes without escaping its contents. Explain why a test using only `'hello'` does not justify using it on arbitrary user text. Give an input containing a quote or newline that needs more than surrounding quotes, and state what a round-trip test should check.

Your notes (optional)
