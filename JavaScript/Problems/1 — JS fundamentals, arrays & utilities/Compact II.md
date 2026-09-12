---
title: Implement compact(value) to recursively remove falsy values from arrays and objects
aliases:
  - Compact II
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[Recursion]]"
concepts:
  - "[[Falsy Values]]"
  - "[[Deep Traversal]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-05
type: coding
---
	
> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Recursively remove all falsy values from nested arrays and objects.

## Problem

Implement a function `compact(value)` that returns a new object with all falsy values removed, including falsy values that are deeply nested. You can assume `value` only contains JSON-serializable values (`null`, `boolean`, `number`, `string`, `Array`, `Object`) and will not contain any other objects like `Date`, `RegExp`, `Map`, or `Set`.

The values `false`, `null`, `0`, `''`, `undefined`, and `NaN` are falsy (you should know this by heart!).

**Arguments:**
- `value (Array|Object)`: The array/object to compact.

**Returns:**
- `(Array|Object)`: Returns the new compact array/object.

**Examples**

```js
compact([0, 1, false, 2, '', 3, null]); // => [1, 2, 3]
compact({ foo: true, bar: null }); // => { foo: true }
```

## Pattern

- [[Recursion]]

## 🤔 Thought Process

* `compact()` receives one value at a time.
* If the value is **falsy**, return `undefined` → parent knows to remove it.
* If it's an **array**, recursively compact each item and build a new array.
* If it's an **object**, recursively compact each property and build a new object.
* Otherwise, return the value unchanged.

## 💻 Final Solution

```js
export default function compact(value) {
  // Handle falsy values (including NaN)
  if (!value || (typeof value === 'number' && isNaN(value))) {
    return undefined;
  }

  // Handle arrays
  if (Array.isArray(value)) {
    const result = [];
    for (const item of value) {
      const compacted = compact(item);
      if (compacted !== undefined) {
        result.push(compacted);
      }
    }
    return result;
  }

  // Handle objects
  if (typeof value === 'object') {
    const result = {};
    for (const key of Object.keys(value)) {
      const compacted = compact(value[key]);
      if (compacted !== undefined) {
        result[key] = compacted;
      }
    }
    return result;
  }

  // Return primitives as-is
  return value;
}
```

## 🤔 Why This Works

The important part is that **recursive calls return their processed result to the parent**.

```js
const child = compact(value[key]);
```

Then the parent decides whether to keep it:

```js
if (child !== undefined) {
  resObj[key] = child;
}
```

For nested data:

```text
parent
  ↓
compact(child)
  ↓
cleaned child
  ↓
return to parent
  ↓
parent adds cleaned child
```

Without the final `return result` statements, the parent receives `undefined` and loses the entire nested structure.

## 🐞 Bugs I Made

In the attempted version:

```js
let resObj = [];
```

should be:

```js
let resObj = {};
```

because you're constructing an object.

You also need to:

```js
return resArr;
return resObj;
return value;
```

at the appropriate branches.

And don't blindly:

```js
resArr.push(child);
```

because a falsy child returns `undefined`, which would leave `undefined` in the result. Check before adding.

Special case: `NaN` is falsy but `!NaN` is `false`, so it needs explicit handling.

## Production Considerations

- This implementation handles all JSON-serializable types as specified.
- For non-recursive alternatives, a stack-based traversal could be used, but recursion is more natural for tree structures.
- The function creates new arrays/objects rather than modifying the originals.
- `Object.keys()` only traverses own enumerable properties, which is correct for plain objects.

## ⭐ Revision Notes

### 🔑 Key Facts

There are **three meanings of `return`** here:

```js
return undefined;    // remove this falsy value
return resultArr;    // return cleaned array to parent
return resultObj;    // return cleaned object to parent
return value;        // keep primitive value
```

The final returns are what allow the cleaned nested result to **travel back up the recursion**.

Special handling for `NaN` because `!NaN` is `false`.

### 🧠 Mental Model

Think of recursion as:

```text
GO DOWN
parent
  ↓
child
  ↓
nested child
  ↓
...

THEN COME BACK UP
...
nested child → cleaned result
  ↓
child → cleaned result
  ↓
parent → final result
```

**Recursion goes down; `return` brings the result back up.**

### Common Interview Questions

- Why check `child !== undefined` before adding? → Because falsy children return `undefined`, and we don't want `undefined` values in our result arrays/objects.
- How is this different from shallow `compact`? → Shallow only checks the top level; this one recurses into nested structures.
- What happens with `NaN`? → Special case: `!NaN` is `false`, so it needs explicit `isNaN()` check.

### Interview Takeaways

For recursive data-processing problems, remember:

> **Process child → recursively call → receive returned result → add it to the new parent.**

That's the fundamental pattern behind this `compact` problem.

### Related

- [[Recursion]]
- [[Deep Traversal]]
- [[Falsy Values]]
- [[Compact]]