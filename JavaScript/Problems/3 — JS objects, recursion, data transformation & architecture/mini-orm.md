---
title: "Mini Object-relational Mapper"
aliases:
  - "miniOrm"
  - "Mini Object-relational Mapper"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Mini Object-relational Mapper

> [!info] Problem
> Implement a simplified Prisma-like in-memory object-relational mapper (ORM) with model delegates and basic CRUD operations

## Problem

## Mini Object-relational Mapper

Many JavaScript apps use object-relational mappers (ORMs) such as Prisma so application code can read and write data through model delegates like `db.user.findMany()` and `db.article.create()`. In this question, you will implement a small in-memory version of that idea.

Implement a `MiniORM` class that exposes one delegate per model name.

The model names are arbitrary. If the input contains `user`, `post`, or `article`, the instance should expose `db.user`, `db.post`, or `db.article` respectively. Do not hardcode any model names.

## Examples

```javascript
const db = new MiniORM({
  article: [
    { id: 1, title: 'Intro to ORMs', published: true },
    { id: 2, title: 'Query Builders', published: false },
  ],
});

db.article.findMany();
// [
//   { id: 1, title: 'Intro to ORMs', published: true },
//   { id: 2, title: 'Query Builders', published: false },
// ]

db.article.findMany({ where: { published: true } });
// [{ id: 1, title: 'Intro to ORMs', published: true }]

db.article.create({
  data: { id: 3, title: 'Includes and Selects', published: true },
});
// { id: 3, title: 'Includes and Selects', published: true }

db.article.update({
  where: { id: 2 },
  data: { published: true },
});
// { id: 2, title: 'Query Builders', published: true }

db.article.delete({
  where: { id: 1 },
});
// { id: 1, title: 'Intro to ORMs', published: true }
```

## MiniORM API

### new MiniORM(data)

Creates a `MiniORM` instance from an object whose keys are model names and whose values are arrays of records.

Each `MiniORM` instance should expose one delegate per model name. For example, if the input contains an `article` array, the instance should expose `db.article`.

Records may contain arbitrary top-level properties.

### model.findMany([args])

Returns the matching records for the model in insertion order.

| Argument | Type | Description |
| --- | --- | --- |
| `args.where` | `Object` | (Optional) Exact-match conditions. A record matches only if all provided fields are strictly equal (`===`) to the corresponding stored values. |

Return a new array each time. Returned records only need shallow copies.

### model.create({ data })

Appends `data` to the model and returns the created record.

Store a shallow copy of `data`, not the original object reference.

| Argument | Type | Description |
| --- | --- | --- |
| `data` | `Object` | The record to append. |

### model.update({ where, data })

Finds the single matching record, shallow-merges `data` into it, stores the result, and returns the updated record.

| Argument | Type | Description |
| --- | --- | --- |
| `where` | `Object` | Exact-match conditions that are guaranteed to match exactly one record. |
| `data` | `Object` | Partial fields to shallow-merge into the stored record. |

### model.delete({ where })

Finds the single matching record, removes it from the model, and returns the deleted record.

| Argument | Type | Description |
| --- | --- | --- |
| `where` | `Object` | Exact-match conditions that are guaranteed to match exactly one record. |

## Notes

- Clone the initial records when constructing the `MiniORM`; later external mutation of the input arrays or records should not mutate the stored data.
- `findMany()` should return shallow-cloned records so callers cannot directly mutate the stored data.
- You do not need argument validation, unique-constraint checks, or support for nested writes.
- `where` is top-level only. Nested filtering is out of scope.
- The delegate names come from the input object keys. `user` is not special.

## Hints

### Hint 1 : How does a delegate know its model?

### Hint 2 : Where can stored records leak?

### Hint 3 : What makes a `where` object match?

## Asked at these companies

- [[OpenAI]]

## 🤔 Thought Process

- **Immediate Recognition:** Building an in-memory Prisma-like ORM exposing dynamic model delegates (`db[model].create()`, `findMany()`, `findUnique()`, etc.).
- **Core Architecture:**
  - The constructor accepts an object/schema where keys are model names (e.g. `user`, `post`). Model names must NOT be hardcoded.
  - For each model, instantiate an isolated in-memory table store (e.g. array of records or `Map`).
  - Auto-generate primary keys (`id: 1, 2, ...`) if not supplied during `create()`.
  - Expose model delegates on `this`:
    - `create({ data })`: Clones record, assigns unique `id`, stores in table, returns cloned record.
    - `findMany({ where }?)`: Filters stored rows by exact matching criteria.
    - `findUnique({ where })`: Returns single matching record or `null`.
    - `update({ where, data })`: Finds matching record, merges `data` into record, returns updated clone.
    - `delete({ where })`: Removes matching record, returns deleted clone.
- **Defensive Immutability:** Always clone records on ingress (`create`, `update`) and egress (`findMany`, `findUnique`) so external mutations don't alter the internal database state.

---

## 🧠 Mental Model

Think of a **Prisma Client Multi-Table Repository**:
```
MiniORM Instance (db)
  ├── db.user  ──► User Model Delegate  ──► Private Users Array
  └── db.post  ──► Post Model Delegate  ──► Private Posts Array
```
Each delegate acts as an isolated DAO (Data Access Object) providing CRUD operations against its private table partition.

