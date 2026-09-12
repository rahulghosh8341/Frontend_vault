---
title: Implement richTextToHTML(text, ranges) for canonical HTML rendering
aliases:
  - Rich Text to HTML
difficulty: Hard
time: 45 min
languages:
  - JavaScript
companies:
  - "[[Meta]]"
pattern:
  - "[[Object Traversal]]"
concepts:
  - "[[HTML]]"
  - "[[Range]]"
  - "[[Tag Nesting]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-08
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 45 min
> Convert rich text ranges to valid HTML with proper tag nesting and deduplication.

## Problem

Implement a function `richTextToHTML(text, ranges)` to convert rich text representation into canonical HTML string.

```js
/**
 * @typedef {{
 *   start: number,
 *   end: number,
 *   tag: string,
 * }} RichTextRange
 */

/**
 * @param {string} text
 * @param {Array<RichTextRange>} ranges
 * @returns {string}
 */
```

**Examples**

```js
richTextToHTML('hello', [{ start: 1, end: 4, tag: 'b' }]);
// 'h<b>ell</b>o'

richTextToHTML('abcdef', [
  { start: 1, end: 4, tag: 'b' },
  { start: 2, end: 5, tag: 'i' },
]);
// 'a<b>b<i>cd</i></b><i>e</i>f'

richTextToHTML('abcdef', [
  { start: 1, end: 4, tag: 'i' },
  { start: 1, end: 4, tag: 'b' },
]);
// 'a<i><b>bcd</b></i>ef'
```

**Constraints:**
- Half-open intervals `[start, end)`
- Ranges may overlap, nest, cross, appear in any order
- Earlier ranges in input must be rendered as outer tags
- Duplicate tags in same segment should merge
- Do not mutate input

## Pattern

- [[Object Traversal]]

## 🤔 Thought Process

1. **Find all boundaries** — collect every range start/end plus `0` and `text.length`.
2. **Split text into segments** where the active ranges don't change.
3. For each segment, find its **active tags** in original range order.
4. Use `Set` to deduplicate same tags within a segment.
5. Compare current `tags` with previous `openTags`.
6. Find the **longest common prefix** (`common`).
7. Close tags that are no longer needed, from **inside → outside**.
8. Open new tags from **outside → inside**.
9. Add the segment's text.
10. Update `openTags = tags` for the next segment.

## 💻 Final Solution

```js
export default function richTextToHTML(text, ranges) {
  const boundaries = new Set([0, text.length]);

  for (const range of ranges) {
    boundaries.add(range.start);
    boundaries.add(range.end);
  }

  const positions = Array.from(boundaries).sort((a, b) => a - b);

  let output = "";
  let openTags = [];

  for (let i = 0; i < positions.length - 1; i++) {
    const start = positions[i];
    const end = positions[i + 1];

    // Get tags active for this segment, preserving input order.
    const tags = [];
    const seen = new Set();

    for (const range of ranges) {
      if (range.start <= start && start < range.end) {
        if (!seen.has(range.tag)) {
          seen.add(range.tag);
          tags.push(range.tag);
        }
      }
    }

    // Keep the longest common prefix of currently open tags.
    let common = 0;

    while (
      common < openTags.length &&
      common < tags.length &&
      openTags[common] === tags[common]
    ) {
      common++;
    }

    // Close tags that changed.
    for (let j = openTags.length - 1; j >= common; j--) {
      output += `</${openTags[j]}>`;
    }

    // Open new tags.
    for (let j = common; j < tags.length; j++) {
      output += `<${tags[j]}>`;
    }

    output += text.slice(start, end);
    openTags = tags;
  }

  // Close remaining tags.
  for (let i = openTags.length - 1; i >= 0; i--) {
    output += `</${openTags[i]}>`;
  }

  return output;
}
```

## 🤔 Why This Works

The algorithm treats HTML tags as a **state machine** that changes only at boundaries. By splitting at every range start/end, each segment has a fixed set of active tags. The `tags` array collects active tags in input order, and `openTags` remembers the previous segment's tags. The longest common prefix identifies which tags can remain open. Tags beyond that prefix must close (in reverse order), and new tags open (in forward order). This ensures correct nesting regardless of how ranges overlap, nest, or cross.

Example with overlapping ranges:

