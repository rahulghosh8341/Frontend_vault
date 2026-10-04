---
title: Styled Text Ranges II
aliases:
  - Styled Text Ranges II
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/styled-text-ranges-ii"
companies:
  - "[[Figma]]"
pattern:
  - "[[Range]]"
concepts:
  - "[[Range]]"
  - "[[Object Immutability]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Styled Text Ranges II

> [!info] Problem
> Implement a function that overwrites the style of a text range and normalizes the result

## Problem

## Styled Text Ranges II

This is a follow-up to [Styled Text Ranges](/questions/javascript/styled-text-ranges).

In design tools, styling a text selection usually means splitting the affected runs, applying the new style, and then merging adjacent runs that now match.

Implement `setTextRangeStyle(node, start, end, style)` using the same simplified styled text representation as part 1.

## Examples

```javascript
setTextRangeStyle(
  {
    text: 'Hello world',
    ranges: [
      { start: 0, end: 6, style: 'body' },
      { start: 6, end: 11, style: 'bold' },
    ],
  },
  3,
  8,
  'highlight',
);
// {
//   text: 'Hello world',
//   ranges: [
//     { start: 0, end: 3, style: 'body' },
//     { start: 3, end: 8, style: 'highlight' },
//     { start: 8, end: 11, style: 'bold' },
//   ],
// }
```

```javascript
setTextRangeStyle(
  {
    text: 'Hello',
    ranges: [{ start: 0, end: 5, style: 'body' }],
  },
  2,
  2,
  'highlight',
);
// {
//   text: 'Hello',
//   ranges: [{ start: 0, end: 5, style: 'body' }],
// }
```

## Arguments

`setTextRangeStyle(node, start, end, style)`

- `node` (`object`): A styled text object containing `text` and normalized `ranges`.
- `start` (`number`): Inclusive range start.
- `end` (`number`): Exclusive range end.
- `style` (`string`): The style token to apply across the selected range.

## Returns

Returns a new styled text object with the selected range overwritten by `style`.

## Notes

- Inputs are guaranteed valid.
- Empty selections should be a no-op.
- Merge adjacent ranges in the result when they have the same `style`.
- Do not mutate the input.

## Hints

### Hint 1 : How can one range cross the selection?

### Hint 2 : What normalization can restyling break?

## Asked at these companies

Figma

## 🤔 Thought Process

- **Immediate Recognition:** Rich-text inline range styling (similar to Figma text layers, Google Docs, or Draft.js attribute spans).
- **Core Problem:** Given a normalized sequence of non-overlapping style intervals covering `0..text.length`, overwrite the style on `[start, end)` and re-normalize adjacent matching intervals.
- **Three-Slice Interval Splitting:**
  - For each existing range `[r.start, r.end)`:
    - If strictly before `start` or strictly after `end`, keep unchanged.
    - If overlapping with selection:
      - Prefix piece before selection: `[r.start, max(r.start, start))` with original style.
      - Overwritten piece inside selection: `[max(r.start, start), min(r.end, end))` with new `style`.
      - Suffix piece after selection: `[min(r.end, end), r.end)` with original style.
  - Alternatively: simply add prefix pieces, one central `[start, end, style]` range, and suffix pieces!
- **Normalization (Run-Length Consolidation):**
  - Walk the raw intervals in ascending order.
  - Drop zero-length intervals (`r.start === r.end`).
  - If current range has same `style` as previous range and `prev.end === curr.start`, extend `prev.end = curr.end`. Otherwise push as new run.

---

## 🧠 Mental Model

Think of **Painting a Colored Paper Ribbon**:
- The ribbon has consecutive colored strips glued edge-to-edge: `[Blue: 0..6][Red: 6..11]`.
- You lay down a stencil from `3` to `8` and spray paint it `Green`.
- Slicing occurs at cut-points `3` and `8`:
  - Leftover blue strip: `0..3`.
  - New green strip: `3..8`.
  - Leftover red strip: `8..11`.
- **Normalization:** If adjacent strips end up having the same color, merge them into a single continuous strip.

---

## 🔑 Key Concepts

- Interval slicing and boundary overlap math (`Math.max`, `Math.min`)
- Interval normalization / merging adjacent runs
- Non-destructive immutable data transformations
- Text buffer representation in rich text editors (attribute runs / spanners)

---

## ⚠️ Edge Cases / Traps