---

## 🔑 Key Concepts

- [[Closure]]
- [[Array Traversal]]
- [[Type Checking]]
- Dynamic Model Delegation / Factory Pattern
- In-memory Table Store & Auto-incrementing IDs
- Defensive Copying for State Isolation

---

## ⚠️ Edge Cases / Traps

- **Hardcoding Model Names:** Model names are dynamic (e.g. `products`, `orders`). You must iterate over the constructor's input keys to generate delegates dynamically.
- **External Mutation Leakage:** If a user modifies the object returned by `findUnique`, the database record must NOT change. Return `{ ...record }`.
- **`findUnique` Not Found:** Must return `null` (not `undefined` or throwing), matching Prisma conventions.
- **Auto-Increment Collisions:** If records are deleted and new ones inserted, auto-incrementing ID counters must always advance forward and never reuse existing IDs.

---

## ⭐ Interview Takeaway

- Generate delegates inside the constructor using `Object.keys(schema).forEach(model => { ... })`.
- Keep private state isolated in a closure per model:
  `const rows = []; let nextId = 1;`
- Always return cloned records: `return { ...row }`.
- Top-level `where` filter matches with `Object.entries(where).every(([k, v]) => row[k] === v)`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does Prisma use model delegates like `db.user.findMany()` instead of `db.findMany('user')`?" (Provides direct autocomplete, type safety, and clean namespace segregation).
- "Why is defensive copying necessary in in-memory databases?" (Prevents client code mutations from corrupting stored state).

### Follow-up Questions
- "How would you support operator filters like `gte`, `lte`, and `in`?" (Answered in Mini ORM II).
- "How would you handle relations between models (e.g. user with posts)?" (Answered in Mini ORM III).

### Conceptual Questions
- "How does the ActiveRecord pattern differ from the Data Mapper pattern used here?" (ActiveRecord models contain both data and DB methods on row instances; Data Mapper/Delegates separate domain entities from persistence operations).

---

## 🔄 Variations

- **Mini ORM II:** Rich operator filtering (`gt`, `lt`, `in`) and sorting.
- **Mini ORM III:** Relations (`include`) and projections (`select`).
- **Drizzle Query Builder:** Method-chaining query syntax instead of Prisma delegate methods.

---

## 📝 Revision Notes

- Delegate structure:
```javascript
export default class MiniORM {
  constructor(schema) {
    for (const model of Object.keys(schema)) {
      const records = [];
      let autoId = 1;

      this[model] = {
        create: ({ data }) => {
          const id = data.id !== undefined ? data.id : autoId++;
          if (data.id === undefined) autoId = Math.max(autoId, id + 1);
          const record = { id, ...data };
          records.push(record);
          return { ...record };
        },
        findMany: (query = {}) => {
          const { where } = query;
          let res = records;
          if (where) {
            res = res.filter(r => Object.entries(where).every(([k, v]) => r[k] === v));
          }
          return res.map(r => ({ ...r }));
        },
        findUnique: ({ where }) => {
          const found = records.find(r => Object.entries(where).every(([k, v]) => r[k] === v));
          return found ? { ...found } : null;
        },
      };
    }
  }
}
```

---

## Official Solution

## Mini Object-relational Mapper ( Official solution )

Premium
Languages
The first design choice is the internal data model. Once records are stored by model name and cloned at the boundaries, each delegate method becomes small and predictable.

## Solution

The recommended model is a `Map` from model name to an array of records, plus one generated delegate object per model name.

That gives three useful properties:

1. Each model's data is isolated from the others.
2. The constructor can clone the input so outside mutations do not leak into the ORM.
3. The delegate methods can stay thin wrappers around shared helper methods.

Only `_data` should own mutable records. Anything crossing the ORM boundary is shallow-cloned either on the way in or on the way out.

That produces a structure like:

```javascript
_data = Map(2) {
  'user' => [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ],
  'post' => [
    { id: 10, title: 'Hello' },
  ],
};
```

### Delegate generation walkthrough

The constructor should inspect the input model names and generate delegates from those keys:

```javascript
new MiniORM({
  article: [...],
  comment: [...],
});
```

That instance should expose `db.article` and `db.comment` because those are the keys in the data object. There should not be any special case for a model named `user`, `post`, or `product`.

Each generated delegate can close over its model name and forward to shared helpers:

```javascript
findMany(args) -> _findMany('article', args)
create(args) -> _create('article', args)
```

That keeps the public API model-specific while the shared helper logic remains reusable.

From there, each operation is straightforward:

- `findMany()` filters with exact top-level equality and returns shallow-cloned records.
- `create()` clones the input record before storing it and returns another clone.
- `update()` finds the first matching record, shallow-merges `data`, stores it, and returns a clone.
- `delete()` removes the first matching record and returns a clone of the removed value.

