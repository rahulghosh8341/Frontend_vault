---
title: Styled Text Ranges III
aliases:
  - Styled Text Ranges III
difficulty: Hard
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/styled-text-ranges-iii"
companies:
  - "[[Figma]]"
pattern:
  - "[[Range]]"
concepts:
  - "[[Range]]"
  - "[[String Manipulation]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Styled Text Ranges III

> [!info] Problem
> Implement a function that replaces a text range and keeps the style ranges normalized

## Problem

## Styled Text Ranges III

This is a follow-up to [Styled Text Ranges II](/questions/javascript/styled-text-ranges-ii).

Once a styled text model can restyle selections, the next natural step is editing the text itself while keeping the style ranges in sync.

Implement `replaceTextRange(node, start, end, replacement)` for the same simplified styled text representation.

## Examples

```javascript
replaceTextRange(
  {
    text: 'Hello world',
    ranges: [
      { start: 0, end: 6, style: 'body' },
      { start: 6, end: 11, style: 'bold' },
    ],
  },
  5,
  11,
  { text: ' there', style: 'body' },
);
// {
//   text: 'Hello there',
//   ranges: [{ start: 0, end: 11, style: 'body' }],
// }
```

```javascript
replaceTextRange(
  {
    text: 'Hello',
    ranges: [{ start: 0, end: 5, style: 'body' }],
  },
  2,
  2,
  { text: '!', style: 'body' },
);
// {
//   text: 'He!llo',
//   ranges: [{ start: 0, end: 6, style: 'body' }],
// }
```

## Arguments

`replaceTextRange(node, start, end, replacement)`

- `node` (`object`): A styled text object containing `text` and normalized `ranges`.
- `start` (`number`): Inclusive replacement start.
- `end` (`number`): Exclusive replacement end.
- `replacement` (`object`):
  - `text` (`string`): Replacement text. An empty string acts as deletion.
  - `style` (`string`): Style for inserted replacement text.

## Returns

Returns a new styled text object with the text replaced and the ranges updated to match.

## Notes

- Inputs are guaranteed valid.
- `start === end` acts as an insertion.
- `replacement.text === ''` acts as a deletion.
- Merge adjacent ranges in the result when they have the same `style`.
- Do not mutate the input.

## Hints

### Hint 1 : Which three regions survive a replacement?

### Hint 2 : How far do later ranges move?

## Asked at these companies

Figma

## 🤔 Thought Process

- **Immediate Recognition:** Editing text content while synchronizing offset-based style ranges (text mutation + range delta adjustment).
- **Core Problem:** Replace the text substring between `[start, end)` with `replacement.text` having `replacement.style`. Then update the entire `ranges` array so indices reflect the expanded/contracted string length and new style.
- **Delta Calculation:**
  - Length of deleted text: `deletedLen = end - start`.
  - Length of inserted text: `insertedLen = replacement.text.length`.
  - Shift offset for all subsequent characters: `delta = insertedLen - deletedLen`.
- **Three-Region Range Partitioning:**
  1. **Before `start` (`r.end <= start`):** Unaffected. Kept as-is.
  2. **Intersecting `start` / `end`:**
     - Prefix before `start`: `[r.start, start)` with original style.
     - New replacement run: `[start, start + insertedLen)` with `replacement.style` (if `insertedLen > 0`).
     - Suffix after `end`: `[end, r.end)` shifted by `delta` -> `[start + insertedLen, r.end + delta)` with original style.
  3. **After `end` (`r.start >= end`):** Shifted entirely by `delta`: `[r.start + delta, r.end + delta)` with original style.
- **Normalization:** Remove 0-length ranges and merge adjacent intervals sharing the same style.

---

## 🧠 Mental Model

Think of an **Elastic Measuring Tape with Colored Sections**:
- You snip out a section from tick mark `5` to `11` (6 units removed).
- You splice in a new rubber band of length 6 (or 1, or 10) with its own color.
- All tape sections to the left of the cut remain at their original tick marks.
- The spliced rubber band sits at `[5, 5 + newLength)`.
- All sections to the right are dragged along: their start and end numbers shift by `newLength - oldLength`.
- Finally, any adjacent sections of the same color melt together.

---

## 🔑 Key Concepts

