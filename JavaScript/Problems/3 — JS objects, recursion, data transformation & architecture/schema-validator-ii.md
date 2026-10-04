---
title: "Schema Validator II"
aliases:
  - "schemaValidatorII"
  - "Schema Validator II"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Schema Validator II

> [!info] Problem
> Implement a tiny schema validation library with chainable min and max rules

## Problem

## Schema Validator II

This is a follow-up to [Schema Validator](/questions/javascript/schema-validator).

Keep the same `v` API and `safeParse(value)` result shape, but now add chainable primitive rules:

- `v.string().min(n).max(n)`
- `v.number().min(n).max(n)`

Object validation stays the same as in the previous question:

- `v.object(shape)` only needs to support flat required fields.
- Unknown object keys are still allowed and preserved.

## Examples

```javascript
const Name = v.string().min(2).max(5);

Name.safeParse('Ada');
// { success: true, data: 'Ada' }

Name.safeParse('');
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected at least 2 characters' }],
// }
```

Schema builder methods must be immutable.

```javascript
const BaseName = v.string();
const ShortName = BaseName.min(2);

BaseName.safeParse('');
// { success: true, data: '' }

ShortName.safeParse('');
// {
//   success: false,
//   errors: [{ path: [], message: 'Expected at least 2 characters' }],
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

Still validates flat required object shapes. Non-object values such as `null` or arrays should return `Expected object`, and missing or `undefined` required fields should return `Required`.

### stringSchema.min(length)

Requires the string length to be at least `length`. If the value is too short, return `Expected at least ${length} characters`.

### stringSchema.max(length)

Requires the string length to be at most `length`. If the value is too long, return `Expected at most ${length} characters`.

### numberSchema.min(value)

Requires the number to be greater than or equal to `value`. If the value is too small, return `Expected number >= ${value}`.

### numberSchema.max(value)

Requires the number to be less than or equal to `value`. If the value is too large, return `Expected number <= ${value}`.

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

- Multiple rules on the same schema should be checked in the order the methods were called.
- Type checks happen before rule checks, so invalid primitive types should still return the original type messages such as `Expected string` and `Expected number`.
- Unknown object keys are allowed and should be preserved in `data`.
- Schema instances are immutable. Calling methods such as `.min()` or `.max()` must return a new schema instance without changing the original schema. `safeParse()` must not mutate the input value.
- You do not need arrays, nested objects, optional fields, coercion, transforms, async validation, or custom error messages.

## Resources

- [Zod](https://zod.dev/)
- [Joi](https://joi.dev/)

## Hints

### Hint 1 : What does each builder call inherit?

### Hint 2 : When are range rules meaningful?

## 🤔 Thought Process

- **Immediate Recognition:** Extending the Zod-like runtime schema validator with chainable rule constraints (`.min(n)` and `.max(n)`).
- **Rule Scope:**
  - `v.string().min(n).max(n)`: Constraints apply to string character length (`val.length`).
  - `v.number().min(n).max(n)`: Constraints apply to numeric value magnitude (`val`).
  - Objects: Keep the flat object schema validation from Part I.
- **Fluent Builder Pattern:**
  - Each schema builder should return a clone or mutate a schema descriptor carrying an array of validation rules:
    `rules: [(val) => errorString | null]`.
  - Calling `.min(n)` or `.max(n)` appends a new rule predicate to the rules list and returns the schema for chaining.
  - Calling `.safeParse(val)` executes the base type check first; if valid, executes each chained rule in order.
  - Returns `{ success: true, data: val }` or the first failing `{ success: false, error: '...' }`.

---

## 🧠 Mental Model

Think of a **Chain of Constraint Gates**:
```
Value: "hello"
  │
  ├── Gate 1: typeof === 'string'? ──────────► PASS
  ├── Gate 2: min(2): length >= 2? ──────────► PASS
  └── Gate 3: max(4): length <= 4? ──────────► FAIL: "String length exceeds max"
