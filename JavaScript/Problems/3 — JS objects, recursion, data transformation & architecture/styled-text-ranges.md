---
title: "Styled Text Ranges"
aliases:
  - "sliceStyledText"
  - "Styled Text Ranges"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Styled Text Ranges

> [!info] Problem
> Implement a function that slices styled text represented as text plus style ranges

## Problem

## Styled Text Ranges

Design tools, for example [Figma](https://developers.figma.com/docs/plugins/working-with-text/), often represent styled text as a string plus a list of ranges that describe which style applies to each part of the text.

In this question, implement `sliceStyledText(node, start, end)` for a simplified Figma-inspired representation.

Each range is half-open, meaning `[start, end)`. Inputs are always valid: ranges are sorted, non-overlapping, and cover the full text.

## Examples

```javascript
sliceStyledText(
  {
    text: 'Hello world',
    ranges: [
      { start: 0, end: 6, style: 'body' },
      { start: 6, end: 11, style: 'bold' },
    ],
  },
  3,
  8,
);
// {
//   text: 'lo wo',
//   ranges: [
//     { start: 0, end: 3, style: 'body' },
//     { start: 3, end: 5, style: 'bold' },
//   ],
// }
```

```javascript
sliceStyledText(
  {
    text: 'Hello',
    ranges: [{ start: 0, end: 5, style: 'body' }],
  },
  2,
  2,
);
// {
//   text: '',
//   ranges: [],
// }
```

## Arguments

`sliceStyledText(node, start, end)`

- `node` (`object`): A styled text object:
  - `text` (`string`): The full text.
  - `ranges` (`Array`): Sorted, non-overlapping ranges that cover the full text.

- `start` (`number`): Inclusive slice start.
- `end` (`number`): Exclusive slice end.

Each range has this shape:

```javascript
{
  start: number,
  end: number,
  style: string,
}
```

## Returns

Returns a new styled text object for the sliced range. Range offsets in the result should be relative to the new sliced text.

## Notes

- Inputs are guaranteed valid.
- Do not mutate `node` or its nested `ranges`.
- Empty slices should return `{ text: '', ranges: [] }`.

## Hints

### Hint : Which part of a style range survives?

## Asked at these companies

- [[Figma]]

## 🤔 Thought Process

- **Immediate Recognition:** Figma-inspired rich text slicing utility. Given `{ text, ranges }` and slice bounds `[start, end)`, slice the text and clip/offset the corresponding style ranges.
- **Range Format:**
  - Half-open intervals: `[range.start, range.end)`.
  - Non-overlapping, sorted, and covering full text.
- **Slice Window `[start, end)` Mechanics:**
  - Sliced text is simply `node.text.slice(start, end)`.
  - For each range `{ start: rStart, end: rEnd, style }`:
    - Check overlap: Intersection between `[start, end)` and `[rStart, rEnd)`.
    - Overlap condition: `Math.max(start, rStart) < Math.min(end, rEnd)`.
    - If overlap exists:
      - Sliced range start: `Math.max(start, rStart) - start` (re-indexed relative to slice start).
      - Sliced range end: `Math.min(end, rEnd) - start`.
      - Preserve `style`.
- **Empty Slice:**
  - If `start >= end`, return `{ text: '', ranges: [] }`.

---

## 🧠 Mental Model

Think of a **Camera Window over an Interval Strip**:
```
Original Text:   H e l l o _ w o r l d  (Length 11)
Original Ranges: [0─────6)   [6──────11)
                   body         bold

Slice Window:          [3──────8)
                         l o _ w o

Clipped Ranges:
  - Range 1: max(3, 0) to min(8, 6) = [3, 6) -> Re-indexed: [0, 3) (body)
  - Range 2: max(3, 6) to min(8, 11) = [6, 8) -> Re-indexed: [3, 5) (bold)
```

---

## 🔑 Key Concepts

- [[Range Checking]]
- [[Array Traversal]]
- Interval Intersection Algorithm (`max(start1, start2) < min(end1, end2)`)
- Coordinate Re-indexing / Normalization (`pos - offset`)
- Half-open interval `[start, end)` arithmetic

---

## ⚠️ Edge Cases / Traps

- **Zero-width Slice:** If `start === end`, return `{ text: '', ranges: [] }`.
- **Slice Covering Single Range:** If the slice lies entirely within one range, return a single range `[{ start: 0, end: end - start, style }]`.
- **Slice Straddling Multiple Ranges:** Each overlapping range must be clipped to the slice bounds and shifted by `-start`.
- **Out of Bounds `start` or `end`:** Ensure clipping guards against indices beyond text length if not pre-clamped.

---

## ⭐ Interview Takeaway

- Two-part algorithm:
  1. Slice raw string: `text.slice(start, end)`.
  2. Map overlapping intervals with clipping:
     ```javascript
     const oStart = Math.max(start, r.start);
     const oEnd = Math.min(end, r.end);
     if (oStart < oEnd) {
       ranges.push({
         start: oStart - start,
         end: oEnd - start,
         style: r.style,
       });
     }
     ```
- Used directly in rich text editors (Draft.js, Slate, Quill) and design tools (Figma, Canva).

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you calculate the intersection of two half-open intervals `[s1, e1)` and `[s2, e2)`?" (`[Math.max(s1, s2), Math.min(e1, e2))` is valid if `max < min`).
- "Why do we subtract `start` from the overlapping range coordinates?" (To normalize coordinates so the new sliced string starts at index 0).

### Follow-up Questions
- "How would you handle overlapping ranges with style priority/layering?" (Merge styles into composite objects or apply z-index).
- "How would you implement inserting text into a styled text node?" (Expand or split the range at the insertion cursor).

### Conceptual Questions
- "Why do modern rich text engines prefer linear range annotations over nested HTML DOM trees?" (Avoids invalid HTML nesting states, simplifies undo/redo histories, and maps cleanly to CRDT operational transforms).

---

## 🔄 Variations

- **Styled Text Ranges II, III, IV:** Adding styling operations, style merging, and deletions.
- **Merge Overlapping Intervals (LeetCode 56):** Merging adjacent intervals.
- **Insert Interval (LeetCode 57):** Slicing and inserting into sorted interval lists.

---

## 📝 Revision Notes

- Clean implementation:
```javascript
export default function sliceStyledText(node, start, end) {
  if (start >= end) {
    return { text: '', ranges: [] };
  }

  const slicedText = node.text.slice(start, end);
  const slicedRanges = [];

  for (const range of node.ranges) {
    const oStart = Math.max(start, range.start);
    const oEnd = Math.min(end, range.end);

    if (oStart < oEnd) {
      slicedRanges.push({
        start: oStart - start,
        end: oEnd - start,
        style: range.style,
      });
    }
  }

  return {
    text: slicedText,
    ranges: slicedRanges,
  };
}
```

---

## Official Solution

## Styled Text Ranges ( Official solution )

Premium
Languages
The failure mode is returning ranges that still point at the original text. Slicing text is easy; every surviving style interval must also be clipped to the slice window and rebased to the new string's index zero.

## Solution

This first part is a clip-and-rebase pass over already-normalized style ranges. The tempting but weaker approach is to slice text character by character and rebuild styles from scratch. The better interview model is interval arithmetic: a range start is where a style run opens, and a range end is where it closes. Slicing does not change the order of those boundaries; it only keeps the parts that are still open inside `[start, end)`.

The condition for every output range is: it represents the intersection of one original range with the slice window, shifted left by `start`.

Treat both the original ranges and the slice `[start, end)` as half-open intervals. The formatter has four jobs:

1. Build the sliced text with `text.slice(start, end)`.
2. Clamp each range to the slice window with `Math.max(range.start, start)` and `Math.min(range.end, end)`.
3. Drop the range when the clamped bounds do not form a positive-length overlap.
4. Subtract `start` from both surviving bounds so offsets are relative to the new sliced text.

Because the input ranges are sorted and non-overlapping, a single pass preserves the output order. No extra open/close bookkeeping is needed beyond clipping each range to the slice window.

The result should still describe the new sliced text, not the original document. That is why a surviving overlap like original `[7, 9)` becomes `[3, 5)` when the slice starts at `4`.

```jsx
/**
 * @typedef {string} TextStyle
 *
 * @typedef {object} StyledTextRange
 * @property {number} start
 * @property {number} end
 * @property {TextStyle} style
 *
 * @typedef {object} StyledText
 * @property {string} text
 * @property {Array<StyledTextRange>} ranges
 */

/**
 * @param {StyledText} node
 * @param {number} start
 * @param {number} end
 * @returns {StyledText}
 */
export default function sliceStyledText(node, start, end) {
  return {
    text: node.text.slice(start, end),
    ranges: node.ranges.flatMap((range) => {
      // Clamp to the slice window, then rebase the surviving range onto the sliced text.
      const overlapStart = Math.max(range.start, start);
      const overlapEnd = Math.min(range.end, end);

      if (overlapStart >= overlapEnd) {
        return [];
      }

      return [
        {
          start: overlapStart - start,
          end: overlapEnd - start,
          style: range.style,
        },
      ];
    }),
  };
}
```

## Walkthrough

For a slice `[4, 9)` over these ranges:

```javascript
[
  { start: 0, end: 2, style: 'body' },
  { start: 2, end: 7, style: 'comment' },
  { start: 7, end: 10, style: 'quote' },
];
```

The first range closes before the slice opens, so it contributes nothing. The `comment` range overlaps as `[4, 7)`, which rebases to `[0, 3)`. The `quote` range overlaps as `[7, 9)`, which rebases to `[3, 5)`. The resulting ranges cover the sliced text from `0` to `5` without gaps.

## Common pitfalls

- **Treating `end` as inclusive:** The ranges are half-open. A range ending exactly at `start`, or starting exactly at `end`, does not overlap the slice. Using inclusive checks can createzero-length ranges or duplicate a boundary character.

### Forgetting to rebase offsets

The returned `text` starts at index `0`, so every surviving range must subtract the original slice `start`. Keeping original offsets makes the result no longer line up with the sliced string.

### Merging or restyling ranges

Part I is read-only. It should preserve the original style sequence except for clipping and offset rebasing.

### Mutating the input ranges

The output should be a new styled text object. The original `node` and its nested `ranges` should remain unchanged.

## Notes

- A slice can fall fully within one range.
- A slice can span multiple ranges and split both the first and last one.
- An empty slice should return an empty `text` and no ranges.
- The original input should remain unchanged.
- Inputs are guaranteed valid: ranges are sorted, non-overlapping, and cover the full text.

## Techniques

- Half-open intervals
- Range overlap checks
- Offset rebasing

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A slice implementation keeps an intersection when `overlapStart <= overlapEnd`. Which test specifically exposes the extra zero-length range this permits?