- **Empty Selection (`start === end`):** Must be an immediate no-op returning a clone of the input node without splitting.
- **Selection Spanning Multiple Intervals:** The selection `[start, end)` can start inside range A, completely swallow range B, and terminate inside range C.
- **Adjacent Same-Style Merge:** If the newly applied `style` matches the existing style of the preceding or succeeding interval, they must merge into one single interval.
- **Do Not Mutate Input:** Never mutate `node.ranges` in-place. Always return fresh objects.
- **Zero-Width Slices:** Slicing at exact boundaries (e.g. `start === r.start`) produces empty `[r.start, r.start)` segments. Filter out zero-length segments before or during normalization.

---

## ⭐ Interview Takeaway

1. **Clean Three-Way Partition:** Partition the timeline into `before` (`< start`), `inside` (`[start, end)`), and `after` (`> end`).
2. **Dedicated Normalizer Function:** Separate the range-application logic from the adjacent-merge normalization logic. Writing a standalone `normalize(ranges)` keeps code modular and bug-free.
3. **Figma / Editor Architecture:** This is the foundational interview question testing your understanding of rich-text data structures (attributed strings).

---

## 🎯 Common Interview Questions

### Direct Questions
- How do you determine whether an existing range overlaps with the target range `[start, end)`?
- Why is a post-processing normalization step required after inserting the styled range?
- How do you handle empty ranges when the selection boundary aligns with an existing range boundary?

### Follow-up Questions
- How would you handle multiple disjoint selections (`multi-cursor` editing)?
- What changes if styles are objects with multiple independent properties (like `{ bold: true, italic: true }`) instead of a single string? (See [[Styled Text Ranges IV]])
- How does inserting or deleting text affect style ranges? (See [[Styled Text Ranges III]])

### Conceptual Questions
- What are the tradeoffs between inline HTML tags (`<b><i>...</i></b>`), linear range spans, and tree-based document models (ProseMirror / Slate)?
- How do collaborative editors (CRDTs / Operational Transformation) resolve conflicting range styling?

---

## 🔄 Variations

- **Styled Text Ranges I:** Basic run representation and rendering to HTML.
- **Styled Text Ranges III:** Replacing text ranges and shifting subsequent interval offsets.
- **Styled Text Ranges IV:** Merging style property objects (`bold`, `italic`, `color`) over text ranges.

---

## 📝 Revision Notes

- **Core idea:** Slice existing ranges at `start` and `end`; insert `[start, end, style]`; merge adjacent matching styles.
- **Remember:** Empty selection (`start === end`) is a no-op; drop zero-length ranges.
- **Watch out for:** Normalization must merge ranges with identical styles when `prev.end === curr.start`.
- **Complexity:** Time: $O(R)$ where $R$ is number of ranges; Space: $O(R)$ for new ranges array.

## Official Solution
## Styled Text Ranges II ( Official solution )

Premium
Languages

## Solution

This follow-up adds the first mutation operation on top of the part I representation. The change from part I is narrow: part I clips an existing window, while part II overwrites the style inside a window and then restores the canonical merged form.

The write operation replaces the overlapping style run, keeps everything else, then renormalizes. The selection boundaries behave like open/close points: before `start`, keep the old style open; inside `[start, end)`, open the new style; after `end`, close the new style and continue with the old suffix when one exists.

The split step has four jobs:

1. Return a cloned copy unchanged for empty selections.
2. Keep non-overlapping ranges in their original order.
3. Split each overlapping range into up to three fragments: untouched prefix, newly styled middle, and untouched suffix.
4. Merge adjacent ranges with the same style.

That final merge pass restores the rule expected by later operations. Without it, a single styling change across several source ranges could leave neighboring fragments that really represent one continuous styled segment.

The half-open interval rule drives every split. A source range overlaps the selection only when `range.start < end` and `range.end > start`; touching at exactly `start` or `end` means the ranges are adjacent, not overlapping.

For a selection `[1, 9)`, a source range `[0, 2) body` emits `[0, 1) body` plus `[1, 2) newStyle`, while `[6, 11) comment` emits `[6, 9) newStyle` plus `[9, 11) comment`. The merge pass then joins neighboring `newStyle` fragments produced from different source ranges.

