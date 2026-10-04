---
title: "Drizzle Query Builder"
aliases:
  - "drizzleQueryBuilder"
  - "Drizzle Query Builder"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Drizzle Query Builder

> [!info] Problem
> Implement a simplified in-memory query builder inspired by Drizzle ORM, with table helpers, filters, sorting, and pagination

## Problem

## Drizzle Query Builder

This question is inspired by [Drizzle ORM](https://orm.drizzle.team/), whose query builder starts from patterns like [`db.select().from(table)`](https://orm.drizzle.team/docs/select) and composes helpers such as `eq()`, `and()`, `or()`, `asc()`, and `desc()`.

In this question, implement a simplified in-memory version of that style of API. The `table()` helper here is a database-agnostic stand-in for Drizzle's schema helpers, so it does not need to match Drizzle's exact schema API.

Like Drizzle ORM's [dynamic query building](https://orm.drizzle.team/docs/dynamic-query-building), each call to `db.select()` should create a new query builder. After that, `.from()`, `.where()`, `.orderBy()`, `.limit()`, and `.offset()` should mutate that query builder and return it for further chaining. This builder mutability is separate from row/result safety: stored rows should still be isolated from external mutation, and `.all()` should still return fresh result objects.

Implement these helpers:

- `integer(name)`
- `text(name)`
- `table(name, columns)`
- `drizzle(data)`
- `eq(left, right)`
- `and(...conditions)`
- `or(...conditions)`
- `asc(column)`
- `desc(column)`

`drizzle(data)` should return a `db` object with a `select()` method that creates query builders.

## Examples

```javascript
const users = table('users', {
  id: integer('id'),
  name: text('name'),
  age: integer('age'),
  role: text('role'),
});

const db = drizzle({
  users: [
    { id: 1, name: 'Ada', age: 31, role: 'admin' },
    { id: 2, name: 'Grace', age: 28, role: 'editor' },
    { id: 3, name: 'Linus', age: 35, role: 'admin' },
  ],
});

db.select().from(users).all();
// [
//   { id: 1, name: 'Ada', age: 31, role: 'admin' },
//   { id: 2, name: 'Grace', age: 28, role: 'editor' },
//   { id: 3, name: 'Linus', age: 35, role: 'admin' },
// ]

db.select()
  .from(users)
  .where(and(eq(users.role, 'admin'), or(eq(users.age, 31), eq(users.age, 35))))
  .orderBy(desc(users.age))
  .limit(1)
  .all();
// [{ id: 3, name: 'Linus', age: 35, role: 'admin' }]
```

## API

Even for a basic version, there are quite a few APIs to implement. It is recommended to consult [Drizzle's documentation](https://orm.drizzle.team/docs/overview) for more details.

### integer(name) and text(name)

Return simple column definitions that `table()` can use. They only need enough information for this question's query builder and do not need runtime type validation.

### table(name, columns)

Creates a table object whose properties are column objects.

For example:

```javascript
const users = table('users', {
  id: integer('id'),
  name: text('name'),
});
```

The returned `users.id` and `users.name` values are later passed to helpers such as `eq(users.id, 1)` and `asc(users.name)`.

### drizzle(data)

Creates an in-memory database whose keys are table names and whose values are arrays of rows.

The returned `db` object must support:

### db.select()

Returns a query builder with the following chainable methods:

- `.from(table)`
- `.where(condition)`
- `.orderBy(...orderings)`
- `.limit(count)`
- `.offset(count)`
- `.all()`

### eq(left, right)

Returns a condition that checks strict equality. In this part, `left` will always be a column and `right` will always be a literal value.

### and(...conditions) and or(...conditions)

Combine conditions with boolean AND / OR semantics.

### asc(column) and desc(column)

Create sort instructions for `orderBy()`.

## Notes

- `db.select().from(table).all()` should return shallow-cloned rows in insertion order by default.
- Clone the initial input rows when creating the database so later external mutation does not change stored data.
- Sorting happens before `offset()` and `limit()`.
- `orderBy()` only needs to support columns from the `from()` table in this part.
- You do not need inserts, updates, deletes, grouped queries, aggregates, aliases, joins, or SQL string generation.

## Follow-up

Implement [Drizzle Query Builder II](/questions/javascript/drizzle-query-builder-ii) to add `select({ ... })` projections and aliases.

## Hints

### Hint 1 : What identity does a column reference need?

### Hint 2 : When should the query actually run?

### Hint 3 : Can conditions share one input shape?

## 🤔 Thought Process

- **Immediate Recognition:** Building an in-memory SQL-like query builder replicating modern ORM ergonomics (Drizzle ORM).
- **Core Architecture:**
  - Table schemas: Helper functions (`table`, `integer`, `text`) defining fields and holding rows.
  - Query Builder lifecycle: Calling `db.select()` creates a fresh builder instance; method calls (`.from()`, `.where()`, `.orderBy()`, `.limit()`, `.offset()`) mutate the builder state and return `this` for chaining.
  - Terminal execution: `.all()` executes the query against stored rows, applies filter predicates, sorts, paginates, and returns fresh cloned objects.
  - Predicate builders: `eq(col, val)`, `and(...predicates)`, `or(...predicates)` returning callable functions `(row) => boolean`.
  - Order helpers: `asc(col)`, `desc(col)` returning sort descriptor functions or objects.
- **Evaluation Order in `.all()`:**
  1. Source rows from `.from(table)` (defensive shallow copy).
  2. Filter rows using `.where(condition)`.
  3. Sort rows with comparator from `.orderBy(...)`.
  4. Apply pagination: `.slice(offset, offset + limit)`.
  5. Return detached clones so caller cannot mutate DB state.

---

## 🧠 Mental Model

Think of a **Query Execution Abstract Syntax Tree (AST)**:
```
db.select()                         <-- Creates query context
  .from(users)                      <-- Sets scan source
  .where(and(eq(users.age, 25)))    <-- Attaches boolean filter tree
  .orderBy(asc(users.name))         <-- Sets comparator chain
  .limit(10)                        <-- Sets window bounds
  .all()                            <-- Evaluates pipeline and emits rows
```

---

## 🔑 Key Concepts

- [[Method Chaining]]
- [[Closure]]
- [[Type Checking]]
- Functional Predicate Composition (`and`, `or`, `eq`)
- Separation of Query Construction vs Execution
- In-memory pagination and stable sorting

---

## ⚠️ Edge Cases / Traps

- **Row Mutation Leakage:** If rows returned by `.all()` are modified by the caller, the underlying table rows must NOT be mutated. Always map results to fresh objects `{ ...row }`.
- **Default Offset / Limit:** If `.limit()` is not called, return all matching rows. If `.offset()` is not called, start at index `0`. Calling `.offset(0)` or `.limit(0)` should be handled accurately (limit 0 returns `[]`).
- **Compound Predicates:** `and()` with no arguments should evaluate to `true`; `or()` with no arguments should evaluate to `false`.
- **Multiple OrderBy Criteria:** If `orderBy` accepts multiple columns (`asc(colA), desc(colB)`), tie-breakers must cascade until non-zero difference or exhausted.

---

## ⭐ Interview Takeaway

- Distinguish builder mutability from data immutability:
  - The query builder instance itself is mutable during assembly (chaining returns `this`).
  - Stored data rows must remain immutable and protected from external side effects.
- Implement conditions as pure predicate closures: `eq = (col, val) => (row) => row[col.name] === val`.
- `.all()` is the terminal sink that runs the array pipeline: `rows.filter(...).sort(...).slice(...)`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does Drizzle ORM use separate helper functions (`eq`, `and`) instead of string literals?" (Type safety, composability, and avoiding SQL injection vulnerabilities).
- "How do you achieve fluent method chaining in JavaScript?" (Methods record parameters on internal state and return `this`).

### Follow-up Questions
- "How would you implement column projection (`select({ name: users.name })`)?" (Answered in Drizzle Query Builder II).
- "How would you handle table joins?" (Answered in Drizzle Query Builder III).

### Conceptual Questions
- "How does this in-memory builder compare to an active database ORM that compiles to raw SQL?" (Instead of emitting an SQL string for a DBMS engine, this evaluator directly executes array transformations on JavaScript objects).

---

## 🔄 Variations

- **Drizzle Query Builder II:** Adding column projections and aliases.
- **Drizzle Query Builder III:** Adding `innerJoin` and `leftJoin`.
- **Mini ORM / Knex Clone:** SQL string compilation rather than in-memory execution.

---

## 📝 Revision Notes

- Core skeleton:
```javascript
export function table(name, columns) {
  const rows = [];
  return {
    name,
    columns,
    insert(row) { rows.push({ ...row }); },
    _rows: rows,
  };
}

export function eq(col, val) {
  return (row) => row[col.name] === val;
}

export function and(...conditions) {
  return (row) => conditions.every(fn => fn(row));
}

export function or(...conditions) {
  return (row) => conditions.some(fn => fn(row));
}
```

---

## Official Solution

## Drizzle Query Builder ( Official solution )

Premium
Languages
The main design choice is choosing a representation that is easy to execute in memory while still feeling like a Drizzle-style query builder.

## Solution

The recommended structure is:

1. Use `table()` to build column objects that remember both their table name and column name.
2. Store each table's rows in a `Map`.
3. Let `eq()`, `and()`, and `or()` return predicate functions over a row context.
4. Let `select().from(table)` build a query object that runs filter, sort, offset, and limit in that order.

For part I, the row context is simple because there is only one table involved:

```javascript
{
  users: { id: 1, name: 'Ada', age: 31, role: 'admin' },
}
```

That means `eq(users.role, 'admin')` can be evaluated by reading the `role` field from the `users` row in the current context. `and()` and `or()` then just compose those predicate functions.

`orderBy()` can store an array of sort instructions and compare rows by each column in sequence until one differs. This keeps the code interview-friendly without introducing a full SQL AST.

### Query pipeline walkthrough

The query builder should collect configuration as methods are chained, then execute the whole pipeline only when `.all()` is called. For example:

```javascript
db.select()
  .from(users)
  .where(eq(users.role, 'admin'))
  .orderBy(desc(users.age))
  .offset(1)
  .limit(1)
  .all();
```

That should run in this order:

1. Build one row context per stored `users` row.
2. Filter contexts with the `where` predicate.
3. Sort the remaining contexts by `users.age`.
4. Apply `offset`, then `limit`.
5. Return shallow-cloned row objects.

The useful property is that the builder methods describe the query, while `.all()` executes it from the original stored rows. That separation makes chained calls predictable and avoids partially mutating the result set after each method call.

The row count through that pipeline might look like this:

| Phase | Example result |
| --- | --- |
| Build contexts | all `users` rows |
| `where(eq(users.role, 'admin'))` | only admin rows |
| `orderBy(desc(users.age))` | same rows, sorted oldest first |
| `offset(1)` | drop the first sorted row |
| `limit(1)` | return one shallow-cloned row |

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
 * @param {...Condition} conditions
 * @returns {Condition}
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
 * @param {Column} column
 * @returns {Ordering}
 */
export function desc(column) {
  return {
    column,
    direction: 'desc',
  };
}

class SelectQuery {
  constructor(data) {
    this._data = data;
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
    // A row context keeps lookups uniform for filters and orderings.
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

    return contexts.map((context) => cloneRow(context[tableName]));
  }
}

class DrizzleDatabase {
  constructor(data) {
    this._data = new Map();

    Object.entries(data).forEach(([tableName, rows]) => {
      this._data.set(tableName, cloneRows(rows));
    });
  }

  select() {
    return new SelectQuery(this._data);
  }
}

/**
 * @param {DatabaseData} data
 * @returns {DrizzleDatabase}
 */
export default function drizzle(data) {
  return new DrizzleDatabase(data);
}
```

## Edge cases

- `select()` creates a fresh mutable builder, so two builders from the same database should not share filter, sort, offset, or limit state.
- Stored rows are shallow-cloned when the database is created, and query results are shallow-cloned again before returning.
- Multi-key ordering should continue to the next ordering only when the previous comparison ties.
- `offset()` and `limit()` apply after filtering and sorting, not while rows are being collected.

## Notes

- Returning predicate functions from the helpers keeps the builder small and avoids building a full AST.
- Cloning at database creation time isolates stored rows from later external mutation.
- Shallow cloning query results is enough for this question, so nested objects may still share references.
- Evaluating helpers against a row context instead of raw rows keeps `eq()`, `and()`, `or()`, and `orderBy()` aligned with the follow-up path toward joins and projections.

## Techniques

- Object-oriented programming
- Fluent query builders
- Predicate composition
- Shallow cloning
- Multi-key sorting

## Resources

- [Drizzle ORM select docs](https://orm.drizzle.team/docs/select)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A query sorts scores descending, skips one row, and takes one row. A faulty executor instead paginates the insertion-order rows before sorting. Which score sequence distinguishes the two executors?
