---
title: Styled Text Ranges IV
aliases:
  - Styled Text Ranges IV
difficulty: Hard
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/styled-text-ranges-iv"
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

# Styled Text Ranges IV

> [!info] Problem
> Implement a function that patches shallow style objects over a text range

## Problem

## Styled Text Ranges IV

This is a follow-up to [Styled Text Ranges III](/questions/javascript/styled-text-ranges-iii).

Earlier questions used `style` as an opaque token. This follow-up makes the style itself slightly richer while still keeping the data model interview-friendly.

Implement `patchTextRangeStyle(node, start, end, patch)`.

In this question, each range has a shallow style object:

```javascript
{
  bold?: boolean,
  italic?: boolean,
  color?: string,
}
```

## Examples

```javascript
patchTextRangeStyle(
  {
    text: 'Hello world',
    ranges: [
      { start: 0, end: 6, style: { color: 'black' } },
      { start: 6, end: 11, style: { italic: true, color: 'black' } },
    ],
  },
  0,
  11,
  { bold: true },
);
// {
//   text: 'Hello world',
//   ranges: [
//     { start: 0, end: 6, style: { bold: true, color: 'black' } },
//     { start: 6, end: 11, style: { bold: true, italic: true, color: 'black' } },
//   ],
// }
```

```javascript
patchTextRangeStyle(
  {
    text: 'Hello',
    ranges: [{ start: 0, end: 5, style: { color: 'black' } }],
  },
  2,
  2,
  { bold: true },
);
// {
//   text: 'Hello',
//   ranges: [{ start: 0, end: 5, style: { color: 'black' } }],
// }
```

## Arguments

`patchTextRangeStyle(node, start, end, patch)`

- `node` (`object`): A styled text object containing `text` and normalized `ranges`.
- `start` (`number`): Inclusive patch start.
- `end` (`number`): Exclusive patch end.
- `patch` (`object`): A shallow style patch whose keys are a subset of `bold`, `italic`, and `color`.

## Returns

Returns a new styled text object with the patched style object applied over the selected range.

## Notes

- Inputs are guaranteed valid.
- Empty selections should be a no-op.
- Unspecified style fields should be preserved.
- Removing style keys is out of scope.
- Merge adjacent ranges in the result when their final style objects are equal by value.
- Do not mutate the input.

## Hints

### Hint 1 : Which fragment receives the patch?

### Hint 2 : Are equal styles the same object?

### Hint 3 : Where can style objects still alias the input?

## Asked at these companies

Figma

## 🤔 Thought Process

- **Immediate Recognition:** Multi-property styling / partial style patch over text intervals (e.g. toggling Bold without erasing existing Italic or Color).
- **Core Problem:** Instead of setting `style: 'bold'`, each range has a style object `{ bold?: boolean, italic?: boolean, color?: string }`. A `patch` like `{ bold: true }` must be shallow-merged with existing styles over `[start, end)`.
- **Interval Splitting with Merged Styles:**
  - For intervals overlapping `[start, end)`:
    - Slice before selection: retains original `r.style`.
    - Slice inside selection: receives `{ ...r.style, ...patch }`.
    - Slice after selection: retains original `r.style`.
  - Non-overlapping intervals remain unchanged.
- **Value-Equality Normalization:**
  - In parts II and III, style equality was primitive string comparison (`a.style === b.style`).
  - Here, style objects must be compared **by value** (e.g. `{ bold: true, color: 'black' }` vs `{ bold: true, color: 'black' }`).
  - Implement a shallow object comparator `isSameStyle(a, b)` comparing sorted keys and values.
  - Merge adjacent ranges if `prev.end === curr.start && isSameStyle(prev.style, curr.style)`.

---

## 🧠 Mental Model

Think of **Layering Color & Style Filters on Transparency Sheets**:
- An existing text range might already have an *Italic* sheet and a *Blue* sheet.
- You apply a new *Bold* filter across index `0..11`.
- The text under the filter retains its existing *Italic* and *Blue* properties while adding *Bold*.
- **Merging:** If two adjacent text segments end up with the identical set of active style filters (same bold, italic, and color values), weld them into one continuous segment.

