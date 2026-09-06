---
aliases:
  - String Search
---

## Core Idea

Find occurrences of a substring (query) within a larger string (text). In interview problems, this often includes:
- Case-insensitive matching.
- Non-overlapping matches (first match wins).
- Combining adjacent/overlapping matches into single regions.
- Highlighting/marking matches (e.g., wrapping in tags).

## Recognition

Use this pattern when:
- The problem asks to find substrings in a string.
- Matches need to be marked, counted, replaced, or wrapped.
- Overlapping/adjacent match handling is specified.
- Case-insensitive comparison is required but original casing must be preserved.

## Template

```js
function stringSearch(text, query) {
  if (!text || !query) return text;

  // Pass 1: mark matches
  const marks = Array(text.length).fill(0);
  const q = query.toLowerCase();

  for (let i = 0; i < text.length; ) {
    if (text.slice(i, i + query.length).toLowerCase() === q) {
      marks.fill(1, i, i + query.length);
      i += query.length; // advance past match to avoid overlap
    } else {
      i++;
    }
  }

  // Pass 2: build result from marks
  let result = '';
  for (let i = 0; i < text.length; i++) {
    const open = marks[i] === 1 && marks[i - 1] !== 1;
    const close = marks[i] === 1 && marks[i + 1] !== 1;

    if (open) result += '<b>';
    result += text[i];
    if (close) result += '</b>';
  }

  return result;
}
```

## Variations

- **Replace matches**: build result with replacement string instead of tags.
- **Count matches**: `marks.reduce((sum, m) => sum + m, 0)`.
- **Return indices**: collect `i` where match starts.
- **Regex-based**: `/query/gi` with `.replace()`, but less control over overlapping/adjacent logic.
- **KMP / Boyer-Moore**: for large texts where O(n*m) is too slow (rare in interviews).

## Complexity

Time: **O(n * m)** where n = text length, m = query length (for naive slice/compare).
With KMP: **O(n + m)**.
Space: **O(n)** for the marks array + output.

## Common Mistakes

- Not advancing `i` by `query.length` after a match → overlapping matches counted multiple times.
- Using `String.replace()` with regex → hard to control "first match wins" and tag merging.
- Forgetting to preserve original casing — must compare lowercase but output original.
- Not handling empty query / text edge cases.

## Interview Tips

- State the two-pass approach clearly: mark → render.
- Explain why the mask array is better than inline string building: automatically merges adjacent matches.
- Mention that for very large texts, the mask array uses O(n) space; a streaming approach is possible but more complex.
- When asked about regex: "Regex can't easily do 'advance past match' without consuming characters, so the mask approach is more precise for this specific requirement."

## Problems Using This Pattern

- [[Text Search]]
- [[Text Search II]]

## Related Patterns

- [[Array Traversal]]
- [[Sliding Window]]

## Related Concepts

- [[String Manipulation]]
- [[Case Insensitive Comparison]]
- [[KMP Algorithm]]