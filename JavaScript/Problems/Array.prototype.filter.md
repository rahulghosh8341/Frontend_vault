---
aliases:
  - Array.prototype.filter
  - filter Polyfill
  - myFilter
difficulty: Easy
time: 15 min
languages:
  - JavaScript
  - TypeScript
companies:
  - "[[Amazon]]"
  - "[[Apple]]"
  - "[[Pinterest]]"
  - "[[Canva]]"
pattern:
  - "[[Array Traversal]]"
concepts:
  - "[[this]]"
  - "[[call, apply and bind]]"
  - "[[Sparse Arrays]]"
solved: true
solvedDate: 2026-08-07
type: coding
---

> [!info] 
>  **Difficulty:** 🟢 Easy | **Time:** 15 min
> Creates a new array containing only the elements for which the callback returns `true`.

---

## Problem

Implement `Array.prototype.filter` as `Array.prototype.myFilter`.

Requirements:

- Return a **new array**.
- Execute the callback for every existing element.
- Skip sparse array holes.
- Support the optional `thisArg`.
- Do **not** modify the original array.

---

## Companies

> [!tip]
> [[Amazon]] • [[Apple]] • [[Pinterest]] • [[Canva]]

---

## Pattern

⭐⭐ [[Array Traversal]]

---

## 🤔 Thought Process

### Initial Approach

1. Create an empty `result` array.
2. Traverse the original array.
3. Skip sparse (hole) elements.
4. Execute the callback using the correct `thisArg`.
5. If callback returns `true`, push the element into `result`.
6. Return the new array.

### Key Observations

- `filter()` never modifies the original array.
- It always returns a new array.
- Sparse array holes must be ignored.
- Callback receives `(value, index, array)`.
- `this` inside `myFilter` refers to the original array.

---

## 💻 Final Solution

```js
Array.prototype.myFilter = function (callbackFn, thisArg) {
  const result = [];

  for (let i = 0; i < this.length; i++) {
    if (
      Object.hasOwn(this, i) &&
      callbackFn.call(thisArg, this[i], i, this)
    ) {
      result.push(this[i]);
    }
  }

  return result;
};
```

**Time:** O(n)

**Space:** O(n)

---

## 🤔 Why This Works

### Why `Object.hasOwn(this, i)`?

Native `filter()` skips sparse array holes.

```js
const arr = [1, 2, , 4];
```

Index `2` doesn't actually exist.

```js
Object.hasOwn(arr, 2);
// false
```

Without this check, the callback would incorrectly execute for missing elements.

Same as `i in arr`

---
### Why `this`?

When calling

```js
arr.myFilter(...)
```

inside the function

```js
this === arr
```

Therefore

```js
this.length
```

becomes

```js
arr.length
```

and

```js
this[i]
```

becomes

```js
arr[i]
```

---

### Sparse vs Undefined

```js
const a = [1, , 3];
Object.hasOwn(a, 1);
// false
```

```js
const b = [1, undefined, 3];
Object.hasOwn(b, 1);
// true
```

Both return

```js
arr[1];
// undefined
```

but internally they represent different cases.

---

### Why use `.call()`?

`filter()` supports an optional `thisArg`.

```js
callbackFn.call(thisArg, value, index, array);
```

Without `.call()`

```js
callbackFn(value);
```

`this` inside the callback would become `undefined` (strict mode).

---

### `this` vs `thisArg`

Inside `myFilter`

```text
this
↓
Original array
```

Inside callback

```text
this
↓
thisArg (obj if any passed)
```

These are different values.

---

### Why not `bind()`?

```js
const fn = callbackFn.bind(thisArg);
```

Possible, but

- `bind()` creates a new function. then need to keep track of each function
- `call()` immediately invokes it.
- Less allocation inside the loop.

---

## 🐞 Bugs I Made

- Initially forgot that native `filter()` skips sparse array holes.
- Needed `Object.hasOwn()` instead of checking `value !== undefined`.
- Forgot that callback receives three arguments:
  - value
  - index
  - original array
- Forgot to invoke the callback using `.call(thisArg, ...)`.

---

## ⭐ Revision Notes

### Mental Model

```text
arr.myFilter(callback, obj)

        │

this = arr

        │

callback.call(
    obj,
    value,
    index,
    arr
)

        │

Inside callback

this = obj
```

---

### Common Interview Questions

**Why not check**

```js
this[i] !== undefined
```

Because

```js
[1, undefined, 3]
```

contains a valid element.

---

**Why is `Object.hasOwn()` needed?**

To distinguish between

- Missing element (hole)
- Existing element whose value is `undefined`

---

**What is `this` inside `myFilter()`?**

The array that invoked the method.

---

**What is `this` inside the callback?**

The supplied `thisArg`.

If no `thisArg` is provided (strict mode), `this` is `undefined`.

---

### Interview Takeaways

- `filter()` does **not** modify the original array.
- Returns a **new array**.
- Skips sparse array holes.
- Uses `Object.hasOwn()`.
- Invokes callback using `.call(thisArg, value, index, array)`.
- `this` inside `myFilter` is the original array.
- `this` inside callback is the provided `thisArg`.

---

### Related

**Patterns**

- [[Array Traversal]]

**Concepts**

- [[this]]
- [[call, apply and bind]]
- [[Sparse Arrays]]