```js
[{ start: 1, end: 4, tag: 'b' }, { start: 2, end: 5, tag: 'i' }]
```

Boundaries: `[0, 1, 2, 4, 5, 6]`

Segments:
- `[0,1)`: tags [] → output "a"
- `[1,2)`: tags ['b'] → output "<b>b</b>"
- `[2,4)`: tags ['b','i'] → output "<b><i>c</i></b>"
- `[4,5)`: tags ['i'] → output "<i>d</i>"
- `[5,6)`: tags [] → output "e"
- `[6,6)`: tags [] → output "f"

Final: "a<b>b<i>cd</i></b><i>e</i>f" — correct nesting.

# The 3 things you should memorize

Don't try to memorize the entire code.

Understand these three ideas:

### ① Boundaries

> Split text wherever a range starts or ends.

```
[0,1) [1,2) [2,4) [4,5) [5,6)
```

---

### ② Active tags

> For each segment, determine which ranges are active.

```
"a"  → []
"b"  → [b]
"cd" → [b,i]
"e"  → [i]
"f"  → []
```

---

### ③ Common prefix

> Compare the previous tag stack with the current one.

```
previous = [b]
current  = [b,i]

keep b
open i
```

and:

```
previous = [b,i]
current  = [i]

keep nothing
close i
close b
open i
```

That is essentially the entire problem.

---

## One final mental model

Think of the algorithm as:

```
TEXT
  ↓
split into segments
  ↓
find active tags for each segment
  ↓
compare with previous tags
  ↓
keep common tags
  ↓
close old tags
  ↓
open new tags
  ↓
append text
```

If you understand **why we split at boundaries** and **why `common` is needed**, the code becomes much less mysterious.

And one small implementation detail specific to the GFE runner you encountered: use

```
Array.from(boundaries)
```

rather than `[...boundaries]` there, since your runner behaved unexpectedly with Set spread.

## 🐞 Bugs I Made

* **Reopening/closing everything** — earlier versions would reset tags each segment, producing redundant HTML.
* **Duplicate tags** — overlapping same-tag ranges would create nested identical tags; `seen` prevents this.
* **Set vs array** — in GFE runner, use `Array.from(boundaries)` not spread.
* **Missing final close** — tags left open after last segment must be closed.

## Production Considerations

- For very large documents, the boundary array may be large; consider streaming or more efficient interval merging.
- The algorithm is O(n + m) where n = number of ranges, m = number of boundaries.
- Tags are assumed safe (no attributes, no special chars).
- This approach works for any number of overlapping/nesting ranges.

## ⭐ Revision Notes

### 🔑 Key Facts

* **Half-open ranges**: `[start, end)` includes `start` but excludes `end`.
* **`seen`**: prevents duplicate tag names within a segment.
* **`openTags`**: stores previous segment's tag structure, updated each iteration.
* **`common`**: length of matching prefix between old and new tag lists.
* **Boundary splitting**: ensures each segment has constant active tags.

### 🧠 Mental Model

Think of the algorithm as walking through the text, maintaining a stack of currently open tags. At each boundary, the stack may change — some tags close, some open. The longest common prefix determines what's unchanged. This is exactly how browsers render rich text internally.

```text
Text
 ↓
Find boundaries
 ↓
Create segments
 ↓
Find active tags
 ↓
        ┌───────────────┐
        │ Compare with  │
        │ openTags      │
        └───────┬───────┘
                ↓
       Find common prefix
          ↙           ↘
      Keep tags     Changed tags
                       ↓
                  Close old
                  Open new
                       ↓
                Add text segment
                       ↓
                openTags = tags
```

### Common Interview Questions

- How do you handle crossing ranges? The boundary splitting naturally handles them — each segment gets the correct active tags.
- Why not just sort ranges by start? Order matters for outer/inner — input order determines nesting.
- What about adjacent ranges with same tag? They merge because `seen` dedupes and `common` keeps the tag open.

### Interview Takeaways

The key insight: **split at every change point, then apply tag state transitions**. This pattern generalizes to any markup problem where you need to maintain valid nesting while handling arbitrary range specifications.

### Related

- [[Object Traversal]]
- [[Range]]
- [[HTML]]
- [[Tag Nesting]]

## Related Concepts

- [[Lexical Analysis]]
- [[State Machine]]
- [[Interval Merging]]