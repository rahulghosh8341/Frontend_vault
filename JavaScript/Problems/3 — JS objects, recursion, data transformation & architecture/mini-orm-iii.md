---
title: "Mini Object-relational Mapper III"
aliases:
  - "miniOrmIII"
  - "Mini Object-relational Mapper III"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Mini Object-relational Mapper III

> [!info] Problem
> Implement a simplified Prisma-like in-memory object-relational mapper (ORM) with field selection and relation includes

## Problem

## Mini Object-relational Mapper III

This is a follow-up to [Mini Object-relational Mapper II](/questions/javascript/mini-orm-ii).

In this question, filtering and sorting still work the same way, but `findMany()` also needs to shape the returned data:

- `select` picks which top-level fields to return.
- `include` loads single-level relations using explicit relation metadata.

Both model names and relation names are arbitrary. They should be derived from the constructor inputs and relation metadata, not hardcoded.

## Examples

```javascript
const db = new MiniORM(
  {
    playlist: [
      { id: 1, name: 'Road Trip' },
      { id: 2, name: 'Focus' },
    ],
    track: [
      { id: 10, title: 'Intro', playlistId: 1 },
      { id: 11, title: 'Highway', playlistId: 1 },
      { id: 12, title: 'Deep Work', playlistId: 2 },
    ],
  },
  {
    playlist: {
      tracks: {
        model: 'track',
        type: 'many',
        sourceKey: 'id',
        targetKey: 'playlistId',
      },
    },
    track: {
      playlist: {
        model: 'playlist',
        type: 'one',
        sourceKey: 'playlistId',
        targetKey: 'id',
      },
    },
  },
);

db.playlist.findMany({
  select: { id: true, name: true },
  include: { tracks: true },
});
// [
//   {
//     id: 1,
//     name: 'Road Trip',
//     tracks: [
//       { id: 10, title: 'Intro', playlistId: 1 },
//       { id: 11, title: 'Highway', playlistId: 1 },
//     ],
//   },
//   {
//     id: 2,
//     name: 'Focus',
//     tracks: [{ id: 12, title: 'Deep Work', playlistId: 2 }],
//   },
// ]
```

## Additional constructor argument

### new MiniORM(data, relations)

`relations` is an optional object describing how models are connected.

Each relation definition has:

| Field | Type | Description |
| --- | --- | --- |
| `model` | `string` | The related model name. |
| `type` | `'one' \| 'many'` | Whether the relation returns one record or many. |
| `sourceKey` | `string` | The field on the current record to read from. |
| `targetKey` | `string` | The field on the related model to compare against. |

## findMany() additions

### args.select

An optional object whose keys are field names and whose values are booleans.

Return only the fields whose values are `true`.

### args.include

An optional object whose keys are relation names and whose values are booleans.

For each `true` relation:

- If the relation `type` is `'one'`, return the related record or `null`.
- If the relation `type` is `'many'`, return an array of related records in insertion order.

## Notes

- `select` and `include` apply only at the top level of `findMany()`.
- Included related records should be shallow clones, not live references into storage.
- Ignore relation names that are absent from the relation metadata, as well as include entries set to `false`.
- Nested `select`, nested `include`, and nested writes are out of scope.
- `create()`, `update()`, and `delete()` keep the same behavior as in the previous questions.
- Neither model names nor relation names are special-cased. Build them from the provided data and relation config.

## Hints

### Hint 1 : When should fields be removed?

### Hint 2 : What does relation metadata connect?

### Hint 3 : Where should selected and included data meet?

## Asked at these companies

- [[OpenAI]]

## 🤔 Thought Process

- **Immediate Recognition:** Extending `MiniORM` with single-level relation loading (`include: { posts: true }`) and field selection (`select: { id: true, name: true }`).
- **Relation Metadata:**
  - The constructor receives schema and relation metadata defining foreign keys:
    e.g. `{ user: { posts: { model: 'post', foreignKey: 'userId', type: 'many' } } }`.
- **Field Selection (`select`):**
  - If `select` is provided, output rows only include fields where `select[field] === true`.
  - If `select` is omitted, all scalar fields are returned.
- **Relation Inclusion (`include`):**
  - If `include: { [relName]: true }` is provided:
    - Look up relation definition from metadata.
    - Query related model table:
      - If `type === 'many'`: Find all rows where `relatedRow[foreignKey] === currentRow.id`.
      - If `type === 'one'`: Find single row matching foreign key.
    - Attach joined relation array or object onto the result object.
