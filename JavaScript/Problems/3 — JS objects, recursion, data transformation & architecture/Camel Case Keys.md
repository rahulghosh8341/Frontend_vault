---
title: "Camel Case Keys"
aliases:
  - "camelCaseKeys"
  - "Camel Case Keys"
difficulty: "Medium"
source: GreatFrontEnd
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Camel Case Keys

> [!info] Problem
> Implement a function to convert all the keys in an object to camel case

## Problem

## Camel Case Keys

Zhenghao He
Engineering Manager, Robinhood
Implement a function `camelCaseKeys` that takes an object and returns a new object with all its keys converted to camel case.

Camel case is a format where word boundaries are represented by capitalizing each word after the first, and the first letter is lowercase. Some examples:

| String | camelCase |
| --- | --- |
| `foo` | Yes |
| `fooBar` | Yes |
| `Foo_Bar` | No |
| `foo_bar` | No |

For simplicity, only the four string formats above need to be considered; there will be no keys containing spaces, hyphens, or PascalCase.

## Examples

```javascript
camelCaseKeys({ foo_bar: true });
// { fooBar: true }

camelCaseKeys({ foo_bar: true, bar_baz: { baz_qux: '1' } });
// { fooBar: true, barBaz: { bazQux: '1' } }

camelCaseKeys([{ baz_qux: true }, { foo: true, bar: [{ foo_bar: 'hello' }] }]);
// [{ bazQux: true }, { foo: true, bar: [{ fooBar: 'hello' }] }]
```

## Notes

You can assume the input is always a valid, plain JavaScript object or array.

## Hints

### Hint 1 : Which nested values need different treatment?

### Hint 2 : How does one supported key change?

## 🤔 Thought Process

- **Immediate Recognition:** Deep object/array transformation problem combining string manipulation with recursive tree traversal.
- **Two Distinct Subproblems:**
  1. **String Conversion:** Convert an individual key string from `snake_case` or `Foo_Bar` into `camelCase`.
  2. **Structure Traversal:** Recursively traverse arrays and objects, renaming keys in objects and preserving array order.
- **String Conversion Details:**
  - Regex approach: `/_([a-zA-Z])/g` replacing match with `match[1].toUpperCase()`, plus lowercase the first letter.
  - Or splitting by `_`, lowering the first word, and capitalizing subsequent words.
- **Traversal Base Cases:**
  - Non-objects / `null`: Return directly.
  - Arrays: Map over elements and recursively call `camelCaseKeys`.
  - Objects: Iterate over keys, transform the key name, and assign the recursively transformed value.

---

## 🧠 Mental Model

Think of a **Contract Adapter / Data Normalizer**:
- Backend APIs frequently return database columns in `snake_case` (`created_at`, `user_id`).
- Frontend applications standardise on `camelCase` (`createdAt`, `userId`).
- This function acts as an API gateway boundary adapter that transforms the schema shape before passing to UI components.

---

## 🔑 Key Concepts

- [[Recursion]]
- String manipulation and Regular Expressions (`/[_]+([a-zA-Z0-9])/g`)
- Object traversal (`Object.keys()`, `Object.entries()`)
- API normalization layers (DTO transformation)

---

## ⚠️ Edge Cases / Traps

- **Leading / Trailing Underscores:** E.g., `_id` or `__v`. Ensure whether leading underscores denote internal fields that should be preserved or stripped.
- **Arrays of Objects:** Arrays themselves don't have camelCase keys, but their elements can be objects that must have their keys transformed.
- **`null` Check:** As always in JS, `typeof null === 'object'`.
- **First Character Capitalization:** If the input is `Foo_Bar`, the result should be `fooBar` (first character lowercased).

---

## ⭐ Interview Takeaway

- Separate string transformation from recursive object traversal into two clean helper functions.
- Handle arrays seamlessly: `if (Array.isArray(val)) return val.map(camelCaseKeys)`.
- Use regex replacement with a callback: `str.replace(/_([a-z])/gi, (_, c) => c.toUpperCase())` and lowercase the leading char.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you convert `snake_case` to `camelCase` using regex?"
- "How do you ensure arrays nested inside the object are also processed?"

### Follow-up Questions
- "What if keys are kebab-case (`foo-bar`) or PascalCase (`FooBar`)?"
- "How would you implement the inverse function: `snakeCaseKeys`?"
- "Can you optimize this to avoid creating new objects if no keys changed?"