```jsx
/**
 * @typedef {string} TextStyle
 *
 * @typedef {{
 *   start: number,
 *   end: number,
 *   style: TextStyle,
 * }} StyledTextRange
 *
 * @typedef {{
 *   text: string,
 *   ranges: Array<StyledTextRange>,
 * }} StyledText
 */
function cloneRange(range) {
  return {
    start: range.start,
    end: range.end,
    style: range.style,
  };
}

function mergeAdjacentRanges(ranges) {
  return ranges.reduce((mergedRanges, range) => {
    const previousRange = mergedRanges[mergedRanges.length - 1];

    // Restyling can make neighbors identical again, so normalize them back into one span.
    if (
      previousRange != null &&
      previousRange.end === range.start &&
      previousRange.style === range.style
    ) {
      previousRange.end = range.end;
      return mergedRanges;
    }

    mergedRanges.push(cloneRange(range));
    return mergedRanges;
  }, []);
}

/**
 * @param {StyledText} node
 * @param {number} start
 * @param {number} end
 * @param {TextStyle} style
 * @returns {StyledText}
 */
export default function setTextRangeStyle(node, start, end, style) {
  if (start === end) {
    return {
      text: node.text,
      ranges: node.ranges.map(cloneRange),
    };
  }

  const nextRanges = [];

  node.ranges.forEach((range) => {
    if (range.end <= start || range.start >= end) {
      nextRanges.push(cloneRange(range));
      return;
    }

    // Rewrite an overlap as up to three fragments: prefix, restyled middle, suffix.
    if (range.start < start) {
      nextRanges.push({
        start: range.start,
        end: start,
        style: range.style,
      });
    }

    nextRanges.push({
      start: Math.max(range.start, start),
      end: Math.min(range.end, end),
      style,
    });

    if (range.end > end) {
      nextRanges.push({
        start: end,
        end: range.end,
        style: range.style,
      });
    }
  });

  return {
    text: node.text,
    ranges: mergeAdjacentRanges(nextRanges),
  };
}
```

## Walkthrough

Suppose the input ranges are:

```javascript
[
  { start: 0, end: 2, style: 'body' },
  { start: 2, end: 6, style: 'link' },
  { start: 6, end: 11, style: 'comment' },
];
```

Applying `highlight` to `[1, 9)` emits `[0, 1)` as `body`, then three adjacent `highlight` fragments from `[1, 2)`, `[2, 6)`, and `[6, 9)`, followed by `[9, 11)` as `comment`. The merge pass collapses those neighboring `highlight` fragments into one `[1, 9)` range while preserving the surrounding order.

| Source range | Emitted fragments before merge |
| --- | --- |
| `[0, 2) body` | `[0, 1) body`, `[1, 2) highlight` |
| `[2, 6) link` | `[2, 6) highlight` |
| `[6, 11) comment` | `[6, 9) highlight`, `[9, 11) comment` |

After merging adjacent `highlight` fragments, the result is `[0, 1) body`, `[1, 9) highlight`, `[9, 11) comment`.

## Common pitfalls

- **Replacing only whole ranges:** The selection can start or end in the middle of a range. Overlapping ranges need prefix and suffix fragments so text outside `[start, end)` keeps its original style.
- **Skipping the normalization pass:**

Restyling can make the middle fragment match its left neighbor, right neighbor, or both. Merge only adjacent ranges whose boundaries touch and whose styles are the same.

### Using inclusive overlap checks

The same half-open interval rules from part I still apply. A range ending at `start` or starting at `end` is not part of the selection.

### Treating an empty selection as a zero-length styled range

When `start === end`, the operation is a no-op. It should not insert a new range or change the text.

### Mutating existing ranges

The code clones preserved ranges before returning them so callers do not observe changes to the input object.

## Notes

- Styling inside one range can split it into three pieces.
- Styling across multiple ranges can create several adjacent fragments with the same new style, which should merge.
- Styling the entire text should usually collapse the result to one range.
- Empty selections should return the original content unchanged.
- The plain text does not change in this part; only the range list is rewritten.
- Inputs are assumed to be valid and normalized from part I, so the update can focus on rewriting existing ranges rather than sorting, filling gaps, or validating offsets.

## Techniques

- Half-open intervals
- Range splitting
- Normalization through adjacent-merge passes

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A restyler changes the style of every source range that overlaps the selection, without splitting those ranges. Whole-text tests pass. Design a small test that proves the implementation changes unselected characters, and explain the fragments a correct result needs.

Your notes (optional)