```

---

## 🔑 Key Concepts

- [[Method Chaining]]
- [[Type Checking]]
- [[Range Checking]]
- Fluent Builder Pattern
- Composable validation rule pipelines
- Result object pattern (`{ success, data } | { success, error }`)

---

## ⚠️ Edge Cases / Traps

- **Base Type Failure Preempts Rules:** If `val` is not a string, calling `v.string().min(5).safeParse(123)` must fail with `'Expected string'`, not an error related to `.length`.
- **String vs Number Min/Max Semantics:** For strings, `.min(n)` tests `val.length >= n`. For numbers, `.min(n)` tests `val >= n`.
- **Independent Rule Instances:** `const s = v.string(); const sMin = s.min(2);` ensure chaining either clones the descriptor or caller expectation of immutability is handled.
- **`NaN` Numbers:** Must still fail base number validation before testing range constraints.

---

## ⭐ Interview Takeaway

- Model schemas as an object containing a list of rule checks:
  ```javascript
  function createChainableSchema(typeCheck) {
    const rules = [typeCheck];
    const schema = {
      safeParse(val) {
        for (const rule of rules) {
          const err = rule(val);
          if (err) return { success: false, error: err };
        }
        return { success: true, data: val };
      },
      min(n) { /* push min rule */ return schema; },
      max(n) { /* push max rule */ return schema; },
    };
    return schema;
  }
  ```
- Method chaining pattern: each modifier method registers a check and returns `this`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How does method chaining work when building validator rules?" (Each method appends a validation predicate to an internal rules list and returns the schema reference).
- "Why does `safeParse` return a result object instead of throwing an error?" (Avoids try/catch performance overhead and provides type-narrowed discriminated unions in TypeScript).

### Follow-up Questions
- "How would you handle custom error messages passed to `.min(n, 'Custom message')`?"
- "How would you extend this to support nested schemas and arrays?" (Answered in Schema Validator III).

### Conceptual Questions
- "How does Zod achieve static type inference (`z.infer<typeof schema>`) in TypeScript?" (Using phantom type variables and conditional type extraction from schema definitions).

---

## 🔄 Variations

- **Schema Validator I:** Basic unchained primitives.
- **Schema Validator III:** Nested objects, arrays, and optional fields.
- **Form Validation Engine:** Accumulating all field errors simultaneously.

---

## 📝 Revision Notes

- Chainable primitive factory:
```javascript
function stringSchema() {
  const rules = [];
  const schema = {
    min(n) {
      rules.push(v => v.length < n ? `Length must be at least ${n}` : null);
      return schema;
    },
    max(n) {
      rules.push(v => v.length > n ? `Length must be at most ${n}` : null);
      return schema;
    },
    safeParse(val) {
      if (typeof val !== 'string') return { success: false, error: 'Expected string' };
      for (const rule of rules) {
        const err = rule(val);
        if (err) return { success: false, error: err };
      }
      return { success: true, data: val };
    },
  };
  return schema;
}
```

---

## Official Solution

## Schema Validator II ( Official solution )

Premium
Languages

## Solution

The main design change in this follow-up is immutable builder methods. Primitive schemas now need to carry rule configuration without mutating earlier schema instances, while keeping the same `safeParse()` and `object(shape)` flow from part I.

Read a schema as a frozen snapshot of its rules. Each call to `.min()` or `.max()` creates a new snapshot with one extra rule. The previous schema keeps the rule list it closed over.

Validation splits into four checks:

1. Keep the common `createSchema()` wrapper so public parsing and shared error collection work like part I.
2. Store string and number rules in closure arrays.
3. Rebuild a new string or number schema whenever a chainable method is called.
4. Validate primitive type first, then run the stored rules in the order they were chained.

That gives two useful properties:

- Previously created schemas keep their old behavior.
- Validation still runs rules in the order they were chained.

For example:

```javascript
const BaseName = v.string();
const ShortName = BaseName.min(2);
```

`BaseName` closes over an empty rule list, so `BaseName.safeParse('')` succeeds. `ShortName` closes over a new list containing the `min(2)` rule, so `ShortName.safeParse('')` returns `Expected at least 2 characters`. The builder method returns a new schema instead of editing the old one.

At validation time, type checks remain the gate. A non-string value should return only `Expected string`; a non-number value should return only `Expected number`. Once the base type is correct, the schema walks its stored rules and appends any min or max failures at the current path. Object validation can then reuse the same delegation model from part I, including required fields, unknown-key preservation, and schema declaration order.

```jsx
/**
 * @typedef {string | number} PathSegment
 *
 * @typedef {{
 *   path: PathSegment[],
 *   message: string,
 * }} ValidationError
 *
 * @typedef {{
 *   success: true,
 *   data: unknown,
 * } | {
 *   success: false,
 *   errors: ValidationError[],
 * }} ParseResult
 *
 * @typedef {{
 *   safeParse(value: unknown): ParseResult,
 *   _validate(value: unknown, path: PathSegment[], errors: ValidationError[]): void,
 * }} Schema
 *
 * @typedef {Schema & {
 *   min(length: number): StringSchema,
 *   max(length: number): StringSchema,
 * }} StringSchema
 *
 * @typedef {Schema & {
 *   min(value: number): NumberSchema,
 *   max(value: number): NumberSchema,
 * }} NumberSchema
 *
 * @typedef {{
 *   string(): StringSchema,
 *   number(): NumberSchema,
 *   boolean(): Schema,
 *   object(shape: Record<string, Schema>): Schema,
 * }} SchemaFactory
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

function createStringSchema(rules = []) {
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
    ...createSchema(validate),
    min(length) {
      // Each chained call returns a fresh schema so previously built validators stay immutable.
      return createStringSchema([...rules, { type: 'min', value: length }]);
    },
    max(length) {
      return createStringSchema([...rules, { type: 'max', value: length }]);
    },
  });
}

function createNumberSchema(rules = []) {
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
    ...createSchema(validate),
    min(minimum) {
      // Rebuild the schema with an extra rule instead of mutating the existing one.
      return createNumberSchema([...rules, { type: 'min', value: minimum }]);
    },
    max(maximum) {
      return createNumberSchema([...rules, { type: 'max', value: maximum }]);
    },
  });
}

/** @type {SchemaFactory} */
const v = Object.freeze({
  string() {
    return createStringSchema();
  },

  number() {
    return createNumberSchema();
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

- **Mutating the existing schema:** Builder methods such as `.min()` and `.max()` must return a new schema. Mutating the current schema would make previously created validators change behavior after they have already been shared or stored.
- **Sharing a mutable rules array:** Copy the rule array when appending a rule. If two schema instances close over the same mutable array, a later builder call can accidentally affect both schemas.
- **Running rules before type checks:** Rules only apply after the primitive type is valid. A value like `123` passed to `v.string().min(2)` should return `Expected string`, not a length-related error.
- **Losing chained rule order:** Multiple rules should be checked in the order the methods were called. Appending copied rule objects to an array preserves that order naturally.
- **Changing object behavior from part I:** This follow-up adds primitive rules, not a new object rule. `object(shape)` should still require declared fields, treat missing or `undefined` fields as `Required`, preserve unknown keys, and collect errors in declaration order.

## Notes

- Freezing the returned schema objects reinforces that builder calls create new instances instead of mutating old ones.
- Reusing the part I `safeParse()` and `object(shape)` flow keeps this follow-up focused on rule composition instead of API redesign.
- String rules use string length; number rules compare numeric value.
- Returning all errors still works through the same shared `errors` array used in part I.

## Techniques

- Immutable builder pattern.
- Closures for rule configuration.
- Ordered rule evaluation.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
An implementation stores just one `min` and one `max`, replacing older rules of the same kind. Which test distinguishes it from the required ordered rule history?
