---
title: "Schema Validator"
aliases:
  - "schemaValidator"
  - "Schema Validator"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Schema Validator

> [!info] Problem
> Implement a tiny schema validation library with primitive schemas and flat object shapes

## Problem

## Schema Validator

Libraries like [Zod](https://zod.dev/) and [Joi](https://joi.dev/) let application code describe data shapes once and reuse them to validate values.

In this question, implement a small validator builder exposed through `v`. Each schema should expose `safeParse(value)`.

This first question is intentionally limited:

- Support `v.string()`, `v.number()`, `v.boolean()`, and `v.object(shape)`.
- `v.object(shape)` only supports a flat structure, and all fields are required.
- Unknown object keys are allowed and preserved.

## Examples

```javascript
const User = v.object({
  name: v.string(),
  age: v.number(),
  admin: v.boolean(),
});

User.safeParse({
  name: 'Alice',
  age: 30,
  admin: false,
  team: 'Core',
});
// {
//   success: true,
//   data: {
//     name: 'Alice',
//     age: 30,
//     admin: false,
//     team: 'Core',
//   },
// }
```

Validation failures should return all top-level field errors in schema declaration order.

```javascript
const User = v.object({
  name: v.string(),
  age: v.number(),
  admin: v.boolean(),
});

User.safeParse({
  name: 123,
  admin: 'yes',
});
// {
//   success: false,
//   errors: [
//     { path: ['name'], message: 'Expected string' },
//     { path: ['age'], message: 'Required' },
//     { path: ['admin'], message: 'Expected boolean' },
//   ],
// }
```

## API

### v.string()

Creates a schema that accepts string values. Non-string values should return `Expected string`.

### v.number()

Creates a schema that accepts number values. Non-number values should return `Expected number`.

### v.boolean()

Creates a schema that accepts boolean values. Non-boolean values should return `Expected boolean`.

### v.object(shape)

Creates a schema that validates an object against `shape`.

`shape` is an object whose values are schemas created by `v`.

Non-object values such as `null` or arrays should return `Expected object`.

Fields in `shape` are required in this question. Missing or `undefined` fields should return `Required`.

### schema.safeParse(value)

Returns one of the following:

```javascript
{ success: true, data: value }
```

or

```javascript
{
  success: false,
  errors: [
    { path: Array<string | number>, message: string },
  ],
}
```

## Notes

- Unknown object keys are allowed and should be preserved in `data`.
- Object fields are required in this question. Missing or `undefined` fields should return `Required`.
- `safeParse()` must not mutate the input value.
- You do not need arrays, nested objects, optional fields, coercion, transforms, async validation, or custom error messages.

## Resources

- [Zod](https://zod.dev/)
- [Joi](https://joi.dev/)

## Hints

### Hint 1 : How can schemas validate at any path?

### Hint 2 : Which failures stop object traversal?

### Hint 3 : Can the caller change an existing schema later?

## 🤔 Thought Process

- **Immediate Recognition:** Building a tiny runtime schema validation engine mimicking Zod (`z.string()`, `z.object()`, `.safeParse()`).
- **Required Schemas:**
  - `v.string()`: Validates `typeof value === 'string'`.
  - `v.number()`: Validates `typeof value === 'number' && !Number.isNaN(value)`.
  - `v.boolean()`: Validates `typeof value === 'boolean'`.
  - `v.object(shape)`: Validates that `value` is a non-null object, and each field in `shape` passes its corresponding validator.
- **Output Format for `safeParse(value)`:**
  - Success: `{ success: true, data: value }` (preserves all keys, including unknown keys).
  - Failure: `{ success: false, error: '...' }` (returns informative failure message).
- **Architecture Pattern:**
  - Each schema builder function returns an object with a `safeParse(value)` method.
  - Composition: `v.object(shape)` delegates validation of each field to `shape[key].safeParse(value[key])`.

---

## 🧠 Mental Model

Think of a **Type Guard Parser Pipeline (Zod Pattern)**:
```
Data ──► Schema.safeParse()
              ├── Valid?   ──► { success: true, data }
              └── Invalid? ──► { success: false, error }
```
Schemas are composable validator objects wrapping a parse predicate.

---

## 🔑 Key Concepts

- [[Type Checking]]
- [[Method Chaining]]
- [[Closure]]
- Runtime Type Validation vs Compile-time TypeScript
- Result Object Pattern (`{ success, data } | { success, error }`)
- Schema Composition & Combinators

---

## ⚠️ Edge Cases / Traps

- **`NaN` Number Check:** `typeof NaN === 'number'`, but `NaN` is not considered a valid number in schema validation. Always check `!Number.isNaN(value)`.
- **`null` Check in Object Validation:** `typeof null === 'object'`. Validating an object against `null` must fail.
- **Preserving Unknown Keys:** The problem specifies that unknown keys are allowed and preserved on the parsed object. Do NOT strip unlisted keys.
- **Missing Required Fields:** If a key declared in `shape` is missing from the input object, validation must fail.

---

## ⭐ Interview Takeaway

- Factory pattern returning `{ safeParse: (val) => ... }`:
  ```javascript
  const createSchema = (validateFn) => ({
    safeParse: (val) => {
      const error = validateFn(val);
      return error ? { success: false, error } : { success: true, data: val };
    }
  });
  ```
- Object validator delegates to field validators:
  Iterate `Object.keys(shape)` and check `shape[k].safeParse(val?.[k])`.
- Explain the role of runtime validation: TypeScript types are erased at runtime; schema validators guard API boundaries and user input.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why is runtime schema validation needed in TypeScript applications?" (TypeScript only verifies types at compile time; runtime validation guarantees external API responses, form inputs, and JSON payloads conform to expected shapes).
- "Why must `Number.isNaN()` be checked for `v.number()`?" (`typeof NaN === 'number'`, which would falsely pass a naive `typeof` check).

### Follow-up Questions
- "How would you add modifier methods like `.optional()` or `.nullable()`?" (Return a new schema wrapper that permits `undefined` or `null` before calling the underlying validator).
- "How would you collect all validation errors across multiple fields instead of failing on the first one?" (Accumulate field error paths and messages into an array).

### Conceptual Questions
- "How does Zod's `parse` differ from `safeParse`?" (`parse` throws a `ZodError` on failure; `safeParse` returns a discriminated union result object).

---

## 🔄 Variations

- **Schema Validator II & III:** Adding `.optional()`, `.array()`, and nested schemas.
- **Type Utilities:** Custom type-checking helpers (`isObject`, `isNumber`).
- **Form Validation Engine:** Field-level and form-level error management.

---

## 📝 Revision Notes

- Complete implementation:
```javascript
const v = {
  string() {
    return {
      safeParse(val) {
        if (typeof val === 'string') return { success: true, data: val };
        return { success: false, error: 'Expected string' };
      },
    };
  },
  number() {
    return {
      safeParse(val) {
        if (typeof val === 'number' && !Number.isNaN(val)) return { success: true, data: val };
        return { success: false, error: 'Expected number' };
      },
    };
  },
  boolean() {
    return {
      safeParse(val) {
        if (typeof val === 'boolean') return { success: true, data: val };
        return { success: false, error: 'Expected boolean' };
      },
    };
  },
  object(shape) {
    return {
      safeParse(val) {
        if (val === null || typeof val !== 'object' || Array.isArray(val)) {
          return { success: false, error: 'Expected object' };
        }
        for (const [key, validator] of Object.entries(shape)) {
          if (!(key in val)) {
            return { success: false, error: `Missing required field: ${key}` };
          }
          const res = validator.safeParse(val[key]);
          if (!res.success) {
            return res;
          }
        }
        return { success: true, data: val };
      },
    };
  },
};

export default v;
```

---

## Official Solution

## Schema Validator ( Official solution )

Premium
Languages

## Solution

Design a reusable schema abstraction, then compose those schemas into `object(shape)`. A useful split is: schemas validate their own node, while parent schemas decide which child path to pass down.

Every schema has two tasks:

1. `safeParse(value)` for the public API.
2. An internal validator that can add path-aware errors to a shared array.

That split keeps the public result shape in one place. `safeParse()` creates a fresh `errors` array, calls the internal validator at the root path `[]`, and returns either `{ success: true, data: value }` or `{ success: false, errors }`.

The validator has three conceptual pieces:

1. `createSchema(validate)` wraps an internal validator with `safeParse()`.
2. Primitive schemas only perform their own type check and append an error at the path they receive.
3. `object(shape)` checks that the current value is a plain object, then walks the declared shape in order and delegates present fields to child schemas with `path + key`.

A schema validates only its own node. Primitive schemas do not know whether they are validating the root value or a field. Object schemas are responsible for deciding whether a declared key is missing, whether it is explicitly `undefined`, and what child path should be used.

For example, validating this schema:

```javascript
const User = v.object({
  name: v.string(),
  age: v.number(),
  admin: v.boolean(),
});
```

against `{ name: 123, admin: 'yes' }` walks the keys as `name`, `age`, then `admin`. `name` delegates to the string schema at `['name']`, `age` is missing so the object schema appends `Required`, and `admin` delegates to the boolean schema at `['admin']`. Because every validator appends to the same `errors` array, all field errors are returned in schema declaration order.

| Field | Value seen | Validator action | Error appended |
| --- | --- | --- | --- |
| `name` | `123` | Delegate to `v.string()` | `Expected string` at `['name']` |
| `age` | Missing | Object schema handles requiredness | `Required` at `['age']` |
| `admin` | `"yes"` | Delegate to `v.boolean()` | `Expected boolean` at `['admin']` |

```jsx
/**
 * @typedef {string | number} PathSegment
 *
 * @typedef {{
 *   path: Array<PathSegment>,
 *   message: string,
 * }} ValidationError
 *
 * @typedef {{
 *   success: true,
 *   data: unknown,
 * } | {
 *   success: false,
 *   errors: Array<ValidationError>,
 * }} ParseResult
 *
 * @typedef {{
 *   safeParse(value: unknown): ParseResult,
 *   _validate(
 *     value: unknown,
 *     path: Array<PathSegment>,
 *     errors: Array<ValidationError>,
 *   ): void,
 * }} Schema
 */
function createSchema(validate) {
  return Object.freeze({
    safeParse(value) {
      const errors = [];
      validate(value, [], errors);

      if (errors.length > 0) {
        return {
          success: false,
          errors,
        };
      }

      return {
        success: true,
        data: value,
      };
    },

    _validate: validate,
  });
}

function isPlainObject(value) {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

/**
 * @type {{
 *   string(): Schema,
 *   number(): Schema,
 *   boolean(): Schema,
 *   object(shape: Record<string, Schema>): Schema,
 * }}
 */
const v = Object.freeze({
  string() {
    return createSchema((value, path, errors) => {
      if (typeof value !== 'string') {
        errors.push({
          path,
          message: 'Expected string',
        });
      }
    });
  },

  number() {
    return createSchema((value, path, errors) => {
      if (typeof value !== 'number') {
        errors.push({
          path,
          message: 'Expected number',
        });
      }
    });
  },

  boolean() {
    return createSchema((value, path, errors) => {
      if (typeof value !== 'boolean') {
        errors.push({
          path,
          message: 'Expected boolean',
        });
      }
    });
  },

  object(shape) {
    const shapeCopy = { ...shape };

    return createSchema((value, path, errors) => {
      if (!isPlainObject(value)) {
        errors.push({
          path,
          message: 'Expected object',
        });
        return;
      }

      for (const key of Object.keys(shapeCopy)) {
        if (!Object.hasOwn(value, key) || value[key] === undefined) {
          errors.push({
            path: [...path, key],
            message: 'Required',
          });
          continue;
        }

        shapeCopy[key]._validate(value[key], [...path, key], errors);
      }
    });
  },
});

export default v;
```

## Common pitfalls

- **Stopping after the first field error:** This validator should collect all top-level field errors. Return early only when the current value cannot be treated as an object at all; otherwise continue walking the shape.
- **Treating arrays or `null` as objects:** The object schema should accept plain object values only. A `typeof value === 'object'` check is not enough because both arrays and `null` need to return `Expected object`.
- **Validating missing fields with child schemas:** Missing and `undefined` fields should return `Required`, not the child schema's type message. The parent object schema needs to detect those cases before delegating.
- **Rejecting unknown keys:** Unknown keys are allowed. Since the solution does not transform data, a successful parse can return the original input value with those keys preserved.
- **Letting external shape mutation change behavior:** Copy the `shape` object when `v.object(shape)` is created. That keeps later changes to the caller's shape object from changing the schema that was already built.

## Notes

- Returning all errors is easiest when every schema appends to the same `errors` array.
- Since this question preserves unknown keys and does not transform data, a successful parse can return the original input value.
- Cloning the `shape` object when building `v.object(shape)` prevents later external mutation from changing the schema behavior.
- `Object.keys(shapeCopy)` gives deterministic field traversal in schema declaration order for the fields used in this question.
- `safeParse()` returns data-or-errors instead of throwing, which makes validation failures ordinary control flow for callers.

## Techniques

- Schema builder pattern.
- Shared error accumulator.
- Path-aware validation.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A validator iterates over the input object’s keys instead of the schema’s keys. Which test exposes both a missed required field and the wrong error order?
