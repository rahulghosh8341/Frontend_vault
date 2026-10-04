---
title: "Drizzle Query Builder II"
aliases:
  - "drizzleQueryBuilderII"
  - "Drizzle Query Builder II"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Drizzle Query Builder II

> [!info] Problem
> Implement a simplified in-memory query builder inspired by Drizzle ORM, with support for `select()` projections and aliases

## Problem

## Drizzle Query Builder II

This is a follow-up to [Drizzle Query Builder](/questions/javascript/drizzle-query-builder).

Like [Drizzle ORM's select API](https://orm.drizzle.team/docs/select), this follow-up adds projection support with `select({ ... })`.

In the first question, `db.select().from(table)` always returned full rows from the source table. In this question, add projection support with `select({ ... })`.

Like Drizzle ORM's [dynamic query building](https://orm.drizzle.team/docs/dynamic-query-building), each call to `db.select()` or `db.select(selection)` should create a new query builder. After that, `.from()`, `.where()`, `.orderBy()`, `.limit()`, and `.offset()` should mutate that query builder and return it for further chaining. Adding projections does not change this mutability model, and `.all()` should still return fresh result objects.

Implement the same helpers and query builder as before, but now `db.select(selection)` should be supported.

The `selection` object maps result keys to columns:

- `db.select({ id: users.id, displayName: users.name })`

The result should contain only those keys, using the selection object keys as aliases.

## Examples

```javascript
const users = table('users', {
  id: integer('id'),
  name: text('name'),
  email: text('email'),
  role: text('role'),
});

const db = drizzle({
  users: [
    { id: 1, name: 'Ada', email: 'ada@example.com', role: 'admin' },
    { id: 2, name: 'Grace', email: 'grace@example.com', role: 'editor' },
  ],
});

db.select({
  userId: users.id,
  displayName: users.name,
})
  .from(users)
  .orderBy(asc(users.id))
  .all();
// [
//   { userId: 1, displayName: 'Ada' },
//   { userId: 2, displayName: 'Grace' },
// ]
```

## select() changes

### db.select(selection)

When `selection` is provided, each key in the returned result object should map to the value of the referenced column in the current row.

For example:

```javascript
db.select({
  id: users.id,
  name: users.name,
})
  .from(users)
  .all();
// [
//   { id: 1, name: 'Ada' },
//   { id: 2, name: 'Grace' },
// ]
```

### db.select()

Calling `select()` with no arguments should keep the old behavior from part I and return full rows.

## Notes

- All filtering, sorting, `offset()`, and `limit()` behavior from part I stays the same.
- `selection` values only need to support columns from the `from()` table in this part.
- The result object should be fresh for every returned row.
- You do not need nested select objects, joins, aggregates, whole-table selects, or computed expressions.
- Inputs are guaranteed to be valid.

## Follow-up

Implement [Drizzle Query Builder III](/questions/javascript/drizzle-query-builder-iii) to add joins and joined result shapes.

## Hints

### Hint 1 : When should projection happen?

### Hint 2 : What determines each output property?

## 🤔 Thought Process

- **Immediate Recognition:** Enhancing the Drizzle query builder with column projection (`select({ alias: table.column })`).
- **Core Requirement: Projections:**
  - `db.select()`: Returns full rows (default behavior from Part I).
  - `db.select({ id: users.id, fullName: users.name })`: Projects only specified keys, remapping column references to the chosen alias keys.
- **Pipeline Adjustment:**
  - Standard filtering (`where`), sorting (`orderBy`), and pagination (`offset`, `limit`) run first on the source rows.
  - Projection step runs *last*: For each resulting row, construct a new object by iterating the projection schema and extracting values:
    `projectedRow[alias] = row[col.name]`.
- **Builder State Preservation:**
  - Store `selection` passed to `db.select(selection)` on the builder instance.
  - Chaining methods continue to mutate the builder and return `this`.

---

## 🧠 Mental Model

Think of **SQL `SELECT col1 AS alias1, col2 AS alias2` Projection**:
```
Source Row: { id: 1, first_name: 'John', last_name: 'Doe', age: 30 }
        │
    Projection Config: { uid: users.id, name: users.first_name }
        │
        ▼
Projected Output: { uid: 1, name: 'John' }
```

---

## 🔑 Key Concepts

- [[Method Chaining]]
- [[Object Path Traversal]]
- [[Closure]]
- Data Projection / DTO Mapping
- Pipeline execution ordering (Filter -> Sort -> Paginate -> Project)

---

## ⚠️ Edge Cases / Traps

- **Projection Execution Timing:** Projection must happen *after* `where` and `orderBy`. If you project first, the row loses unselected columns that may be required by `where` clauses or sorting criteria.
- **Empty / Null Projection:** `db.select()` with no arguments must retain the original full row behavior from Part I.
- **Preserving Prototypes:** Ensure projected objects are clean plain `{}` objects.
- **Missing Columns in Source Rows:** If a projected column is undefined in a source row, set the alias key to `undefined` or omit appropriately per spec.

---

## ⭐ Interview Takeaway

- Projection is always the **final stage** of the query execution pipeline:
  `filter` -> `sort` -> `slice` -> `project`.
- Projection map pattern:
  ```javascript
  function projectRow(row, selection) {
    if (!selection) return { ...row };
    const out = {};
    for (const [alias, col] of Object.entries(selection)) {
      out[alias] = row[col.name];
    }
    return out;
  }
  ```

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why must column projection be evaluated after `where` and `orderBy`?" (The condition or sort expression may reference fields that are not included in the final projected output).
- "How does Drizzle achieve compile-time type safety with `select({ ... })` in TypeScript?" (Using TypeScript mapped types and generics that infer the shape of the selection object).

### Follow-up Questions
- "How would you handle computed fields or SQL expressions in the projection (e.g. `count()`, `concat()`)?"
- "What changes when adding table joins with conflicting column names?" (Answered in Drizzle Query Builder III).

### Conceptual Questions
- "How does projection in ORMs optimize performance in relational databases?" (In real SQL, `SELECT col1` reduces network I/O and memory overhead by avoiding pulling unused large text/blob columns).

---

## 🔄 Variations

- **Drizzle Query Builder I:** Base builder without projections.
- **Drizzle Query Builder III:** Adding `innerJoin` and `leftJoin`.
- **GraphQL Field Resolver:** Selecting a sparse subset of an object graph based on a client query.

---

## 📝 Revision Notes

- Clean projection handling in `.all()`:
```javascript
all() {
  let result = this.tableInstance._rows.map(r => ({ ...r }));

  if (this.whereCondition) {
    result = result.filter(this.whereCondition);
  }
  if (this.sortComparator) {
    result.sort(this.sortComparator);
  }
  const start = this.offsetVal ?? 0;
  const end = this.limitVal !== undefined ? start + this.limitVal : undefined;
  result = result.slice(start, end);

  if (this.selection) {
    result = result.map(row => {
      const projected = {};
      for (const [alias, col] of Object.entries(this.selection)) {
        projected[alias] = row[col.name];
      }
      return projected;
    });
  }

  return result;
}
```

---

## Official Solution

## Drizzle Query Builder II ( Official solution )

Premium
Languages
Part II reuses the same table helpers, predicates, and query pipeline from [Drizzle Query Builder](/questions/javascript/drizzle-query-builder). The only new work is shaping rows with `select({ ... })`, similar to the projection style used by Drizzle ORM.

## Solution

This is a small extension of part I: keep the same query pipeline, then add one final projection step that shapes each output row.

`selection` is a recipe for the result object, not a change to how rows are filtered or sorted. The default implementation to internalize is therefore:

1. Keep the same row-context pipeline for filtering, sorting, offset, and limit.
2. Store the optional `selection` object on the query builder.
3. At the very end, either:
  - clone the full source row when no `selection` was provided, or
  - build a new result object by reading each selected column from the current row context.

That final branch defines the public API of `select()`: it does not change which rows qualify, only which fields are exposed after the row pipeline has finished. This is why `selection` belongs on the query builder as metadata until `.all()` materializes results.

Because the selection object uses aliases as keys, the output shape is completely controlled by the caller:

```javascript
db.select({
  userId: users.id,
  displayName: users.name,
});
```

That should become:

```javascript
{ userId: row.id, displayName: row.name }
```

### Projection timing

Projection should be the final step, not an early transformation. The original row context still needs to contain full rows while filtering, sorting, offset, and limit run:

```javascript
db.select({ displayName: users.name })
  .from(users)
  .where(eq(users.role, 'admin'))
  .orderBy(desc(users.age))
  .all();
```

Even though the final result only includes `displayName`, the query still needs `users.role` for filtering and `users.age` for sorting. Keeping full row contexts until the end lets every query helper read from the same source of truth. Only after the query has selected the final contexts should it turn each context into the caller's requested output shape.

For a projected query, the pipeline is:

| Stage | Data shape | Why it stays this way |
| --- | --- | --- |
| `from(users)` | Full row context | All columns remain available. |
| `where(...)` | Full row context | Predicates may read columns outside the projection. |
| `orderBy(...)` | Full row context | Sort keys may also be unselected. |
| `offset()` / `limit()` | Full row context | These operate on row positions. |
| Final projection | Fresh result object | Aliases control the returned shape. |

Fresh projection objects matter because callers can mutate returned rows. The database storage should stay isolated from those mutations, even though selected nested values are still shared references.

Projecting earlier looks simpler, but it breaks as soon as `where()` or `orderBy()` reads an unselected column. Keeping full row contexts until the final mapping is the tradeoff that preserves part I behavior while adding aliases.

```jsx
/**
 * @typedef {Record<string, unknown>} Row
 * @typedef {Record<string, Array<Row>>} DatabaseData
 * @typedef {Record<string, Row | null>} RowContext
 * @typedef {{ kind: 'integer' | 'text', name: string }} ColumnBuilder
 * @typedef {{ kind: 'integer' | 'text', name: string, tableName: string }} Column
 * @typedef {(context: RowContext) => boolean} Condition
 * @typedef {{ column: Column, direction: 'asc' | 'desc' }} Ordering
 */

/**
 * @param {string} name
 * @returns {ColumnBuilder}
 */
export function integer(name) {
  return {
    kind: 'integer',
    name,
  };
}

/**
 * @param {string} name
 * @returns {ColumnBuilder}
 */
export function text(name) {
  return {
    kind: 'text',
    name,
  };
}

/**
 * @param {string} name
 * @param {Record<string, ColumnBuilder>} columns
 * @returns {{ _: { name: string } } & Record<string, Column | { name: string }>}
 */
export function table(name, columns) {
  const table = {
    _: {
      name,
    },
  };

  Object.entries(columns).forEach(([key, column]) => {
    // Columns carry their table name so predicates can resolve values later.
    table[key] = {
      kind: column.kind,
      name: column.name,
      tableName: name,
    };
  });

  return table;
}

function isColumn(value) {
  return (
    typeof value === 'object' &&
    value !== null &&
    'tableName' in value &&
    'name' in value
  );
}

function cloneRow(row) {
  return { ...row };
}

function cloneRows(rows) {
  return rows.map(cloneRow);
}

function resolveColumnValue(column, context) {
  const row = context[column.tableName];

  if (row == null) {
    return undefined;
  }

  return row[column.name];
}

function resolveValue(value, context) {
  return isColumn(value) ? resolveColumnValue(value, context) : value;
}

function projectSelection(selection, context) {
  const result = {};

  Object.entries(selection).forEach(([alias, column]) => {
    // Selection aliases are projected by reading from the current row context.
    result[alias] = resolveColumnValue(column, context);
  });

  return result;
}

function compareValues(left, right) {
  if (left === right) {
    return 0;
  }

  return left > right ? 1 : -1;
}

function compareContexts(left, right, orderings) {
  for (const ordering of orderings) {
    const comparison = compareValues(
      resolveColumnValue(ordering.column, left),
      resolveColumnValue(ordering.column, right),
    );

    if (comparison !== 0) {
      return ordering.direction === 'asc' ? comparison : -comparison;
    }
  }

  return 0;
}

/**
 * @param {Column} left
 * @param {unknown} right
 * @returns {Condition}
 */
export function eq(left, right) {
  return (context) =>
    resolveColumnValue(left, context) === resolveValue(right, context);
}

/**
 * @param {...Condition} conditions
 * @returns {Condition}
 */
export function and(...conditions) {
  return (context) => conditions.every((condition) => condition(context));
}

/**
 * @param {...*} conditions
 * @returns {*}
 */
export function or(...conditions) {
  return (context) => conditions.some((condition) => condition(context));
}

/**
 * @param {Column} column
 * @returns {Ordering}
 */
export function asc(column) {
  return {
    column,
    direction: 'asc',
  };
}

/**
 * @param {*} column
 * @returns {*}
 */
export function desc(column) {
  return {
    column,
    direction: 'desc',
  };
}

class SelectQuery {
  constructor(data, selection) {
    this._data = data;
    this._selection = selection;
    this._fromTable = null;
    this._condition = null;
    this._orderings = [];
    this._limit = null;
    this._offset = 0;
  }

  from(table) {
    this._fromTable = table;
    return this;
  }

  where(condition) {
    this._condition = condition;
    return this;
  }

  orderBy(...orderings) {
    this._orderings = orderings;
    return this;
  }

  limit(count) {
    this._limit = count;
    return this;
  }

  offset(count) {
    this._offset = count;
    return this;
  }

  all() {
    const table = this._fromTable;
    const tableName = table._.name;
    // A row context keeps lookups uniform for filters, projections, and sorting.
    let contexts = (this._data.get(tableName) ?? []).map((row) => ({
      [tableName]: row,
    }));

    if (this._condition != null) {
      contexts = contexts.filter((context) => this._condition(context));
    }

    if (this._orderings.length > 0) {
      contexts = contexts
        .slice()
        .sort((left, right) => compareContexts(left, right, this._orderings));
    }

    if (this._offset > 0) {
      contexts = contexts.slice(this._offset);
    }

    if (this._limit != null) {
      contexts = contexts.slice(0, this._limit);
    }

    if (this._selection == null) {
      return contexts.map((context) => cloneRow(context[tableName]));
    }

    return contexts.map((context) =>
      projectSelection(this._selection, context),
    );
  }
}

class DrizzleDatabase {
  constructor(data) {
    this._data = new Map();

    Object.entries(data).forEach(([tableName, rows]) => {
      this._data.set(tableName, cloneRows(rows));
    });
  }

  select(selection) {
    return new SelectQuery(this._data, selection ?? null);
  }
}

/**
 * @param {DatabaseData} data
 * @returns {object}
 */
export default function drizzle(data) {
  return new DrizzleDatabase(data);
}
```

## Edge cases

- `select()` with no arguments must keep the part I behavior and return full-row clones.
- `select(selection)` should create a fresh query builder; later chain methods mutate that builder, not the database or another builder.
- The same source column can be selected under multiple aliases.
- Result objects should be fresh across `.all()` calls, even though nested values may remain shared references.

## Notes

- Projection should happen after filtering and sorting so those earlier steps can still work with the original full row.
- Keeping `select()` optional preserves backwards compatibility with part I.
- A flat alias-to-column map is enough for this follow-up. Nested select objects can stay out of scope.

## Techniques

- Object-oriented programming
- Query pipelines
- Result shaping
- Aliasing selected fields
- Preserving backwards-compatible APIs

## Resources

- [Drizzle ORM select docs](https://orm.drizzle.team/docs/select)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A refactor applies `select({ displayName: users.name })` immediately and discards every other field before executing `where(eq(users.role, 'admin'))`. Why does this break a query even though `role` is not wanted in the final result?

Your notes (optional)
