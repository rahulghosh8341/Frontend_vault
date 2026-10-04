---
title: "Mini Object-relational Mapper II"
aliases:
  - "miniOrmII"
  - "Mini Object-relational Mapper II"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Mini Object-relational Mapper II

> [!info] Problem
> Implement a simplified Prisma-like in-memory object-relational mapper (ORM) with richer filtering and sorting

## Problem

## Mini Object-relational Mapper II

This is a follow-up to [Mini Object-relational Mapper](/questions/javascript/mini-orm).

In the base question, `findMany()` only supported exact top-level equality. In this question, the write APIs stay the same, but `findMany()` becomes more expressive.

Implement an enhanced `MiniORM` class with support for operator-based filtering and sorting.

Like the base question, model names are still arbitrary. The instance properties should come from the constructor input keys, not hardcoded names like `products`.

## Examples

```javascript
const db = new MiniORM({
  product: [
    { id: 1, name: 'Keyboard', price: 40, category: 'hardware' },
    { id: 2, name: 'Notebook', price: 12, category: 'stationery' },
    { id: 3, name: 'Monitor', price: 220, category: 'hardware' },
    { id: 4, name: 'Mouse Pad', price: 18, category: 'hardware' },
  ],
});

db.product.findMany({
  where: {
    price: { gte: 10, lt: 50 },
    category: { in: ['hardware', 'accessories'] },
    name: { contains: 'o' },
  },
  orderBy: { price: 'asc' },
});
// [
//   { id: 4, name: 'Mouse Pad', price: 18, category: 'hardware' },
//   { id: 1, name: 'Keyboard', price: 40, category: 'hardware' },
// ]
```

## findMany() changes

`model.findMany([args])`

### args.where

Each field can now be either:

- an exact-match value, as in the original question, or
- an operator object with one or more of the following keys:
  - `in`
  - `gt`
  - `gte`
  - `lt`
  - `lte`
  - `contains`

Records should still match with AND semantics across different fields. If multiple operators are provided for the same field, that field must satisfy all of them.

### args.orderBy

An optional object containing exactly one field name whose value is either `'asc'` or `'desc'`.

Apply sorting after filtering.

## Notes

- `create()`, `update()`, and `delete()` keep the same behavior as in [Mini Object-relational Mapper](/questions/javascript/mini-orm).
- `update()` and `delete()` still use exact top-level equality in their `where` clauses.
- `contains` only needs to work for string fields.
- Sorting should preserve insertion order when records have equal values for the `orderBy` field.
- You do not need `skip`, `take`, nested filtering, `OR`, or `NOT`.
- Inputs are guaranteed to be valid.
- `db.user` is only one possible delegate. `db.product`, `db.article`, and other model names should work too.

## Hints

### Hint 1 : Is every object an operator condition?

### Hint 2 : How many checks must pass?

### Hint 3 : Which order should a query change?

## Asked at these companies

- [[OpenAI]]

## 🤔 Thought Process

- **Immediate Recognition:** Enhancing `MiniORM` with operator-based filtering (`equals`, `gt`, `gte`, `lt`, `lte`, `in`, `notIn`) and sorting (`orderBy: { field: 'asc' | 'desc' }`).
- **Filtering Logic Enhancement:**
  - In Part I, `where: { age: 25 }` performed direct equality.
  - In Part II, `where: { age: { gte: 18, lt: 65 }, status: { in: ['active', 'pending'] } }`.
  - Operator evaluation:
    - Direct primitive value: check `row[field] === val`.
    - Object value: iterate through operator keys:
      - `equals`: `row[field] === opVal`
      - `gt`: `row[field] > opVal`
      - `gte`: `row[field] >= opVal`
      - `lt`: `row[field] < opVal`
      - `lte`: `row[field] <= opVal`
      - `in`: `Array.isArray(opVal) && opVal.includes(row[field])`
      - `notIn`: `Array.isArray(opVal) && !opVal.includes(row[field])`
- **Sorting Logic (`orderBy`):**
  - Accepts `{ [field]: 'asc' | 'desc' }` or array of order descriptors.
  - Comparator compares values: for `'asc'`, `a[field] < b[field] ? -1 : 1`; reverse for `'desc'`.

---

## 🧠 Mental Model