| Operation | Reads from storage | Writes to storage | Returns |
| --- | --- | --- | --- |
| `findMany` | Matching records | Nothing | New array of shallow clones |
| `create` | Target model array | Shallow clone of `data` appended | Shallow clone of created record |
| `update` | First exact match | Shallow-merged replacement | Shallow clone of updated record |
| `delete` | First exact match | Removes that record | Shallow clone of removed record |

Because the question only requires shallow cloning, nested objects and arrays can still share references with the stored records.

That boundary is what keeps the interview solution small: it protects top-level records from accidental caller mutation without implementing a general-purpose deep clone or schema validator.

```jsx
/**
 * @typedef {Record<string, unknown>} ORMRecord
 * @typedef {Record<string, Array<ORMRecord>>} ORMData
 * @typedef {Record<string, unknown>} ExactWhere
 * @typedef {{ where?: ExactWhere }} FindManyArgs
 * @typedef {{ data: ORMRecord }} CreateArgs
 * @typedef {{ where: ExactWhere, data: Partial<ORMRecord> }} UpdateArgs
 * @typedef {{ where: ExactWhere }} DeleteArgs
 * @typedef {{
 *   findMany(args?: FindManyArgs): Array<ORMRecord>,
 *   create(args: CreateArgs): ORMRecord,
 *   update(args: UpdateArgs): ORMRecord,
 *   delete(args: DeleteArgs): ORMRecord,
 * }} ModelDelegate
 */
function cloneRecord(record) {
  return { ...record };
}

function cloneRecords(records) {
  return records.map(cloneRecord);
}

function matchesWhere(record, where = {}) {
  return Object.entries(where).every(
    ([field, value]) => record[field] === value,
  );
}

export default class MiniORM {
  /**
   * @param {ORMData} data
   */
  constructor(data) {
    this._data = new Map();

    Object.entries(data).forEach(([modelName, records]) => {
      // Clone once on ingest so later outside mutations do not leak into the store.
      this._data.set(modelName, cloneRecords(records));
      this[modelName] = this._createDelegate(modelName);
    });
  }

  /**
   * @param {string} modelName
   * @returns {ModelDelegate}
   */
  _createDelegate(modelName) {
    return {
      findMany: (args = {}) => this._findMany(modelName, args),
      create: (args) => this._create(modelName, args),
      update: (args) => this._update(modelName, args),
      delete: (args) => this._delete(modelName, args),
    };
  }

  /**
   * @param {string} modelName
   * @param {FindManyArgs} [args={}]
   * @returns {Array<ORMRecord>}
   */
  _findMany(modelName, args = {}) {
    const records = this._getModelRecords(modelName);
    const { where } = args;

    // Reads clone on the way out too, so callers cannot mutate stored records in place.
    return cloneRecords(
      records.filter((record) => matchesWhere(record, where)),
    );
  }

  /**
   * @param {string} modelName
   * @param {CreateArgs} args
   * @returns {ORMRecord}
   */
  _create(modelName, args) {
    const records = this._getModelRecords(modelName);
    const nextRecord = cloneRecord(args.data);

    records.push(nextRecord);

    return cloneRecord(nextRecord);
  }

  /**
   * @param {string} modelName
   * @param {UpdateArgs} args
   * @returns {ORMRecord}
   */
  _update(modelName, args) {
    const records = this._getModelRecords(modelName);
    const recordIndex = records.findIndex((record) =>
      matchesWhere(record, args.where),
    );
    const nextRecord = {
      ...records[recordIndex],
      ...args.data,
    };

    records[recordIndex] = nextRecord;

    return cloneRecord(nextRecord);
  }

  /**
   * @param {string} modelName
   * @param {DeleteArgs} args
   * @returns {ORMRecord}
   */
  _delete(modelName, args) {
    const records = this._getModelRecords(modelName);
    const recordIndex = records.findIndex((record) =>
      matchesWhere(record, args.where),
    );
    const [deletedRecord] = records.splice(recordIndex, 1);

    return cloneRecord(deletedRecord);
  }

  /**
   * @param {string} modelName
   * @returns {Array<ORMRecord>}
   */
  _getModelRecords(modelName) {
    return this._data.get(modelName) ?? [];
  }
}
```

## Edge cases

- Delegate names must come from the input object keys; no model name is special.
- Initial data, created data, and returned records need shallow cloning, but nested references are intentionally shared.
- `where` matching is exact top-level `===` comparison across all provided fields.
- The prompt guarantees update and delete matches, so argument validation and not-found handling stay out of scope.

## Notes

- Using a `Map` keeps model lookup straightforward and avoids mixing stored data with helper properties on the instance.
- Exact-match filtering is just an `every()` over the `where` entries.
- Shallow cloning is enough for this question, so nested objects and arrays may still share references.

## Techniques

- Object-oriented programming
- Dynamic object properties
- Shallow cloning
- Filtering arrays of records

## Resources

- [Prisma Client overview](https://www.prisma.io/docs/orm/prisma-client)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A stored user has `settings: { theme: 'dark', alerts: true }`. The caller runs:

```javascript
db.user.update({
  where: { id: 1 },
  data: { settings: { theme: 'light' } },
});
```

Why does `alerts` disappear, and what must the caller supply if it wants to keep that setting?

Your notes (optional)
