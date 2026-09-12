---
title: Implement textSearch(text, query) to bold case-insensitive matches
aliases:
  - Text Search
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[Anthropic]]"
  - "[[Palantir]]"
pattern:
  - "[[String Search]]"
concepts:
  - "[[String Manipulation]]"
  - "[[Case Insensitive]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-06
type: coding
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Find all case-insensitive matches of a query in text and wrap them in `<b>` tags, combining consecutive matches.

## Problem

Implement a function `textSearch(text, query)` that finds all case-insensitive matches with the query string, wrapping the matches in `<b>...</b>` tags.

```js
/**
 * @param {string} text
 * @param {string} query
 * @return {string}
 */
```

**Examples**

```js
textSearch('The Quick Brown Fox Jumps Over The Lazy Dog', 'fox');
// 'The Quick Brown <b>Fox</b> Jumps Over The Lazy Dog'

textSearch('The hardworking Dog overtakes the lazy dog', 'dog');
// 'The hardworking <b>Dog</b> overtakes the lazy <b>dog</b>'
```

**Rules:**
- A character will not match the same query more than once, with letters appearing earlier taking priority.
  ```js
  textSearch('aaa', 'aa'); // '<b>aa</b>a'
  ```
- Consecutive matches should be combined into a single `<b>` tag.
  ```js
  textSearch('aaaa', 'aa'); // '<b>aaaa</b>'
  ```

## Companies

- [[Anthropic]]
- [[Palantir]]

## Pattern

- [[String Search]]

## 🤔 Thought Process

* Scan the string from left to right using an index.
* At each index, check whether `query` matches the substring starting there, case-insensitively.
* When a match is found, mark those character positions as bold and jump ahead by `query.length` to prevent overlapping matches.
* After marking matches, scan the string again to build the final output.
* Open `<b>` when a normal character is followed by a bold region.
* Close `</b>` when a bold character is followed by a normal/non-bold region.

## 💻 Final Solution

```js
export default function textSearch(text, query) {
  if (text.trim() === '' || query.trim() === '') {
    return text;
  }
  const boldChars = Array.from({ length: text.length }, () => 0);

  for (let i = 0; i < text.length; ) {
    let substr = text.slice(i, i + query.length);
    if (substr.toLowerCase() === query.toLowerCase()) {
      boldChars.fill(1, i, i + query.length);
      i += query.length;
    } else {
      i++;
    }
  }

  let result = '';
  for (let i = 0; i < text.length; i++) {
    let char = text[i];

    const openingTag = boldChars[i] === 1 && boldChars[i - 1] !== 1;
    const closingTag = boldChars[i] === 1 && boldChars[i + 1] !== 1;

    if (openingTag) {
      char = '<b>' + char;
    }
    if (closingTag) {
      char = char + '</b>';
    }

    result += char;
  }
  return result;
}
```

## 🤔 Why This Works

Use a `boldChars` array as a **mask**:

```text
text:       a a a a
boldChars:  1 1 1 1
```

Then detect transitions:

```text
0 → 1  = add <b>
1 → 0  = add </b>
```

For:

```text
aaaa + aa
```

both matches produce:

```text
1 1 1 1
```

so there is only one bold region:

```html
<b>aaaa</b>
```

rather than two separate tags.

## 🐞 Bugs I Made

* Splitting by spaces would fail because matches can occur **inside words**.
* You cannot simply replace every match independently because adjacent matches must be combined.
* Overlapping matches must not reuse characters:

  * `aaa` + `aa` → `<b>aa</b>a`
  * not `<b>aaa</b>`.
* Comparison should be case-insensitive, but the **original text/casing must be preserved**.

## Production Considerations

- This is a simplified version of browser find-in-page functionality.
- For production text highlighting, consider: nested HTML, overlapping queries, Unicode normalization, and performance on large texts.
- The mask approach avoids complex string concatenation or regex replacement edge cases.

## ⭐ Revision Notes

### 🔑 Key Facts

* `slice(i, i + query.length)` checks whether the query starts at index `i`.
* Use lowercase versions only for comparison.
* After a match:
  ```js
  i += query.length;
  ```
  prevents overlapping matches.
* `boldChars` separates **finding matches** from **generating HTML**.
* Opening tag condition:
  ```js
  boldChars[i] === 1 && boldChars[i - 1] !== 1
  ```
* Closing tag condition:
  ```js
  boldChars[i] === 1 && boldChars[i + 1] !== 1
  ```

### 🧠 Mental Model

Think of it as **two passes**:

```text
PASS 1
String
  ↓
Find matches
  ↓
[0, 0, 1, 1, 1, 0, 0]


PASS 2
Use the 0/1 mask
  ↓
0 → 1 = <b>
1 → 0 = </b>
  ↓
Final HTML string
```

The key mental model:

> **First mark what should be bold, then convert the marks into tags.**

### Common Interview Questions

- How would you handle overlapping matches differently? → The mask + `i += query.length` naturally gives "first match wins" behavior.
- What if query is empty? → Return original text (handled by the trim check).
- How does this handle Unicode? → `slice`/`toLowerCase` work on UTF-16 code units; for full Unicode support, use `Array.from(text)` or spread operator.

### Interview Takeaways

The clean pattern for this problem is:

**`scan → mark matches → detect bold-region boundaries → build result`**

The `boldChars` mask is particularly useful because it automatically handles **consecutive matches** without needing complicated tag-merging logic.

### Related

- [[String Search]]
- [[Text Search II]]