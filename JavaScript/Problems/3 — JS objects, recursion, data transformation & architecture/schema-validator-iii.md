---
title: "Schema Validator III"
aliases:
  - "schemaValidatorIII"
  - "Schema Validator III"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Schema Validator III

> [!info] Problem
> Implement a tiny schema validation library with nested objects, arrays, and optional fields

## Problem

## Schema Validator III

This is a follow-up to [Schema Validator II](/questions/javascript/schema-validator-ii).

Keep the same `v` API and `safeParse(value)` result shape, but now add recursive schema composition:

- Nested `v.object(shape)`
- `v.array(itemSchema)`
- `.optional()`

Validation errors should include full paths with string and number segments.

## Examples

```javascript
const UserList = v.object({
  users: v.array(
    v.object({
      name: v.string(),
      age: v.number().min(18),
    }),
  ),
});

UserList.safeParse({
  users: [
    { name: 'Alice', age: 30 },
    { name: 123, age: 16 },
  ],
});
// {
//   success: false,
//   errors: [
//     { path: ['users', 1, 'name'], message: 'Expected string' },
//     { path: ['users', 1, 'age'], message: 'Expected number >= 18' },
//   ],
// }
```

Optional fields may be omitted or set to `undefined`.

```javascript
const User = v.object({
  name: v.string(),
  nickname: v.string().optional(),
  tags: v.array(v.string()).optional(),
});

User.safeParse({ name: 'Alice' });
// { success: true, data: { name: 'Alice' } }
```

Schema builder methods must still be immutable.

```javascript
const Email = v.string();
const User = v.object({
  primaryEmail: Email,
  backupEmail: Email.optional(),
});

Email.safeParse(undefined);
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected string' }],
// }

User.safeParse({ primaryEmail: 'alice@example.com' });
// {
//   success: true,
//   data: { primaryEmail: 'alice@example.com' },
// }
```

## API

### v.string()

Still accepts string values. Non-string values should return `Expected string`.

### v.number()

Still accepts number values. Non-number values should return `Expected number`.

### v.boolean()

Still accepts boolean values. Non-boolean values should return `Expected boolean`.

### v.object(shape)

Now supports recursive nesting. Non-object values such as `null` or arrays should return `Expected object`, and missing or `undefined` required object fields should return `Required` at the full nested path.

### stringSchema.min(length)

Still requires the string length to be at least `length`. If the value is too short, return `Expected at least ${length} characters`.

### stringSchema.max(length)

Still requires the string length to be at most `length`. If the value is too long, return `Expected at most ${length} characters`.

### numberSchema.min(value)

Still requires the number to be greater than or equal to `value`. If the value is too small, return `Expected number >= ${value}`.

### numberSchema.max(value)

Still requires the number to be less than or equal to `value`. If the value is too large, return `Expected number <= ${value}`.

### v.array(itemSchema)

Creates a schema that validates arrays whose items must match `itemSchema`. Non-array values should return `Expected array`.

### schema.optional()

Creates a new schema that also accepts `undefined`.

When `optional()` is used on an object field, that field may be missing or `undefined`. If an optional value is present but invalid, it should return the same error message as the underlying schema.

## Returns

`schema.safeParse(value)` still returns:

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

- Errors should be collected in deterministic order: object keys in schema declaration order, array items in index order, and chained rule checks in call order.
- Unknown object keys are allowed and should be preserved in `data`.
- Schema instances are immutable. Calling methods such as `.min()`, `.max()`, or `.optional()` must return a new schema instance without changing the original schema. `safeParse()` must not mutate the input value.
- You do not need coercion, transforms, async validation, unions, custom refinements, unknown-key stripping, or custom error messages.

## Resources