- **Constraint:** In Prisma, `select` and `include` cannot be used at the same top-level object without specific nesting, but `select` can project relations if specified.

---

## 🧠 Mental Model

Think of **Eager Loading in ORMs (N+1 Query Resolver)**:
```
Primary Query: Find Users
     │
     ▼
[ { id: 1, name: 'Alice' }, { id: 2, name: 'Bob' } ]
     │
     ├── Include 'posts'?
     │     └── Query Posts where userId in [1, 2]
     │     └── Attach posts: [ ... ] to each user
     │
     └── Apply Select Projections (id, name, posts)
```

---

## 🔑 Key Concepts

- [[Object Path Traversal]]
- [[Array Traversal]]
- [[Closure]]
- Eager Loading / Foreign Key Joins
- DTO Projection (`select`)
- Data Normalization vs Denormalization

---

## ⚠️ Edge Cases / Traps

- **`select` vs Full Row:** If `select` is specified, unselected fields MUST be dropped. If `select` includes relation fields, relations must be loaded and included in the projection.
- **One-to-Many vs One-to-One:** For `type: 'many'`, return an array `[]` (even if empty). For `type: 'one'`, return `null` if no match exists.
- **Immutability of Joined Entities:** Cloned copies of related records must be attached, preventing mutation leakage into the related model's table.
- **Arbitrary Model & Relation Names:** Never hardcode table or foreign key names; always inspect the dynamic relation schema.

---

## ⭐ Interview Takeaway

- Step 1: Filter and sort primary model rows using Mini ORM II mechanics.
- Step 2: For each row, resolve relations requested in `include`:
  ```javascript
  if (include) {
    for (const [relName, enabled] of Object.entries(include)) {
      if (enabled) {
        const meta = relations[model][relName];
        row[relName] = meta.type === 'many'
          ? this[meta.model].findMany({ where: { [meta.foreignKey]: row.id } })
          : this[meta.model].findUnique({ where: { id: row[meta.foreignKey] } });
      }
    }
  }
  ```
- Step 3: Apply `select` filtering to keep only requested keys.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How does Prisma handle relation loading under the hood?" (It executes a secondary batched query using `WHERE foreignKey IN (...)` and stitches records in memory).
- "Why does Prisma disallow using both `select` and `include` at the top level?" (Ambiguity over whether unselected scalar fields should be excluded while relations are included).

### Follow-up Questions
- "How would you prevent the N+1 problem if loading relations for 1,000 users?" (Batch all user IDs into a single `findMany({ where: { userId: { in: ids } } })` query, then group by user ID in a `Map`).
- "How would you support nested includes (e.g. user -> posts -> comments)?" (Recursively invoke `findMany` passing nested `include` blocks).

### Conceptual Questions
- "How does in-memory relational mapping compare to SQL JOIN queries?" (SQL JOIN flattens rows across a network boundary; ORM in-memory stitching maintains hierarchical object graphs).

---

## 🔄 Variations

- **Mini ORM I & II:** Foundation and filter/sort capabilities.
- **Drizzle Query Builder III:** Relational SQL `innerJoin` and `leftJoin`.
- **DataLoader Pattern:** Batching and caching database reads to solve N+1 problems.

---

## 📝 Revision Notes

- Projection & Inclusion flow:
```javascript
function shapeResult(records, select, include, resolveRelation) {
  return records.map(record => {
    const enriched = { ...record };

    if (include) {
      for (const [rel, val] of Object.entries(include)) {
        if (val) enriched[rel] = resolveRelation(record, rel);
      }
    }

    if (!select) return enriched;

    const projected = {};
    for (const [key, enabled] of Object.entries(select)) {
      if (enabled) projected[key] = enriched[key];
    }
    return projected;
  });
}
```

---

## Official Solution

## Mini Object-relational Mapper III ( Official solution )

Premium
Languages
This follow-up adds one extra layer after the part II query pipeline: result shaping.

## Solution

`findMany()` still reads as a pipeline. The only new step is that, after filtering and sorting, the returned records are now shaped.

That gives four jobs:

1. Filter the stored records.
2. Sort them.
3. Shape each result with `select`.
4. Attach requested relations with `include`.