### Conceptual Questions
- "Where does this pattern belong in frontend architecture?" (Axios/fetch response interceptors or API transformation layers).

---

## 🔄 Variations

- **Snake Case Keys:** Reverse transformation (`camelCase` -> `snake_case`).
- **Kebab Case Keys:** Converting keys for CSS/HTML attribute usage.
- **Deep Map:** Applying an arbitrary transformation function to all keys or values in an object.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
function toCamelCase(str) {
  return str
    .toLowerCase()
    .replace(/_([a-z0-9])/g, (_, char) => char.toUpperCase());
}

function camelCaseKeys(val) {
  if (val === null || typeof val !== 'object') return val;
  if (Array.isArray(val)) return val.map(camelCaseKeys);
  const out = {};
  for (const [k, v] of Object.entries(val)) {
    out[toCamelCase(k)] = camelCaseKeys(v);
  }
  return out;
}
```

---

## Official Solution

## Camel Case Keys ( Official solution )

Premium
Zhenghao He
Engineering Manager, Robinhood
Languages
This is a recursive shape-preserving transform: keep the same nested arrays and objects, but rebuild each object's keys in camel case.

## Clarification questions

- Is every key going to be snake-cased? Ignore keys that already use another naming convention.
- Do inherited or non-enumerable keys matter? No. This interview version only needs enumerable own string keys.
- Can the object contain cyclic references? No. Cycles are out of scope here.

## Solution

The simplest frame is to handle three input categories:

1. If the value is an array, recursively convert each item and return a new array.
2. If the value is a non-null object, rebuild it from `Object.entries(...)`, converting each key and recursively converting each value.
3. Otherwise, return the value as-is.

The function never mutates the original input. Every recursive step returns a fresh array or object, so parent calls can assemble a converted copy safely.

For the key conversion itself, any snake-case-to-camel-case helper works. This implementation uses a regex replacement, but splitting on `_` and rebuilding the string would also be fine.

In this implementation, the helper lowercases the whole key before promoting letters after underscores. That is why a key like `Boo_Bar` becomes `booBar` rather than preserving the initial capital.

The output key is computed before the nested value is attached to the rebuilt object. If two original keys normalize to the same camel-cased name, this `Object.fromEntries` style keeps the later entry, which is the normal object assignment behavior rather than a special merge rule.

For `{ foo_bar: true, bar_baz: [{ baz_qux: 1 }] }`, the recursion does this:

| Current value | Case | Returned shape |
| --- | --- | --- |
| root object | rebuild entries | `{ fooBar: ..., barBaz: ... }` |
| `true` | primitive leaf | `true` |
| array under `bar_baz` | map items | `[converted item]` |
| `{ baz_qux: 1 }` | rebuild entries | `{ bazQux: 1 }` |

That trace is the whole algorithm: preserve the container shape, rename object keys, and recurse into values.

```jsx
/**
 * @param {string} str
 * @return {string}
 */
function camelCase(str) {
  return str
    .toLowerCase()
    .replace(/([_])([a-z])/g, (_match, _p1, p2) => p2.toUpperCase());
}

/**
 * @param {unknown} object
 * @return {unknown}
 */
export default function camelCaseKeys(object) {
  // Arrays keep their shape; only the nested values need recursive processing.
  if (Array.isArray(object)) {
    return object.map((item) => camelCaseKeys(item));
  }

  if (typeof object !== 'object' || object === null) {
    return object;
  }

  // Rebuild plain objects so both keys and nested values are transformed.
  return Object.fromEntries(
    Object.entries(object).map(([key, value]) => [
      camelCase(key),
      camelCaseKeys(value),
    ]),
  );
}
```

## Edge cases

- Arrays are traversed, but their numeric indexes are not renamed.
- Because the solution uses `Object.entries`, it only processes enumerable own string keys.
- Existing camel-case keys pass through the helper unchanged except for the code's lowercasing behavior.
- The prompt's scoped formats exclude hyphenated keys, spaces, PascalCase, and cyclic structures.
- Cyclic references are intentionally unsupported in this interview-scoped version.

## Resources

- [`camelcase-keys` library on GitHub](https://github.com/sindresorhus/camelcase-keys)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A response contains both `Foo_Bar` and `foo_bar`:

```javascript
camelCaseKeys({ Foo_Bar: 1, foo_bar: 2 });
```

Both keys normalize to `fooBar`. What does this reveal about using the result to reconstruct the original response?
