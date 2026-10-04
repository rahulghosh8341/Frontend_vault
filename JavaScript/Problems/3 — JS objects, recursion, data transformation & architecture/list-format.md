---
title: "List Format"
aliases:
  - "listFormat"
  - "List Format"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# List Format

> [!info] Problem
> Implement a function that formats a list of items into a single readable string

## Problem

## List Format

Yangshun Tay
Ex-Meta Staff Engineer
Given a list of strings, implement a function `listFormat` that returns the items concatenated into a single string. A common use case is summarizing reactions on social media posts.

The function should support a few options as the second parameter:

- `sorted`: Sorts the items alphabetically.
- `length`: Shows only the first `length` items, using "and X other(s)" for the remaining. Ignore invalid values (negative, 0, etc.).
- `unique`: Removes duplicate items.

## Examples

```javascript
listFormat([]); // ''

listFormat(['Bob']); // 'Bob'
listFormat(['Bob', 'Alice']); // 'Bob and Alice'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John']);
// 'Bob, Ben, Tim, Jane and John'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 4,
}); // 'Bob, Ben, Tim, Jane and 1 other'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
  sorted: true,
}); // 'Ben, Bob, Jane and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John', 'Bob'], {
  length: 3,
  unique: true,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', 'Tim', 'Jane', 'John'], {
  length: 3,
  unique: true,
}); // 'Bob, Ben, Tim and 2 others'

listFormat(['Bob', 'Ben', '', '', 'John']); // 'Bob, Ben and John'
```

## Hints

### Hint 1 : Which list does `length` apply to?

### Hint 2 : What belongs after the final “and”?

## Asked at these companies

- [[Dropbox]]

## 🤔 Thought Process

- **Immediate Recognition:** String formatting and aggregation utility mimicking the browser's native `Intl.ListFormat`.
- **Formatting Patterns by Item Count:**
  - 0 items: `""` (empty string).
  - 1 item: `"A"`
  - 2 items: `"A and B"`
  - 3+ items: `"A, B and C"` (oxford comma rules or standard conjunction).
- **Options Handling:**
  - `unique`: Deduplicate items while preserving order (e.g. `Array.from(new Set(items))`).
  - `sorted`: Sort items alphabetically (`.sort()`).
  - `length`: Truncate to first `length` items and append `"and X other(s)"`:
    - E.g. 4 items with `length: 2` -> `"A, B and 2 others"`.
    - Handle singular vs plural: `other` (1 remaining) vs `others` (>1 remaining).
    - Ignore invalid lengths (negative, 0, non-integer).
- **Order of Option Operations:**
  1. Filter out empty/falsy strings if required.
  2. If `unique`: Deduplicate items.
  3. If `sorted`: Sort items.
  4. If `length` is valid and `length < items.length`: Truncate and construct suffix `"and ${remainder} other(s)"`.
  5. Format remaining items with commas and conjunction.

---

## 🧠 Mental Model

Think of a **Social Feed Reaction Formatter**:
```
Raw Reactions: ['Alice', 'Bob', 'Alice', 'Charlie', 'David']
        │
      unique   --> ['Alice', 'Bob', 'Charlie', 'David']
        │
      sorted   --> ['Alice', 'Bob', 'Charlie', 'David']
        │
    length: 2  --> ['Alice', 'Bob'] + "2 others"
        │
      format   --> "Alice, Bob and 2 others"
```

---

## 🔑 Key Concepts

- [[Array Traversal]]
- [[Set Lookup]]
- String concatenation & comma formatting
- `Intl.ListFormat` browser API
- Pluralization rules (`other` vs `others`)

---

## ⚠️ Edge Cases / Traps

- **Singular vs Plural Suffix:** If exactly 1 item remains after truncation, format as `"and 1 other"`, NOT `"and 1 others"`.
- **Invalid `length` Values:** If `length <= 0`, negative, or larger than array length, ignore `length` option completely.
- **Empty Array:** Must return an empty string `""`.
- **Single Element After Truncation:** If `items = ['A', 'B']` and `length: 1`, output is `"A and 1 other"`.

---

## ⭐ Interview Takeaway

- Use `new Set(items)` for `unique: true`.
- Normalize pipeline steps: Deduplicate -> Sort -> Truncate -> Stringify.
- Always mention `Intl.ListFormat` in frontend interviews as the production-ready standard internationalization API.

---

## 🎯 Common Interview Questions

### Direct Questions
- "What native browser API replaces custom list formatting functions?" (`Intl.ListFormat`).
- "How do you handle singular vs plural suffixes cleanly?" (Ternary: `count === 1 ? 'other' : 'others'`).

### Follow-up Questions
- "How would you support different conjunctions like 'or' ('A, B or C')?"
- "How would you internationalize this for languages that do not use commas or 'and'?" (Rely on `Intl.ListFormat(locale, { style: 'long', type: 'conjunction' })`).

### Conceptual Questions
- "What is the difference between `Intl.ListFormat` and `Array.prototype.join()`?" (`join` applies uniform separators; `Intl.ListFormat` applies grammatical conjunctions and locale-aware separators).

---

## 🔄 Variations

- **Relative Time Format:** Formatting timestamps into human-readable strings (`Intl.RelativeTimeFormat`).
- **Plural Rules:** Localized plural categorization (`Intl.PluralRules`).

