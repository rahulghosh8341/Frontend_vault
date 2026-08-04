# Fixed Size Grouping

## Core Idea

Process elements sequentially and collect them into groups of a fixed size.

```
current.push(item)

if current is full
    add to result
    reset current
```

---

## Recognition

Use this pattern when you see:

- Split into groups
- Batch processing
- Pagination
- Rows / Columns
- Fixed window output

---

## Template

```js
const result = [];
let current = [];

for (const item of items) {
  current.push(item);

  if (current.length === size) {
    result.push(current);
    current = [];
  }
}

if (current.length) {
  result.push(current);
}
```

---

## Complexity

| Time | Space |
|------|------|
| O(n) | O(n) |

---

## Common Mistakes

- Forgetting the last partial chunk.
- Not resetting the temporary array.
- Returning references incorrectly.
- Ignoring invalid `size`.

---

## Production Considerations

- Prefer a single-pass solution.
- `Array.slice()` is often simpler but creates more intermediate arrays.

---

## Problems

```dataview
LIST
FROM "JavaScript/Problems"
WHERE contains(file.outlinks, this.file.link)
SORT file.name
```