Think of a **Filter & Sort Predicate Pipeline**:
```
Rows
 │
 ├──► Where Predicate Evaluator
 │      ├── Direct Equality? (row.role === 'admin')
 │      └── Operator Object? (row.age >= 18 && row.age < 65)
 │
 ├──► Sorter
 │      └── Compare by orderBy field ('asc' vs 'desc')
 │
 └──► Emit Cloned Results
```

---

## 🔑 Key Concepts

- [[Range Checking]]
- [[Set Lookup]]
- [[Array Traversal]]
- Operator Dispatch Pattern
- Multi-field sorting & tie-breaking
- Prisma query syntax parity

---

## ⚠️ Edge Cases / Traps

- **Mixing Direct Values and Operator Objects:** `where: { name: 'Alice', age: { gt: 20 } }`. Code must check `typeof val === 'object' && val !== null` before inspecting operator keys.
- **`null` Equality:** `where: { deletedAt: null }` must check `row.deletedAt === null`.
- **Missing or Undefined Fields:** Ensure comparisons on undefined properties fail gracefully without throwing.
- **Empty `in` Array:** `status: { in: [] }` should match zero records.

---

## ⭐ Interview Takeaway

- Build an operator dispatcher map:
  ```javascript
  const OPERATORS = {
    equals: (v, target) => v === target,
    gt: (v, target) => v > target,
    gte: (v, target) => v >= target,
    lt: (v, target) => v < target,
    lte: (v, target) => v <= target,
    in: (v, target) => target.includes(v),
    notIn: (v, target) => !target.includes(v),
  };
  ```
- Sorting comparator: always clone before sorting (`[...rows].sort()`) because `Array.prototype.sort()` mutates in place!

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does `Array.prototype.sort()` require cloning before execution in query methods?" (Native `.sort()` mutates the source array in place, corrupting the stored database order).
- "How do you distinguish an operator filter object from a nested relation filter?" (Operator objects only contain recognized operator keys like `gt`, `lt`, `in`).

### Follow-up Questions
- "How would you support compound logical operators like `OR` and `AND`?"
- "How would you handle sorting by multiple columns with tie-breakers?"

### Conceptual Questions
- "How do real databases evaluate `IN (...)` queries efficiently?" (Using B-tree index lookups or temporary hash tables rather than linear scans).

---

## 🔄 Variations

- **Mini ORM I:** Basic equality matching only.
- **Mini ORM III:** Relations (`include`) and projections (`select`).
- **Data Selection:** SQL-like in-memory filtering utility.

---

## 📝 Revision Notes

- Operator matching helper:
```javascript
function matchesCondition(rowValue, condition) {
  if (condition === null || typeof condition !== 'object') {
    return rowValue === condition;
  }
  for (const [op, target] of Object.entries(condition)) {
    if (op === 'equals' && rowValue !== target) return false;
    if (op === 'gt' && !(rowValue > target)) return false;
    if (op === 'gte' && !(rowValue >= target)) return false;
    if (op === 'lt' && !(rowValue < target)) return false;
    if (op === 'lte' && !(rowValue <= target)) return false;
    if (op === 'in' && !target.includes(rowValue)) return false;
    if (op === 'notIn' && target.includes(rowValue)) return false;
  }
  return true;
}
```

---

## Official Solution

## Mini Object-relational Mapper II ( Official solution )

Premium
Languages
Mini ORM II reuses the same storage and delegate model from [Mini Object-relational Mapper](/questions/javascript/mini-orm). The extra work is all inside `findMany()`.

## Solution

Keep the part I storage model, but make `findMany()` a query pipeline over stored records:

1. Read the stored records for the model.
2. Filter them with `where`.
3. Sort the matched records with `orderBy`.
4. Return shallow-cloned results.

The key addition is supporting two kinds of `where` values:

- Plain values for exact equality.
- Operator objects such as `{ gte: 18, lt: 30 }`.

The compatibility rule is important: an object is treated as an operator object only if it contains a recognized operator key. Otherwise it falls back to the exact-match behavior from part I.

For each field condition:

- exact values use `===`
- `in` checks membership in an array
- `gt`, `gte`, `lt`, and `lte` compare with the stored field value
- `contains` checks substring membership for strings

Then `orderBy` is just a single-field sort applied after filtering.

