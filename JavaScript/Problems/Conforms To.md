---
title: Implement conformsTo(object, source) for predicate-based validation
aliases:
  - Conforms To
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[Object]]"
  - "[[Predicate Functions]]"
solved: true
solvedDate: 2026-09-05
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Check whether an object conforms to a set of predicate rules defined by a source object.

## Problem

Implement a function `conformsTo(object, source)` that checks whether `object` conforms to `source` by invoking the predicate properties of `source` with the corresponding property values of `object`.

```js
conformsTo(object, source);
```

**Arguments:**
- `object (Object)`: The object to inspect.
- `source (Object)`: The object of property predicates to conform to.

**Returns:** `(boolean)` Returns `true` if `object` conforms; otherwise, returns `false`.

**Examples**

```js
conformsTo({ a: 1, b: 2 }, { b: (n) => n > 1 });
// => true

conformsTo({ a: 1, b: 2 }, { b: (n) => n > 2 });
// => false
```

The function should return `false` when `object` is empty.

```js
conformsTo({}, { b: (n) => n > 1 }); // => false
```

**Constraints:**
- `object`: JavaScript object.
- `source`: JavaScript object. Its own properties must be predicate functions.

## Pattern

- [[Array Traversal]]

## 🤔 Thought Process

* `source` contains the **rules/predicates** we need to check.
* For every property in `source`:

  1. Get the corresponding value from `object`.
  2. Pass that value to the predicate.
  3. If **any predicate returns false**, the whole function returns false.
* If all predicates return true → return true.
* Special requirement: an empty `object` must return false.

## 💻 Final Solution

```js
export default function conformsTo(object, source) {
  if (object === null || typeof object !== "object") {
    return false;
  }
  if (Object.keys(object).length === 0) {
    return false;
  }
  return Object.entries(source).every(([key, predicate]) => {
    return predicate(object[key]);
  });
}
```

## 🤔 Why This Works

Example:

```js
conformsTo(
  { a: 1, b: 2 },
  { b: n => n > 1 }
)
```

`Object.entries(source)` gives:

```js
[['b', predicate]]
```

Then:

```js
predicate(object[key])
```

becomes:

```js
(n => n > 1)(object.b)
```

which is:

```js
2 > 1
```

→ `true`.

With:

```js
{ b: n => n > 2 }
```

you get:

```js
2 > 2
```

→ `false`.

`every()` is appropriate because **all predicates must pass**.

## 🐞 Bugs I Made

The solution is **correct for the stated constraints**.

One thing worth noticing:

```js
if (object === null || typeof object !== "object")
```

is necessary because:

```js
typeof null === "object"
```

The explicit `null` check handles that correctly.

Also, because the constraint guarantees that `source`'s own properties are functions, you don't need to validate `predicate`.

## Production Considerations

- Lodash ships `_.conforms(source)` (creates a function) and `_.conformsTo(object, source)` (validates).
- This implementation does **not mutate** either object.
- For complex validation schemas, libraries like Joi, Yup, or Zod provide more feature-rich alternatives.

## ⭐ Revision Notes

### 🔑 Key Facts

* `source` = **rules**
* `object` = **data being tested**
* `Object.entries(source)` → gives `[key, predicate]`
* `object[key]` → gets the value to test
* `predicate(object[key])` → performs the test
* `every()` → all rules must pass
* Empty `object` → explicitly returns `false`
* The implementation does **not mutate** either object.

### 🧠 Mental Model

Think of `source` as a set of tests:

```text
source
  ↓
b: n => n > 1
  ↓
look at object.b
  ↓
2
  ↓
predicate(2)
  ↓
true
```

For multiple predicates:

```text
source
 ├── a → predicate(object.a) → true
 ├── b → predicate(object.b) → true
 └── c → predicate(object.c) → false
                         ↓
                      every()
                         ↓
                       false
```

So:

> **`conformsTo` = run every predicate from `source` against the corresponding value in `object`.**

### Common Interview Questions

- How is `conformsTo` different from schema validation? → It only validates properties defined in `source`; extra properties on `object` are ignored. Schema validators like Joi enforce required fields, types, etc.
- What if a predicate throws? → The exception propagates; no try/catch in this implementation.
- Can `source` have non-function values? → Constraint guarantees functions, but the implementation would call them as functions which would throw on non-callables.

### Interview Takeaways

The clean approach for this problem:

```js
return Object.entries(source).every(([key, predicate]) => {
  return predicate(object[key]);
});
```

The main pattern to remember is:

**`Object.entries()` + `every()` + dynamic property access.**

### Related

- [[Object Traversal]]
- [[Get]]
- [[Type Checking]]