That separation is the main correctness rule in this question:

- Filtering and sorting still work on the full stored records.
- `select` decides which top-level scalar fields to expose.
- `include` resolves relations from metadata using `sourceKey` and `targetKey`.

For relation loading:

- A `'one'` relation returns the first matching related record or `null`.
- A `'many'` relation returns all matching related records in insertion order.

Because relation definitions are explicit, there is no need to infer foreign keys from naming conventions.

If neither `select` nor `include` is passed, the shaped result is just a shallow clone of the filtered and sorted record, preserving the behavior from the previous parts.

That clone boundary applies to related records too. Returned rows should be safe to mutate from the caller's perspective without changing the ORM's internal arrays, even though the clone is only shallow.

### Worked shaping example

If a playlist query asks for:

```javascript
{
  select: { name: true },
  include: { tracks: true },
}
```

the selected scalar shape starts as `{ name: record.name }`, but relation lookup should still use the original stored playlist record. That matters because `select` might omit the `id` field that `tracks.sourceKey` needs. Treat `select` as output shaping, not as a mutation of the record used by later relation work.

For `include`, the relation metadata tells the loader exactly how to compare records:

```text
playlist.id -> track.playlistId
```

The ORM should read the source key from the current playlist, find matching rows in the target model, shallow-clone those related rows, and attach them under the requested relation name.

| Pipeline step | Input used | Output |
| --- | --- | --- |
| `where` | full stored records | matching stored records |
| `orderBy` | full matching records | sorted stored records |
| `select` | one sorted record | top-level output fields |
| `include` | original sorted record plus relation metadata | cloned related records attached to output |

Keeping relation lookup after `select` but based on the original record is the key design decision. It lets `select: { name: true }` and `include: { tracks: true }` work even though `id` was not selected.

