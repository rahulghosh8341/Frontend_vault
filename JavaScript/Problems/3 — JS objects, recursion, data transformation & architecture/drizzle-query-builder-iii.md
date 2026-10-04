---
title: "Drizzle Query Builder III"
aliases:
  - "drizzleQueryBuilderIII"
  - "Drizzle Query Builder III"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Drizzle Query Builder III

> [!info] Problem
> Implement a simplified in-memory query builder inspired by Drizzle ORM, with support for `innerJoin()` and `leftJoin()`

## Problem

## Drizzle Query Builder III

This is a follow-up to [Drizzle Query Builder II](/questions/javascript/drizzle-query-builder-ii).

This part is inspired by [Drizzle ORM's join API](https://orm.drizzle.team/docs/joins).

In this question, keep the same table helpers, conditions, sorting, pagination, and `select({ ... })` support, but now add joins:

- `.innerJoin(table, on)`
- `.leftJoin(table, on)`

Like Drizzle, the join condition can compare columns directly, such as `eq(users.id, posts.authorId)`.

Like Drizzle ORM's [dynamic query building](https://orm.drizzle.team/docs/dynamic-query-building), each call to `db.select()` or `db.select(selection)` should create a new query builder. After that, `.from()`, `.innerJoin()`, `.leftJoin()`, `.where()`, `.orderBy()`, `.limit()`, and `.offset()` should mutate that query builder and return it for further chaining. This mutability model does not change the result guarantees: stored rows should still be isolated from external mutation, and `.all()` should still return fresh result objects.

## Examples

```javascript
const users = table('users', {
  id: integer('id'),
  name: text('name'),
});

const posts = table('posts', {
  id: integer('id'),
  authorId: integer('authorId'),
  title: text('title'),
});

const db = drizzle({
  users: [
    { id: 1, name: 'Ada' },
    { id: 2, name: 'Grace' },
  ],
  posts: [
    { id: 10, authorId: 1, title: 'Intro' },
    { id: 11, authorId: 1, title: 'Advanced' },
  ],
});

db.select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .orderBy(asc(users.id), asc(posts.id))
  .all();
// [
//   {
//     users: { id: 1, name: 'Ada' },
//     posts: { id: 10, authorId: 1, title: 'Intro' },
//   },
//   {
//     users: { id: 1, name: 'Ada' },
//     posts: { id: 11, authorId: 1, title: 'Advanced' },
//   },
//   {
//     users: { id: 2, name: 'Grace' },
//     posts: null,
//   },
// ]
```

## Join behavior

### innerJoin(table, on)

Keeps only row combinations where the join condition matches.

### leftJoin(table, on)

Keeps every row from the left side. If no row matches in the joined table, that table's value should be `null` in the result context.

## Result shape

### db.select().from(...).join(...).all()

When `select()` is called without a selection object and the query includes joins, return one object per joined row context keyed by table name.

### db.select(selection).from(...).join(...).all()

When a selection object is provided, keep returning a flat object whose keys come from the selection aliases.

For example:

```javascript
db.select({
  userId: users.id,
  userName: users.name,
  postTitle: posts.title,
})
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .all();
// [
//   { userId: 1, userName: 'Ada', postTitle: 'Intro' },
//   { userId: 1, userName: 'Ada', postTitle: 'Advanced' },
//   { userId: 2, userName: 'Grace', postTitle: null },
// ]
```

## Notes

- `eq(left, right)` must now support both column-to-literal and column-to-column comparisons.
- Queries may include multiple joins, and each join should build on the row contexts produced by the previous step.
- When a selected column comes from a missing row on a `leftJoin()`, return `null` for that field.
- When ordering by a nullable joined column, place `null` after non-null values in ascending order and before them in descending order.
- You do not need right joins, full joins, grouping, aggregates, table aliases, nested select objects, or SQL generation.

## Hints

### Hint 1 : How does one join transform a row context?

### Hint 2 : Why must joins be applied in call order?

### Hint 3 : How can `eq()` compare two columns?

## 🤔 Thought Process

- **Immediate Recognition:** Extending the in-memory Drizzle query builder with relational join capabilities (`innerJoin` and `leftJoin`).
- **Core Requirements:**
  - Support `.innerJoin(table, onCondition)` and `.leftJoin(table, onCondition)`.
  - Joined row structure:
    - In Drizzle, joined rows group columns by table name: `{ users: { id: 1, name: '...' }, posts: { id: 10, title: '...' } }`.
    - If a projection `select({ ... })` is specified, resolve aliases from their respective table namespaces.
  - Join mechanics:
    - `innerJoin`: Only keep combined rows where `onCondition(leftRow, rightRow)` evaluates to `true`.
    - `leftJoin`: Keep all rows from left table; if no right row satisfies `onCondition`, combine with `null` for the right table (`{ posts: null }`).
- **Execution Pipeline:**
  1. Base scan of `from(table)` rows, normalized into `{ [tableName]: row }`.
  2. For each join clause:
     - For each row in current accumulated result set:
       - Match against all rows in target join table using `on(accumulatedRow, rightRow)`.
       - For `innerJoin`: emit matched combinations.
       - For `leftJoin`: emit matched combinations, or emit single combination with right table set to `null` if no match.
  3. Filter with `where`, sort with `orderBy`, slice with `offset`/`limit`.
  4. Project final output.

---

## 🧠 Mental Model

Think of **Nested Loop Joins with Table Namespaces**:
```
Left Set (Users):   [ { id: 1, name: 'Alice' } ]
Right Set (Posts):  [ { id: 101, userId: 1, title: 'Post 1' } ]

Inner Join Match:
{
  users: { id: 1, name: 'Alice' },
  posts: { id: 101, userId: 1, title: 'Post 1' }
}
```

---

## 🔑 Key Concepts

- [[Method Chaining]]
- [[Array Traversal]]
- [[Closure]]
- Relational Algebra (Nested Loop Joins)
- Cartesian Product & Join Predicate Evaluation
- Nullable record handling in `LEFT JOIN`

---

## ⚠️ Edge Cases / Traps

- **`leftJoin` with No Matches:** If a user has zero posts, `leftJoin` must still output the user, setting `posts: null` in the composite row.
- **Multiple Matches (One-to-Many):** If a user has 3 posts, the user row must duplicate 3 times in the output (one for each post match).
- **Column Name Collision:** Tables frequently share column names like `id`, `created_at`. Grouping by table name (`{ users: { id }, posts: { id } }`) avoids clobbering keys before projection.
- **Projection with Joins:** Projections referencing joined columns (`posts.title`) must read from the joined table namespace.

---

## ⭐ Interview Takeaway

- Model joined rows as namespaced objects: `{ [table.name]: row }`.
- Nested Loop Join is the simplest and clearest implementation for in-memory JS interview questions:
  - For each `leftRow`: find matching `rightRows`.
  - If `innerJoin` and no match: skip.
  - If `leftJoin` and no match: produce `{ ...leftRow, [rightTable.name]: null }`.
- Run joins first, then filter, sort, paginate, and project.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How does an INNER JOIN differ from a LEFT JOIN in SQL and in-memory execution?" (INNER JOIN drops unmatched left rows; LEFT JOIN keeps left rows and fills right table columns with `null`).
- "Why are joined rows structured under table names in Drizzle instead of being flattened into one object?" (To prevent property collisions when multiple tables have identical column names like `id`).

### Follow-up Questions
- "How would you optimize nested loop joins for large in-memory arrays?" (Use Hash Join: index the right table by the join key in a `Map` for $O(1)$ lookups instead of $O(N \cdot M)$ scans).
- "How would you implement `rightJoin` and `fullJoin`?"

### Conceptual Questions
- "What is the time complexity of a nested-loop join vs a hash join?" (Nested loop: $O(N \cdot M)$; Hash join: $O(N + M)$).

---

## 🔄 Variations

- **Drizzle Query Builder I & II:** Foundation and projection stages.
- **Data Merging:** Grouping and aggregating related records.
- **Hash Join Utility:** Implementing $O(N + M)$ relational joining in JavaScript.

---

## 📝 Revision Notes

- Join execution pattern:
```javascript
function executeJoins(baseRows, baseTable, joins) {
  let combined = baseRows.map(row => ({ [baseTable.name]: row }));

  for (const join of joins) {
    const next = [];
    const { table: joinTable, on, type } = join;

    for (const left of combined) {
      let matched = false;
      for (const right of joinTable._rows) {
        if (on(left, right)) {
          matched = true;
          next.push({ ...left, [joinTable.name]: right });
        }
      }
      if (!matched && type === 'left') {
        next.push({ ...left, [joinTable.name]: null });
      }
    }
    combined = next;
  }
  return combined;
}
```

---

## Official Solution

## Drizzle Query Builder III ( Official solution )

Premium
Languages
This follow-up adds join expansion on top of the part II query pipeline, mirroring the shape of Drizzle ORM's join-oriented queries without trying to reproduce the full production API.

## Solution

Frame it as row-context expansion:

1. Start with one row context per source-table row.
2. For each join, expand every current context into zero, one, or many new contexts.
3. Run `where`, `orderBy`, `offset`, and `limit` on the expanded contexts.
4. Shape the final result:
  - nested table objects when `select()` was called without a selection object
  - flat alias objects when `select({ ... })` was provided

For example, after:

```javascript
.from(users).leftJoin(posts, eq(users.id, posts.authorId))
```

the intermediate contexts could look like:

```javascript
[
  {
    users: { id: 1, name: 'Ada' },
    posts: { id: 10, authorId: 1, title: 'Intro' },
  },
  {
    users: { id: 2, name: 'Grace' },
    posts: null,
  },
];
```

For an `innerJoin()`, a context that has no matching joined row disappears. For a `leftJoin()`, that same context survives with the joined table set to `null`. This is why the column-reading helper needs to treat a missing left-joined row as `null` instead of `undefined` when projecting selected columns.

Join order also matters. Each join consumes the contexts produced by the previous join, so multiple joins form a progressive expansion:

```text
base users -> users + posts -> users + posts + comments
```

After that expansion is complete, `where()` and `orderBy()` should run over the joined contexts. This lets predicates and orderings reference columns from any table that has already joined the query.

Once every intermediate row is represented as a `{tableName: rowOrNull}` context, joins, filters, and projections can all share the same "read a column from the current context" helper.

Trace for a left join where user `2` has no post:

| Current context | Matching post rows | Join type | Output context |
| --- | --- | --- | --- |
| `{ users: { id: 1 } }` | `[{ id: 10, authorId: 1 }]` | `left` | `{ users: { id: 1 }, posts: { id: 10, authorId: 1 } }` |
| `{ users: { id: 2 } }` | `[]` | `left` | `{ users: { id: 2 }, posts: null }` |

The same second row would disappear for `innerJoin()`. For `select({ postTitle: posts.title })`, reading `posts.title` from that `null` joined row returns `null`, matching Drizzle's left-join projection shape.

```jsx
/**
 * @typedef {Record<string, unknown>} Row
 * @typedef {Record<string, Array<Row>>} DatabaseData
 * @typedef {Record<string, Row | null>} RowContext
 * @typedef {'integer' | 'text'} ColumnKind
 *
 * @typedef {object} ColumnBuilder
 * @property {ColumnKind} kind
 * @property {string} name
 *
 * @typedef {object} Column
 * @property {ColumnKind} kind
 * @property {string} name
 * @property {string} tableName
 *
 * @typedef {{ _: { name: string } }} AnyTable
 * @typedef {Record<string, Column>} Selection
 * @typedef {(context: RowContext) => boolean} Condition
 *
 * @typedef {object} Ordering
 * @property {Column} column
 * @property {'asc' | 'desc'} direction
 *
 * @typedef {object} SelectQuery
 * @property {(table: AnyTable) => SelectQuery} from
 * @property {(table: AnyTable, on: Condition) => SelectQuery} innerJoin
 * @property {(table: AnyTable, on: Condition) => SelectQuery} leftJoin
 * @property {(condition: Condition) => SelectQuery} where
 * @property {(...orderings: Array<Ordering>) => SelectQuery} orderBy
 * @property {(count: number) => SelectQuery} limit
 * @property {(count: number) => SelectQuery} offset
 * @property {() => Array<Row>} all
 *
 * @typedef {object} DrizzleDatabase
 * @property {(selection?: Selection) => SelectQuery} select
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
 * @returns {AnyTable}
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

function cloneJoinedContext(context) {
  const result = {};

  Object.entries(context).forEach(([tableName, row]) => {
    result[tableName] = row == null ? null : cloneRow(row);
  });

  return result;
}

function resolveColumnValue(column, context) {
  const row = context[column.tableName];

  if (row === null) {
    return null;
  }

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
    // Selection aliases are projected by reading from the assembled join context.
    result[alias] = resolveColumnValue(column, context);
  });

  return result;
}

function compareValues(left, right) {
  if (left === right) {
    return 0;
  }

  if (left == null) {
    return 1;
  }

  if (right == null) {
    return -1;
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
  constructor(data, selection) {
    this._data = data;
    this._selection = selection;
    this._fromTable = null;
    this._condition = null;
    this._orderings = [];
    this._joins = [];
    this._limit = null;
    this._offset = 0;
  }

  from(table) {
    this._fromTable = table;
    return this;
  }

  innerJoin(table, on) {
    this._joins.push({
      type: 'inner',
      table,
      on,
    });
    return this;
  }

  leftJoin(table, on) {
    this._joins.push({
      type: 'left',
      table,
      on,
    });
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

  _applyJoin(contexts, join) {
    const joinTableName = join.table._.name;
    const joinRows = this._data.get(joinTableName) ?? [];
    const nextContexts = [];

    contexts.forEach((context) => {
      // Expand each context once per matching joined row.
      const matches = joinRows.filter((row) =>
        join.on({
          ...context,
          [joinTableName]: row,
        }),
      );

      if (matches.length === 0) {
        if (join.type === 'left') {
          // Left joins keep the base row and expose the missing side as null.
          nextContexts.push({
            ...context,
            [joinTableName]: null,
          });
        }

        return;
      }

      matches.forEach((row) => {
        nextContexts.push({
          ...context,
          [joinTableName]: row,
        });
      });
    });

    return nextContexts;
  }

  all() {
    const table = this._fromTable;
    const tableName = table._.name;
    // Start with one context per base-table row, then layer joins onto it.
    let contexts = (this._data.get(tableName) ?? []).map((row) => ({
      [tableName]: row,
    }));

    this._joins.forEach((join) => {
      contexts = this._applyJoin(contexts, join);
    });

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

    if (this._selection != null) {
      return contexts.map((context) =>
        projectSelection(this._selection, context),
      );
    }

    if (this._joins.length === 0) {
      return contexts.map((context) => cloneRow(context[tableName]));
    }

    return contexts.map(cloneJoinedContext);
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
 * @returns {DrizzleDatabase}
 */
export default function drizzle(data) {
  return new DrizzleDatabase(data);
}
```

## Common pitfalls

- Treating joins as context expansion keeps the join logic much simpler than trying to materialize partial objects early.
- `leftJoin()` is just the join-expansion logic plus a fallback `null` row when there are no matches.
- Doing projection at the end lets the same join pipeline support both nested default results and flat aliased selects.
- `select()` creates a fresh builder, while builder methods mutate only that builder and return it for chaining.
- Stored rows and returned rows are cloned so callers cannot mutate the database through query results.

## Techniques

- Object-oriented programming
- Query pipelines
- Join expansion
- Result shaping from row contexts
- Column-to-column predicate evaluation

## Resources

- [Drizzle ORM joins docs](https://orm.drizzle.team/docs/joins)

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A report left-joins users to their posts, then adds `row.users.credits` across all returned rows. Ada has 10 credits and two posts; Grace has 20 credits and no posts. Why does the total become 40 instead of 30?

Your notes (optional)