- Text mutation with index re-mapping (Delta offset math)
- Pure insertions (`start === end`), pure deletions (`replacement.text === ''`), and replacements
- Coordinate shifting for downstream intervals
- Post-replacement interval compaction / normalization

---

## ⚠️ Edge Cases / Traps

- **Pure Insertion (`start === end`):** `deletedLen === 0`. The new text is inserted and shifts subsequent ranges right by `replacement.text.length`.
- **Pure Deletion (`replacement.text === ''`):** `insertedLen === 0`. No replacement interval should be emitted; subsequent intervals shift left by `end - start`.
- **Replacement Boundary Splitting Inside Single Range:** Replacing a word inside an existing range splits that single range into three pieces: prefix + replacement + suffix (shifted).
- **Empty Output Text:** Deleting the entire text produces `{ text: '', ranges: [] }`.
- **Adjacent Range Coalescing:** If replacement text has style `'body'` and is inserted right next to an existing `'body'` interval, they must merge into one range.

---

## ⭐ Interview Takeaway

1. **Shift Delta Formula:** Calculate `const delta = replacement.text.length - (end - start)` immediately. Every coordinate that begins at or after `end` is shifted by `+ delta`.
2. **Handle Empty Insertion:** Only push the replacement range if `replacement.text.length > 0`.
3. **Re-use Normalizer:** A single helper `normalizeRanges(ranges)` handles both dropping zero-width slices and merging adjacent identical styles.

---

## 🎯 Common Interview Questions

### Direct Questions
- How is the coordinate delta calculated when replacing a range of text?
- What happens to ranges strictly to the right of `end` versus ranges strictly to the left of `start`?
- How does your implementation distinguish between text insertion, text deletion, and text replacement?

### Follow-up Questions
- How would you implement multiple simultaneous replacements (e.g. Find and Replace All) without offset collision? (Sort replacements in reverse document order and apply back-to-front).
- How do real text engines (like Monaco Editor / Ace) optimize character range updates over massive files? (Piece tables or 2-3 rope trees with interval trees).

### Conceptual Questions
- What is Operational Transformation (OT) and how does it maintain range offsets across distributed edits?
- What are the advantages of representing styled text as flat offset runs versus nested DOM nodes?

---

## 🔄 Variations

- **Styled Text Ranges II:** Restyling without changing text characters.
- **Styled Text Ranges IV:** Patching composite style objects (`bold`, `italic`, `color`).
- **Batch Replacement:** Applying a list of non-overlapping text replacements simultaneously.

---

## 📝 Revision Notes

- **Core idea:** Slice at `start` and `end`; insert replacement at `start`; shift all subsequent coordinates by `delta = text.length - (end - start)`; normalize.
- **Remember:** If `replacement.text === ''` (deletion), do not add a range for the replacement.
- **Watch out for:** Suffix of an overlapped range must be shifted by `delta`.
- **Complexity:** Time: $O(R + N)$ where $R$ is ranges count and $N$ is text length; Space: $O(R + N)$.

## Official Solution
## Styled Text Ranges III ( Official solution )

Premium
Languages

## Solution

The representation is unchanged, but now the text length can change, so later range offsets can move. The change from part II is that the selected window is not merely restyled; it is spliced out and replaced by new text with its own style.

This is a string splice with range boundaries attached. Everything before `start` keeps its old offsets, the replacement opens at `start`, and everything after `end` reopens at a shifted offset. The shift is the text length delta:

```javascript
replacement.text.length - (end - start);
```

Use the splice structure directly:

1. Build the new plain text with string slicing.
2. Keep the parts of ranges that stay before `start`.
3. Add one inserted replacement range when `replacement.text` is non-empty.
4. Keep the parts of ranges that stay after `end`, shifted by `delta`.
5. Merge adjacent ranges that now share the same style.

The output ordering follows the splice order: before fragments, optional replacement fragment, then shifted after fragments. That is the range-list version of closing any style runs removed by the edit, opening the replacement run, and reopening later runs at their new positions.

The merge pass is intentionally last. The replacement can join with the left fragment, the right fragment, both, or neither depending on style and adjacency after offsets have been translated.