---

## 📝 Revision Notes

- Reference implementation:
```javascript
export default function listFormat(items, options = {}) {
  if (!items || items.length === 0) return '';

  let list = items.filter(item => item && item.trim() !== '');
  if (list.length === 0) return '';

  if (options.unique) {
    list = Array.from(new Set(list));
  }

  if (options.sorted) {
    list.sort();
  }

  const len = options.length;
  if (len && typeof len === 'number' && len > 0 && len < list.length) {
    const remaining = list.length - len;
    const suffix = `${remaining} other${remaining > 1 ? 's' : ''}`;
    list = list.slice(0, len);
    list.push(suffix);
  }

  if (list.length === 1) return list[0];
  if (list.length === 2) return `${list[0]} and ${list[1]}`;

  return `${list.slice(0, -1).join(', ')} and ${list[list.length - 1]}`;
}
```

---

## Official Solution

## List Format ( Official solution )

Yangshun Tay
Ex-Meta Staff Engineer
Languages
The exercise mirrors the [`Intl.ListFormat.prototype.format()` API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat/format), which assists with language-specific list formatting.

## Solution

This is easiest with data cleanup separated from string rendering.

1. Normalize the input items according to the options.
2. Format the normalized items into the final sentence.

The first phase handles things like removing empty values, sorting, and deduplicating. The important detail is to finish all of that before applying the `length` option, because truncation should operate on the already-normalized list.

The second phase is mostly a small branching problem:

- The `length` option can split the array into a visible prefix and a hidden remainder.
- The visible prefix is joined with `', '`.
- The tail is either joined with `'and'` or replaced by the `'X other(s)'` text, depending on whether truncation is actually needed.

Option order is part of the behavior. In code, empty strings are removed first, then sorting and uniqueness are applied before truncation:

The `unique` option uses `Set`, so it preserves first-seen order when `sorted` is not enabled. When both options are enabled, sorting happens first, then duplicates are removed from the sorted order.

| Input/options | Normalized list before `length` | Rendered output |
| --- | --- | --- |
| `['Bob', 'Ben', '', 'John']` | `['Bob', 'Ben', 'John']` | `Bob, Ben and John` |
| `{ sorted: true, length: 3 }` | alphabetically sorted list | first 3 names plus hidden count |
| `{ unique: true, length: 3 }` | duplicates removed in first-seen order | first 3 unique names plus hidden count |

For `['Zoe', 'Ada', 'Zoe', 'Lin']` with `{ sorted: true, unique: true, length: 2 }`, the list becomes `['Ada', 'Lin', 'Zoe']` before truncation, so the result is `Ada, Lin and 1 other`. If `length` were applied before sorting and deduplication, the hidden count would describe the wrong normalized list.

```jsx
const SEPARATOR = ', ';
const OTHERS_SEPARATOR = ' and ';
const OTHERS_LABEL = 'other';

/**
 * @param {Array<string>} itemsParam
 * @param {{sorted?: boolean, length?: number, unique?: boolean}} [options]
 * @return {string}
 */
export default function listFormat(itemsParam, options = {}) {
  // Filter falsey values.
  let items = itemsParam.filter((item) => !!item);

  if (!items || items.length === 0) {
    return '';
  }

  // No processing is needed if there's only one item.
  if (items.length === 1) {
    return items[0];
  }

  // Sort values.
  if (options.sorted) {
    items.sort();
  }

  // Remove duplicate values.
  if (options.unique) {
    items = Array.from(new Set(items));
  }

  // After deduping, the list may collapse down to a single item; bail out
  // before assembling separators.
  if (items.length === 1) {
    return items[0];
  }

  // Length is specified and valid.
  if (options.length && options.length > 0 && options.length < items.length) {
    const firstSection = items.slice(0, options.length).join(SEPARATOR);
    const count = items.length - options.length;
    const secondSection = `${count} ${OTHERS_LABEL + (count > 1 ? 's' : '')}`;
    return [firstSection, secondSection].join(OTHERS_SEPARATOR);
  }

  // Case where length is not specified.
  const firstSection = items.slice(0, items.length - 1).join(SEPARATOR);
  const secondSection = items[items.length - 1];
  return [firstSection, secondSection].join(OTHERS_SEPARATOR);
}
```

## Edge cases

- Empty strings are removed before any formatting, so all-empty input returns `''`.
- A one-item normalized list returns that item without separators.
- Invalid `length` values such as `0`, negative numbers, or values greater than the list size are ignored.
- The hidden count is pluralized as `1 other` or `N others`.
- Sorting mutates the local filtered array, not the caller's original array reference.

## Notes

This function is not as flexible as the `Intl.ListFormat.prototype.format()` API because the separators are hard-coded in English. The `Intl` API is meant for internationalization (i18n) and also allows customization of the separators (the comma and the `and`), so separators should not be hard-coded if this function is meant for production use.

A stronger version could allow customization of the list separator and the "others" separator.

## Resources

- [`Intl.ListFormat` MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/ListFormat)

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
What does this call return?

```javascript
listFormat(['Zoe', 'Ada', 'Zoe', '', 'Lin'], {
  sorted: true,
  unique: true,
  length: 2,
});
```