The read path is therefore "predicate first, ordering second, cloning last". If cloning happens too early, the filtering code does extra work; if sorting happens before filtering, records that will be discarded can still influence comparator work; if cloning is skipped, callers can mutate stored records through the returned array.

### Worked filtering example

Given this condition:

```javascript
{
  price: { gte: 10, lt: 50 },
  category: { in: ['hardware', 'accessories'] },
}
```

a record has to pass every top-level field. For `price`, it must pass both operators on that one field. For `category`, it must appear in the allowed list. A record that passes the price range but has `category: 'stationery'` should still be excluded because different fields use AND semantics.

| Record | Price check | Category check | Included? |
| --- | --- | --- | --- |
| `{ price: 12, category: 'stationery' }` | passes | fails | no |
| `{ price: 60, category: 'hardware' }` | fails | passes | no |
| `{ price: 18, category: 'hardware' }` | passes | passes | yes |

Sorting should see only the records that survived filtering. Cloning should happen after sorting so the code can sort stored record references without doing extra work, then return safe shallow copies to the caller.

```jsx
const OPERATOR_KEYS = new Set(['in', 'gt', 'gte', 'lt', 'lte', 'contains']);

/**
 * @typedef {Record<string, unknown>} ORMRecord
 * @typedef {Record<string, Array<ORMRecord>>} ORMData
 */
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
  // Objects without recognized operators still use the exact-match behavior from part I.
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
   */
  constructor(data) {
    this._data = new Map();

    Object.entries(data).forEach(([modelName, records]) => {
      this._data.set(modelName, cloneRecords(records));
      this[modelName] = this._createDelegate(modelName);
    });
  }

  _createDelegate(modelName) {
    return {
      findMany: (args = {}) => this._findMany(modelName, args),
      create: (args) => this._create(modelName, args),
      update: (args) => this._update(modelName, args),
      delete: (args) => this._delete(modelName, args),
    };
  }

  _findMany(modelName, args = {}) {
    const records = this._getModelRecords(modelName);
    const filteredRecords = records.filter((record) =>
      matchesWhere(record, args.where),
    );

    // Sort the matched records before cloning them for the caller.
    return cloneRecords(sortRecords(filteredRecords, args.orderBy));
  }

  _create(modelName, args) {
    const records = this._getModelRecords(modelName);
    const nextRecord = cloneRecord(args.data);

    records.push(nextRecord);

    return cloneRecord(nextRecord);
  }

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

  _delete(modelName, args) {
    const records = this._getModelRecords(modelName);
    const recordIndex = records.findIndex((record) =>
      matchesExactWhere(record, args.where),
    );
    const [deletedRecord] = records.splice(recordIndex, 1);

    return cloneRecord(deletedRecord);
  }

  _getModelRecords(modelName) {
    return this._data.get(modelName) ?? [];
  }
}
```

## Edge cases

- Multiple operators on the same field also use AND semantics; a field must satisfy all provided operators.
- `contains` is intentionally string-only. Non-string stored values should not match a `contains` operator.
- `orderBy` contains exactly one field, so the comparator only needs one key and one direction.
- Returned arrays and records are shallow clones, while nested object references are preserved.
- `update()` and `delete()` deliberately keep exact top-level `where` matching, so the richer operator language is confined to reads.

## Common pitfalls

- Treating every object condition as an operator object breaks exact-match behavior from the first question. Only objects with recognized operator keys should take the operator path.
- Filtering and sorting after cloning creates extra work and makes the clone helpers responsible for query semantics.
- Extending `update()` or `delete()` with the richer operator syntax goes beyond this follow-up; the new query language applies only to reads.

## Notes

- Treating unrecognized objects as exact-match values preserves backwards compatibility with part I.

## Techniques

- Object-oriented programming
- Query predicate composition
- Sorting arrays of records
- Preserving backwards-compatible APIs

## Resources

- [Prisma Client filtering and sorting](https://www.prisma.io/docs/orm/prisma-client/queries/filtering-and-sorting)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
The read API treats an object condition as operators only when it contains a recognized key such as `gte` or `in`. A refactor treats every non-null object as an operator condition and runs only the recognized checks.

What goes wrong for `where: { settings: wantedSettings }` when `wantedSettings` is `{ theme: 'dark' }`?

Your notes (optional)