---

## 🔑 Key Concepts

- Object attribute merging (`{ ...r.style, ...patch }`)
- Structural/Value equality comparison for shallow dictionaries
- Attribute span partitioning in rich text models (Figma text properties)
- Range normalization based on object equality

---

## ⚠️ Edge Cases / Traps

- **Object Identity vs Value Equality:** Checking `prev.style === curr.style` using reference equality will fail to merge newly patched intervals because `{ bold: true } !== { bold: true }`. A value equality check is mandatory.
- **Empty Selection (`start === end`):** Must immediately no-op and return a shallow clone of the input node without splitting.
- **Preserving Unspecified Properties:** A patch `{ bold: true }` applied to `{ italic: true, color: 'red' }` must preserve both `italic` and `color`: `{ bold: true, italic: true, color: 'red' }`.
- **Undefined / Missing Keys:** `{ bold: true }` and `{ bold: true, italic: undefined }` should be treated consistently (spec states patch keys are a subset of `bold`, `italic`, `color`).
- **Adjacent Merge after Patch:** A patch might make a modified slice identical to its neighbor that wasn't modified (or vice-versa), requiring a merge.

---

## ⭐ Interview Takeaway

1. **Custom Equality for Normalization:** Whenever range attributes are objects rather than primitives, write a targeted equality helper (`isEqualStyle(a, b)`) to guide the adjacent merger loop.
2. **Three-Way Interval Decomposition:** Slicing logic remains identical to Part II; only the payload assignment changes from assignment to object spread (`{ ...r.style, ...patch }`).
3. **Real-World Relevance:** This is exactly how rich text toolbars (e.g., Bold, Italic, Color picker buttons) work in Figma, Notion, and Google Docs.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why can't reference equality (`===`) be used to merge adjacent ranges in this question?
- How do you implement value equality for shallow style objects?
- What happens if the patch applies to a range that already has the exact same style properties?

### Follow-up Questions
- How would you handle removing a style property (e.g. un-bolding `{ bold: false }` or deleting the `bold` key)?
- How would you scale this if style objects contained nested objects (like typography font metrics or shadows)?
- How would you implement an `isSelectionBold(start, end)` query that returns `true`, `false`, or `mixed`?

### Conceptual Questions
- What is the difference between inline formatting models (HTML DOM) and run-based linear models in text engines?
- Why do modern text editors avoid storing DOM nodes as the source of truth for rich text?

---

## 🔄 Variations

- **Styled Text Ranges II:** Scalar string styles with overwrite semantics.
- **Styled Text Ranges III:** Text editing and character replacement with style run tracking.
- **Style Querying (`queryFormat`):** Inspecting whether a selection has uniform or mixed styles.

---

## 📝 Revision Notes

- **Core idea:** Slice at `start` and `end`; apply `{ ...style, ...patch }` to intersecting slices; merge adjacent ranges by style value equality.
- **Remember:** Write a shallow object equality checker for `prev.style` vs `curr.style`.
- **Watch out for:** Empty selections (`start === end`) are no-ops; do not mutate original node.
- **Complexity:** Time: $O(R \times K)$ where $R$ is ranges count and $K$ is number of style keys (constant $\le 3$); Space: $O(R)$.

## Official Solution
## Styled Text Ranges IV ( Official solution )

Premium
Languages

## Solution

This final follow-up reuses the split-and-merge pattern from part II, but the style comparison becomes more interesting because styles are now shallow objects instead of opaque strings. The change from part III is that the text no longer changes; only selected style fields are patched.

The selection still behaves like an open/close window over the ordered range list. Outside the window, preserve the original style object values. Inside the window, shallow-merge the patch over the existing style so unspecified fields remain in place.

