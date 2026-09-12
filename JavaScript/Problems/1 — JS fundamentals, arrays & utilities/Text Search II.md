---
title: Implement textSearch(text, queries) to bold case-insensitive matches from multiple queries
aliases:
  - Text Search II
difficulty: Medium
time: 25 min
languages:
  - JavaScript
companies: []
pattern:
  - "[[String Search]]"
concepts:
  - "[[String Manipulation]]"
  - "[[Case Insensitive]]"
  - "[[Array Iteration]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-07
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 25 min
> Find all case-insensitive matches from an array of queries in a text, wrapping them in `<b>` tags and merging adjacent/overlapping matches.

## Problem

Implement a function `textSearch(text, queries)` that finds all case‑insensitive matches from the `queries` array within `text`, wrapping matches in `<b>...</b>` tags. If matches overlap or are consecutive, they should be wrapped in a single pair of tags.

```js
/**
 * @param {string} text
 * @param {string[]} queries
 * @return {string}
 */
```

**Examples**

```js
textSearch('The Quick Brown Fox Jumps Over The Lazy Dog', ['fox']);
// 'The Quick Brown <b>Fox</b> Jumps Over The Lazy Dog'

textSearch('The quick brown fox jumps over the lazy dog', ['fox', 'dog']);
// 'The quick brown <b>fox</b> jumps over the lazy <b>dog</b>'

textSearch('This is Uncopyrightable!', ['copy', 'right']);
// 'This is Un<b>copyright</b>able!'

textSearch('This is Uncopyrightable!', ['copy', 'right', 'table']);
// 'This is Un<b>copyrightable</b>!'

textSearch('aaa', ['aa']);
// '<b>aa</b>a'

textSearch('aaaa', ['aa']);
// '<b>aaaa</b>'
```

The input `queries` contains no duplicates. Empty queries or an empty text should return the original text unchanged.

## Pattern

- [[String Search]]

## 🤔 Thought Process

* Iterate over each query independently.
* For each query, scan the text left‑to‑right.
* When a match is found, mark the corresponding positions in a `bold` mask and advance the index by the query length to avoid re‑using characters.
* After all queries are processed, build the output by inserting `<b>`/`</b>` at transitions in the mask.

## 💻 Final Solution

```js
export default function textSearch(text, queries) {
  if (!text || text.trim() === '' || !queries || queries.length === 0) {
    return text;
  }

  const bold = Array.from({ length: text.length }, () => 0);

  for (const query of queries) {
    if (!query || query.trim() === '') continue;
    const qLow = query.toLowerCase();

    for (let i = 0; i < text.length; ) {
      const substr = text.slice(i, i + query.length);
      if (substr.toLowerCase() === qLow) {
        bold.fill(1, i, i + query.length);
        i += query.length; // skip over matched segment
      } else {
        i++;
      }
    }
  }

  let result = '';
  for (let i = 0; i < text.length; i++) {
    const ch = text[i];
    const open = bold[i] === 1 && bold[i - 1] !== 1;
    const close = bold[i] === 1 && bold[i + 1] !== 1;
    if (open) result += '<b>';
    result += ch;
    if (close) result += '</b>';
  }
  return result;
}
```

## 🤔 Why This Works

The `bold` array marks every character that must be highlighted. By filling the array only once per query and advancing the index after a match, the algorithm guarantees that a character is never used twice, giving priority to earlier matches. After all queries are processed, the mask is converted to HTML by detecting transitions from non‑bold to bold (insert `<b>`) and from bold to non‑bold (insert `</b>`), automatically merging adjacent matches.

## Production Considerations

* The algorithm runs in `O(n * m)` time, where `n` is text length and `m` is the sum of query lengths. For most interview cases this is acceptable.
* The solution keeps the original casing of `text` while performing case‑insensitive comparison.
* Empty strings in `queries` are skipped; an empty `text` returns unchanged.

## ⭐ Revision Notes

### 🔑 Key Facts

* Use a mask array to record bold positions.
* Advance the scanning index by `query.length` after a match to prevent overlapping.
* Merge adjacent matches automatically via transition detection.

### Common Interview Questions

- How do you handle overlapping queries? The mask approach naturally gives "first match wins".
- What if a query is longer than the remaining text? The slice simply returns a shorter string; comparison fails and the loop continues.
- How would you optimize for many queries? Pre‑process queries into a trie or use Aho‑Corasick to find all matches in one pass.

### Interview Takeaways

The clean pattern: **scan each query → mark matches → render transitions**.