- [Zod](https://zod.dev/)
- [Joi](https://joi.dev/)

## Hints

### Hint 1 : How does a full error path grow?

### Hint 2 : Who decides whether a missing field is allowed?

### Hint 3 : Does optional mean unvalidated?

## 🤔 Thought Process

- **Immediate Recognition:** Full recursive schema engine supporting nested objects, arrays (`v.array`), optional modifiers (`.optional()`), and path-aware error reporting (e.g. `users.0.email`).
- **New Capabilities:**
  - `v.array(itemSchema)`: Validates that value is an array, then validates every element with `itemSchema`.
  - `.optional()`: Appended to any schema; allows `undefined` without error.
  - Nested `v.object(shape)`: Recursively parses object properties.
  - Path Tracking: When an error occurs deeply nested inside an object or array, the failure object must identify the exact path:
    `{ success: false, error: '...', path: ['users', 0, 'email'] }` or formatted string.
- **Recursive Parsing Pipeline:**
  - Pass an accumulated `path` array (or build it on return) through the parser recursion.
  - Optional flag: If `value === undefined` and `schema.isOptional`, immediately return `{ success: true, data: undefined }`.
  - Arrays: Loop by index; if `itemSchema.safeParse(item, [...path, index])` fails, bubble error with index segment.
  - Objects: Loop keys in `shape`; if field validation fails, bubble error with key segment.

---

## 🧠 Mental Model

Think of a **Recursive Document Validator with Breadcrumbs**:
```
Document: { users: [ { name: "Alice" }, { name: 123 } ] }
                              │              │
                    Path: ['users', 0]    Path: ['users', 1]
                            PASS          FAIL at 'name': Expected string
                                          Final Path: "users.1.name"
```

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Recursion]]
- [[Method Chaining]]
- [[Object Path Traversal]]
- Recursive AST Traversal
- Path Breadcrumbs in Error Reporting
- Optionality / Nullability Modifier Wrappers

---

## ⚠️ Edge Cases / Traps

- **`undefined` vs Missing Keys:** If a field is `.optional()`, `val === undefined` or omitting the key is valid. However, passing `null` is NOT valid unless explicitly marked nullable.
- **Empty Arrays:** `v.array(schema).safeParse([])` must succeed.
- **Preserving Unknown Keys in Objects:** Validating a nested object must retain unvalidated sibling keys on the returned `data` object.
- **Array Index Segments in Paths:** Path segments must be numbers for array indices (`['users', 0, 'name']`) and strings for object properties.

---

## ⭐ Interview Takeaway

- Wrap schemas to handle `.optional()`:
  ```javascript
  function makeOptional(schema) {
    schema.isOptional = true;
    schema.optional = () => schema;
    return schema;
  }
  ```
- Pass `path = []` recursively to track breadcrumbs when descending into arrays and nested objects.
- Short-circuit on `val === undefined` if `isOptional` is true:
  `if (val === undefined && this.isOptional) return { success: true, data: undefined };`

---

## 🎯 Common Interview Questions

### Direct Questions
- "How are path breadcrumbs constructed during nested validation errors?" (Each container layer prepends its key or numeric index to the error path reported by its child).
- "What is the difference between `.optional()` and `.nullable()` in schema validators?" (`optional` accepts `undefined`; `nullable` accepts `null`).

### Follow-up Questions
- "How would you implement union types (`v.union([v.string(), v.number()])`)?"
- "How would you implement `.transform()` (e.g. `z.string().transform(s => s.trim())`)?"

### Conceptual Questions
- "Why does Zod format error issues with an array of path segments (`['users', 0, 'email']`) rather than a joined string?" (Programmatic consumers can access the exact object path for field-level form error highlighting).

---

## 🔄 Variations

- **Schema Validator I & II:** Precursor levels.
- **JSON Schema / Ajv:** Standard declarative JSON schema validator.
- **Yup / Joi:** Alternative schema validation libraries.

---

## 📝 Revision Notes

- Recursive array and object parser:
```javascript
function arraySchema(itemSchema) {
  return {
    safeParse(val, path = []) {
      if (!Array.isArray(val)) {
        return { success: false, error: 'Expected array', path };
      }
      const data = [];
      for (let i = 0; i < val.length; i++) {
        const res = itemSchema.safeParse(val[i], [...path, i]);
        if (!res.success) return res;
        data.push(res.data);
      }
      return { success: true, data };
    },
  };
}
```