The rule is: every output range is either an unchanged fragment from outside `[start, end)`, or an overlapping fragment whose style equals `{ ...originalStyle, ...patch }`. After that, a normalization pass merges adjacent fragments when their final style values match.

The replacement pass has five jobs:

1. Return cloned ranges unchanged for empty selections.
2. Keep non-overlapping ranges in order.
3. Split overlapping ranges into prefix, patched middle, and suffix.
4. Shallow-merge the patch into only the middle fragment.
5. Merge adjacent ranges whose final style objects match by value.

The most important helper here is style equality. Two ranges should merge when their `bold`, `italic`, and `color` values are the same, even if their style objects are different references.

```jsx
/**
 * @typedef {{
 *   bold?: boolean,
 *   italic?: boolean,
 *   color?: string,
 * }} TextStyle
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
 *
 * @typedef {Partial<TextStyle>} TextStylePatch
 */
function cloneStyle(style) {
  return { ...style };
}

function cloneRange(range) {
  return {
    start: range.start,
    end: range.end,
    style: cloneStyle(range.style),
  };
}

function patchStyle(style, patch) {
  return {
    ...style,
    ...patch,
  };
}

function stylesEqual(a, b) {
  return a.bold === b.bold && a.italic === b.italic && a.color === b.color;
}

function mergeAdjacentRanges(ranges) {
  return ranges.reduce((mergedRanges, range) => {
    const previousRange = mergedRanges[mergedRanges.length - 1];

    // Adjacent ranges merge by style value, not by object identity.
    if (
      previousRange != null &&
      previousRange.end === range.start &&
      stylesEqual(previousRange.style, range.style)
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
 * @param {TextStylePatch} patch
 * @returns {StyledText}
 */
export default function patchTextRangeStyle(node, start, end, patch) {
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

    if (range.start < start) {
      nextRanges.push({
        start: range.start,
        end: start,
        style: cloneStyle(range.style),
      });
    }

    // Only the overlapping middle gets patched; the outer fragments keep their original style.
    nextRanges.push({
      start: Math.max(range.start, start),
      end: Math.min(range.end, end),
      style: patchStyle(range.style, patch),
    });

    if (range.end > end) {
      nextRanges.push({
        start: end,
        end: range.end,
        style: cloneStyle(range.style),
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

Given adjacent ranges:

```javascript
[
  { start: 0, end: 2, style: { bold: true, color: 'red' } },
  { start: 2, end: 4, style: { bold: true } },
];
```

Patching `[2, 4)` with `{ color: 'red' }` produces `{ bold: true, color: 'red' }` for the second range. The two style objects are not the same object, but their supported fields now match by value, so the merge pass collapses them into one `[0, 4)` range.

## Common pitfalls

- **Replacing the whole style object:** The patch is shallow and additive for this exercise. Unspecified fields from the original style, such as `italic` or `color`, should remain on the patched middle fragment.
- **Comparing style objects by reference:** Adjacent ranges should merge when their `bold`, `italic`, and `color` values are equal. Object identity is not enough because equivalent styles may be stored in different objects.
- **Patching prefix and suffix fragments:** Only the overlap with `[start, end)` receives the patch. Prefix and suffix fragments keep cloned copies of their original styles.
- **Mutating nested style objects:** The input node and its nested style objects should remain unchanged. Clone style objects when preserving or returning ranges.
- **Treating missing keys as removals:** Removing style keys is out of scope. A missing key in `patch` means "leave the existing value alone", not "delete this property".

## Notes

- Patching one field should preserve the others.
- Patching across several ranges can cause neighbors to become equal and merge.
- Empty selections should return the original content unchanged.
- The input node and its nested style objects should remain unchanged.
- Merge adjacent ranges by style value, not by object identity.
- The text itself does not change in this part.

## Techniques

- Shallow object merging
- Value-based equality
- Reusing a normalization pass after updates

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A patcher applies a field only when `if (patch[key])` is true. Which patch demonstrates a valid requested update that this guard silently discards?