```jsx
/**
 * @typedef {Record<string, unknown>} ORMRecord
 * @typedef {Record<string, Array<ORMRecord>>} ORMData
 * @typedef {Record<string, unknown>} ExactWhere
 * @typedef {{ in?: Array<unknown>, gt?: unknown, gte?: unknown, lt?: unknown, lte?: unknown, contains?: string }} FieldOperators
 * @typedef {unknown | FieldOperators} WhereValue
 * @typedef {Record<string, WhereValue>} WhereClause
 * @typedef {{ model: string, type: 'one' | 'many', sourceKey: string, targetKey: string }} RelationDefinition
 * @typedef {Record<string, Record<string, RelationDefinition>>} RelationMap
 * @typedef {Record<string, boolean>} SelectMap
 * @typedef {Record<string, boolean>} IncludeMap
 * @typedef {{ where?: WhereClause, orderBy?: Record<string, 'asc' | 'desc'>, select?: SelectMap, include?: IncludeMap }} FindManyArgs
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

const OPERATOR_KEYS = new Set(['in', 'gt', 'gte', 'lt', 'lte', 'contains']);

function cloneRecord(record) {
  return { ...record };
}

function cloneRecords(records) {
  return records.map(cloneRecord);
}

function isOperatorObject(value) {
  return (
    value != null &&
    !Array.isArray(value) &&
    typeof value === 'object' &&
    Object.keys(value).some((key) => OPERATOR_KEYS.has(key))
  );
}

function matchesExactWhere(record, where = {}) {
  return Object.entries(where).every(
    ([field, value]) => record[field] === value,
  );
}

function matchesFieldCondition(recordValue, condition) {
  if (!isOperatorObject(condition)) {
    return recordValue === condition;
  }

  if ('in' in condition && !condition.in.includes(recordValue)) {
    return false;
  }

  if ('gt' in condition && !(recordValue > condition.gt)) {
    return false;
  }

  if ('gte' in condition && !(recordValue >= condition.gte)) {
    return false;
  }

  if ('lt' in condition && !(recordValue < condition.lt)) {
    return false;
  }

  if ('lte' in condition && !(recordValue <= condition.lte)) {
    return false;
  }

  if (
    'contains' in condition &&
    (typeof recordValue !== 'string' ||
      !recordValue.includes(condition.contains))
  ) {
    return false;
  }

  return true;
}

function matchesWhere(record, where = {}) {
  return Object.entries(where).every(([field, condition]) =>
    matchesFieldCondition(record[field], condition),
  );
}

function sortRecords(records, orderBy) {
  if (orderBy == null) {
    return records;
  }

  const [field, direction] = Object.entries(orderBy)[0];
  const multiplier = direction === 'desc' ? -1 : 1;

  return records.slice().sort((recordA, recordB) => {
    if (recordA[field] === recordB[field]) {
      return 0;
    }

    return recordA[field] > recordB[field] ? multiplier : -multiplier;
  });
}

export default class MiniORM {
  /**
   * @param {ORMData} data
   * @param {RelationMap} [relations]
   */
  constructor(data, relations = {}) {
    this._data = new Map();
    this._relations = relations;

    Object.entries(data).forEach(([modelName, records]) => {
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
   * @param {FindManyArgs} [args]
   * @returns {Array<ORMRecord>}
   */
  _findMany(modelName, args = {}) {
    const records = this._getModelRecords(modelName);
    const filteredRecords = records.filter((record) =>
      matchesWhere(record, args.where),
    );
    const sortedRecords = sortRecords(filteredRecords, args.orderBy);

    // Shape only after filtering/sorting so those steps still see the full stored record.
    return sortedRecords.map((record) =>
      this._shapeRecord(modelName, record, args),
    );
  }

  /**
   * @param {string} modelName
   * @param {ORMRecord} record
   * @param {FindManyArgs} args
   * @returns {ORMRecord}
   */
  _shapeRecord(modelName, record, args) {
    const nextRecord =
      args.select == null
        ? cloneRecord(record)
        : this._applySelect(record, args.select);

    if (args.include == null) {
      return nextRecord;
    }

    Object.entries(args.include).forEach(([relationName, shouldInclude]) => {
      if (!shouldInclude) {
        return;
      }

      const relationDefinition = this._relations[modelName]?.[relationName];

      if (relationDefinition == null) {
        return;
      }

      // Includes resolve from the original stored record, not the select-trimmed shape.
      nextRecord[relationName] = this._resolveRelation(
        record,
        relationDefinition,
      );
    });

    return nextRecord;
  }

  /**
   * @param {ORMRecord} record
   * @param {SelectMap} select
   * @returns {ORMRecord}
   */
  _applySelect(record, select) {
    const nextRecord = {};

    Object.entries(select).forEach(([field, selected]) => {
      if (selected) {
        nextRecord[field] = record[field];
      }
    });

    return nextRecord;
  }

  /**
   * @param {ORMRecord} record
   * @param {RelationDefinition} relationDefinition
   * @returns {unknown}
   */
  _resolveRelation(record, relationDefinition) {
    const relatedRecords = this._getModelRecords(relationDefinition.model);
    const matches = relatedRecords.filter(
      (relatedRecord) =>
        relatedRecord[relationDefinition.targetKey] ===
        record[relationDefinition.sourceKey],
    );

    return relationDefinition.type === 'one'
      ? matches.length === 0
        ? null
        : cloneRecord(matches[0])
      : cloneRecords(matches);
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
      matchesExactWhere(record, args.where),
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
      matchesExactWhere(record, args.where),
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

- Unknown include keys are ignored instead of creating extra output fields.
- Include values set to `false` should not load that relation.
- A `'one'` relation with no match returns `null`; a `'many'` relation with no matches returns `[]`.
- Included records are shallow clones so callers cannot mutate the ORM's internal storage through query results.
- Relation and model names come only from the constructor data and relation metadata; they are not hardcoded.

## Notes

- Running `select` and `include` after filtering/sorting keeps the query pipeline easy to follow.
- Relation resolution should use the original stored record, not the selected shape, so missing selected keys do not break includes.
- Returning shallow-cloned related records prevents callers from mutating the internal store.

## Techniques

- Object-oriented programming
- Query pipelines
- Result shaping
- Relation resolution from metadata

## Resources

- [Prisma Client select fields](https://www.prisma.io/docs/orm/prisma-client/queries/select-fields)

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
What does the query return?

```javascript
const db = new MiniORM(
  {
    user: [{ id: 1, name: 'Ada' }],
    post: [{ id: 10, title: 'Hello', authorId: 1 }],
  },
  {
    user: {
      posts: {
        model: 'post',
        type: 'many',
        sourceKey: 'id',
        targetKey: 'authorId',
      },
    },
  },
);

const result = db.user.findMany({
  select: { name: true },
  include: { posts: true },
});
```