---

## Official Solution

## Schema Validator III ( Official solution )

Premium
Languages

## Solution

This question keeps the same immutable builder design from part II, but the validators now need to recurse into nested arrays and objects while carrying a full path. Each schema validates the current node, and composite schemas decide which child path comes next.

The simplest structure is to give every schema three pieces of behavior:

1. `safeParse(value)` for the public entry point.
2. An internal `_validate(value, path, errors)` method for recursive validation.
3. An `_isOptional` flag so parent object schemas know whether a missing field should produce `Required`.

The shared `createBaseSchema()` handles the common optional behavior. If a schema is optional and the value is `undefined`, both top-level `safeParse()` and nested `_validate()` short-circuit successfully before type-specific validation runs. Otherwise, validation continues with the same shared error accumulator used in the earlier parts.

The main schema jobs are:

1. Primitive schemas perform their type check, then run ordered min or max rules when applicable.
2. `object(shape)` checks for a plain object, walks declared fields in order, handles missing or `undefined` fields, and delegates present fields with `path + key`.
3. `array(itemSchema)` checks for an array, walks items by numeric index, and delegates each item with `path + index`.
4. `.optional()` returns a new schema instance with the same configuration but `_isOptional: true`.

For the prompt's nested example, the path starts as `[]`. The `users` object field delegates to the array schema with `['users']`. The second item delegates to the object schema with `['users', 1]`, and that object delegates `name` and `age` as `['users', 1, 'name']` and `['users', 1, 'age']`. Because every nested validator appends to the same `errors` array, the final result preserves object declaration order, array index order, and rule call order.

Because every chainable method returns a new frozen schema object with copied configuration, the builder API stays immutable even as rules and optional flags are added. That is what lets `Email` stay required while `Email.optional()` is reused in another branch.