```jsx
/**
 * @typedef {string} TextStyle
 * @typedef {{
 *   start: number,
 *   end: number,
 *   style: TextStyle,
 * }} StyledTextRange
 * @typedef {{
 *   text: string,
 *   ranges: Array<StyledTextRange>,
 * }} StyledText
 * @typedef {{
 *   text: string,
 *   style: TextStyle,
 * }} TextReplacement
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
 * @param {TextReplacement} replacement
 * @returns {StyledText}
 */
export default function replaceTextRange(node, start, end, replacement) {
  // Ranges after the edit shift by the net change in text length.
  const delta = replacement.text.length - (end - start);
  const beforeRanges = [];
  const afterRanges = [];

  node.ranges.forEach((range) => {
    if (range.end <= start) {
      beforeRanges.push(cloneRange(range));
    } else if (range.start < start) {
      beforeRanges.push({
        start: range.start,
        end: start,
        style: range.style,
      });
    }

    if (range.start >= end) {
      afterRanges.push({
        start: range.start + delta,
        end: range.end + delta,
        style: range.style,
      });
    } else if (range.end > end) {
      afterRanges.push({
        start: end + delta,
        end: range.end + delta,
        style: range.style,
      });
    }
  });

  const insertedRanges =
    // Deletions contribute no replacement span, but insertions need a fresh styled segment.
    replacement.text.length === 0
      ? []
      : [
          {
            start,
            end: start + replacement.text.length,
            style: replacement.style,
          },
        ];

  return {
    text: node.text.slice(0, start) + replacement.text + node.text.slice(end),
    ranges: mergeAdjacentRanges([
      ...beforeRanges,
      ...insertedRanges,
      ...afterRanges,
    ]),
  };
}
```

## Walkthrough

For `abcdefghij`, replacing `[1, 8)` with `XYZ` removes seven characters and inserts three, so `delta` is `-4`.

```javascript
[
  { start: 0, end: 2, style: 'body' },
  { start: 2, end: 5, style: 'code' },
  { start: 5, end: 10, style: 'quote' },
];
```

The first range contributes a before fragment `[0, 1)` as `body`. The replacement contributes `[1, 4)` using `replacement.style`. The last range has a suffix after `end`, so `[8, 10)` shifts left by four positions to `[4, 6)`. Those fragments cover the new text `aXYZij` exactly.

## Common pitfalls

- **Shifting by the replacement length alone:** Later ranges move by the net `delta`, not just by `replacement.text.length`. Deletions shift ranges left, insertions shift ranges right, and equal-length replacements do not shift later ranges.
- **Emitting a range for pure deletion:** When `replacement.text === ''`, there is no replacement span to cover. The replacement style is ignored because no inserted text exists.
- **Dropping surviving prefixes or suffixes:** The edit can start or end in the middle of an existing range. Prefixes before `start` and suffixes after `end` must survive, with suffixes shifted by `delta`.
- **Forgetting to merge after the splice:** Inserted text can match the left neighbor, the right neighbor, or both. The final normalization pass keeps the representation compact.

### Leaving gaps or overlaps

The final ranges should still cover the new text exactly, in order, with no overlaps. This is easiest to check by reasoning in before/replacement/after order.

## Edge cases

- `start === end` is an insertion. It can split an existing range, then the merge pass may reconnect adjacent fragments with the same style.
- `replacement.text === ''` is a deletion. It removes the selected span and shifts later ranges left by the deleted length.
- Equal-length replacement has `delta === 0`, so later ranges keep their offsets even though the covered span is restyled.
- Replacements can merge with the left neighbor, the right neighbor, or both when styles match.
- The input node and its existing range objects should not be mutated.

## Notes

- Inserting at a cursor position can split one range into two before the merge pass reconnects them.
- Deleting text removes the covered fragments and shifts later ranges left.
- Replacing text can merge with the left neighbor, the right neighbor, or both.
- The final ranges should still cover the new text exactly.
- `start === end` acts as an insertion.
- `replacement.text === ''` acts as a deletion.
- The input node and its ranges should remain unchanged.

## Techniques

- Interval splitting
- Offset translation
- Modeling text edits as splice operations

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
An edit shifts later ranges by the number of inserted characters, forgetting to subtract the removed length. Which test best distinguishes that bug while keeping the expected suffix positions simple?