```jsx
function createBaseSchema(validate, isOptional) {
  return {
    _isOptional: isOptional,

    safeParse(value) {
      // Optional schemas short-circuit before any type-specific validation runs.
      if (value === undefined && isOptional) {
        return {
          success: true,
          data: value,
        };
      }

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

    _validate(value, path, errors) {
      // Nested validators use the same optional rule when parents recurse into them.
      if (value === undefined && isOptional) {
        return;
      }

      validate(value, path, errors);
    },
  };
}

function isPlainObject(value) {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function createStringSchema(rules = [], isOptional = false) {
  const validate = (value, path, errors) => {
    if (typeof value !== 'string') {
      errors.push({
        path,
        message: 'Expected string',
      });
      return;
    }

    for (const rule of rules) {
      if (rule.type === 'min' && value.length < rule.value) {
        errors.push({
          path,
          message: `Expected at least ${rule.value} characters`,
        });
      }

      if (rule.type === 'max' && value.length > rule.value) {
        errors.push({
          path,
          message: `Expected at most ${rule.value} characters`,
        });
      }
    }
  };

  return Object.freeze({
    ...createBaseSchema(validate, isOptional),
    min(length) {
      return createStringSchema(
        [...rules, { type: 'min', value: length }],
        isOptional,
      );
    },
    max(length) {
      return createStringSchema(
        [...rules, { type: 'max', value: length }],
        isOptional,
      );
    },
    optional() {
      return createStringSchema([...rules], true);
    },
  });
}

function createNumberSchema(rules = [], isOptional = false) {
  const validate = (value, path, errors) => {
    if (typeof value !== 'number') {
      errors.push({
        path,
        message: 'Expected number',
      });
      return;
    }

    for (const rule of rules) {
      if (rule.type === 'min' && value < rule.value) {
        errors.push({
          path,
          message: `Expected number >= ${rule.value}`,
        });
      }

      if (rule.type === 'max' && value > rule.value) {
        errors.push({
          path,
          message: `Expected number <= ${rule.value}`,
        });
      }
    }
  };

  return Object.freeze({
    ...createBaseSchema(validate, isOptional),
    min(minimum) {
      return createNumberSchema(
        [...rules, { type: 'min', value: minimum }],
        isOptional,
      );
    },
    max(maximum) {
      return createNumberSchema(
        [...rules, { type: 'max', value: maximum }],
        isOptional,
      );
    },
    optional() {
      return createNumberSchema([...rules], true);
    },
  });
}

function createBooleanSchema(isOptional = false) {
  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (typeof value !== 'boolean') {
        errors.push({
          path,
          message: 'Expected boolean',
        });
      }
    }, isOptional),
    optional() {
      return createBooleanSchema(true);
    },
  });
}

function createObjectSchema(shape, isOptional = false) {
  const shapeCopy = { ...shape };

  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (!isPlainObject(value)) {
        errors.push({
          path,
          message: 'Expected object',
        });
        return;
      }

      for (const key of Object.keys(shapeCopy)) {
        const fieldSchema = shapeCopy[key];

        // Missing keys are allowed only when the child schema was marked optional.
        if (!Object.hasOwn(value, key) || value[key] === undefined) {
          if (!fieldSchema._isOptional) {
            errors.push({
              path: [...path, key],
              message: 'Required',
            });
          }

          continue;
        }

        fieldSchema._validate(value[key], [...path, key], errors);
      }
    }, isOptional),
    optional() {
      return createObjectSchema(shapeCopy, true);
    },
  });
}

function createArraySchema(itemSchema, isOptional = false) {
  return Object.freeze({
    ...createBaseSchema((value, path, errors) => {
      if (!Array.isArray(value)) {
        errors.push({
          path,
          message: 'Expected array',
        });
        return;
      }

      for (let index = 0; index < value.length; index += 1) {
        itemSchema._validate(value[index], [...path, index], errors);
      }
    }, isOptional),
    optional() {
      return createArraySchema(itemSchema, true);
    },
  });
}

const v = Object.freeze({
  string() {
    return createStringSchema();
  },

  number() {
    return createNumberSchema();
  },

  boolean() {
    return createBooleanSchema();
  },

  object(shape) {
    return createObjectSchema(shape);
  },

  array(itemSchema) {
    return createArraySchema(itemSchema);
  },
});

export default v;
```

## Common pitfalls

- **Losing the full nested path:** Always pass a new path with the next segment appended before delegating. Object fields append string keys, and array items append numeric indices.
- **Treating optional fields as globally optional mutations:** `.optional()` should return a new schema. It should not mutate the original schema, because the same base schema may be reused as required in one branch and optional in another.
- **Reporting `Required` for optional object fields:** The parent object schema decides whether a missing or `undefined` field is required. It should inspect the child schema's `_isOptional` flag before appending `Required`.
- **Skipping validation for present optional values:** Optional only means `undefined` is allowed. If an optional field is present with an invalid value, the underlying schema should still produce the same error it would produce when required.
- **Returning only the first nested error:** Nested validation should continue after failures whenever the current node can still be traversed. That is how arrays can report multiple invalid items and objects can report errors from multiple fields.
- **Accepting arrays as objects:** Object schemas still need a plain-object check. Arrays are handled by `v.array(itemSchema)` and should return `Expected object` when passed to an object schema.

## Notes

- Returning all errors becomes straightforward once each nested validator appends to the same shared array.
- Since the validator does not coerce or transform anything, successful parses can still return the original input value.
- Optional fields are handled in two places: top-level `safeParse()` and parent schemas when a child value is missing.
- Deterministic error order comes from using object declaration order, array index order, and chained rule order.
- Unknown object keys are still allowed and preserved in `data`.
- This question does not require coercion, transforms, async validation, unions, custom refinements, unknown-key stripping, or custom error messages.

## Techniques

- Recursive validation.
- Shared error accumulator.
- Immutable builder pattern.
- Path propagation.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A recursive validator reuses one mutable path array: push a segment, recurse, then pop it. Each failure stores that array directly as `error.path`. Why can all returned paths end up empty even though traversal visited the right locations? Give two valid repair strategies.

Your notes (optional)
