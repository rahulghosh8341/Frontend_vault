---
title: Template Engine II
aliases:
  - Template Engine II
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/template-engine-ii"
pattern:
  - "[[String Search]]"
  - "[[Object Path Traversal]]"
concepts:
  - "[[String Search]]"
  - "[[Object Path Traversal]]"
  - "[[Security & XSS]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Template Engine II

> [!info] Problem
> Implement a templating engine with nested lookups and escaped or raw output

## Problem

## Template Engine II

This is a follow-up to [Template Engine](/questions/javascript/template-engine).

Implement the same `renderTemplate(template, data)` API, but now with behavior that is closer to a real template renderer:

- Tags can use dotted paths such as `{{user.name}}`.
- Normal `{{...}}` tags should HTML-escape their output.
- Triple-brace tags like `{{{html}}}` should render raw output.
- Missing paths should still render as an empty string.

This question still keeps the engine intentionally small:

- `data` only contains plain objects and primitive values.
- Arrays are out of scope.
- Inputs are guaranteed valid.

## Examples

```javascript
renderTemplate('Hello {{user.name}}!', {
  user: { name: 'Alice' },
});
// 'Hello Alice!'
```

Normal tags should escape HTML.

```javascript
renderTemplate('{{content}}', {
  content: '<strong>Hello</strong>',
});
// '&lt;strong&gt;Hello&lt;/strong&gt;'
```

Triple-brace tags should render raw output.

```javascript
renderTemplate('{{{content}}}', {
  content: '<strong>Hello</strong>',
});
// '<strong>Hello</strong>'
```

## Arguments

`renderTemplate(template, data)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `template` | `string` | The template string containing `{{...}}` or `{{{...}}}` tags. |
| `data` | `Object` | A nested object containing plain objects and primitive values. |

## Returns

Returns a new rendered string.

## Notes

- Missing paths and resolved `null` / `undefined` values should render as an empty string.
- A path that resolves to an object should also render as an empty string.
- `false`, `0`, and `''` are valid resolved values and should still render.
- You do not need to support sections, loops, or malformed templates in this question.

## Hints

### Hint 1 : How far can a dotted lookup continue?

### Hint 2 : Which brace form controls escaping?

## 🤔 Thought Process

- **Immediate Recognition:** Advanced template engine adding **nested dot-notation resolution** and **contextual HTML escaping** (`{{...}}` escaped vs `{{{...}}}` raw).
- **Core Problem:**
  1. Tokenize both triple-brace `{{{path}}}` (raw HTML) and double-brace `{{path}}}` (escaped HTML) tags.
  2. Resolve deep object paths like `user.profile.name`.
  3. Apply HTML entity escaping to `&`, `<`, `>`, `"`, `'` for double-brace tags only.
- **Regex Parsing:**
  - Match triple braces first or capture brace count: `/\{\{\{([^}]+)\}\}\}|\{\{([^}]+)\}\}/g`.
  - Group 1: raw path; Group 2: escaped path.
- **Path Resolution:**
  - Split path by `.` and traverse object.
  - If any intermediate step is `null`, `undefined`, or a primitive while expecting an object, abort and return `""`.
  - If final resolved value is an object or array, spec requires rendering `""`.
  - If resolved value is `null` or `undefined`, render `""`.
  - If resolved value is a primitive (`0`, `false`, string, number), render it.

---

## 🧠 Mental Model

Think of a **Two-Track Pipeline**:
- **Lexer / Pattern Matcher:** Identifies tag mode based on brace depth (Triple = Raw Output; Double = Escaped Output).
- **Resolver:** Navigates the data tree via dot paths (`path.split('.').reduce(...)`).
- **Formatter:** Converts value to string, then branches:
  - If raw track: emit directly.
  - If escaped track: sanitize through an HTML entity translation table (`&` -> `&amp;`, `<` -> `&lt;`, `>` -> `&gt;`, `"` -> `&quot;`, `'` -> `&#39;`).

---

## 🔑 Key Concepts

- HTML entity escaping / Cross-Site Scripting (XSS) defense
- Deep object path traversal / property reduction
- Regex branch matching (`{{{...}}}` vs `{{...}}}`)
- Handling objects as non-renderable terminal values

---

## ⚠️ Edge Cases / Traps

- **Ampersand First in Escaping:** When replacing HTML characters, replace `&` with `&amp;` **first**! If you replace `<` with `&lt;` first, a subsequent `&` replacement will turn `&lt;` into `&amp;lt;` (double-escaping bug).
- **Triple Braces Matched as Double:** A regex matching `\{\{` before `\{\{\{` will match the outer braces of a triple tag and leave an extra brace behind. Match `\{\{\{` before `\{\{`.
- **Objects Resolving as Terminals:** If a path resolves to an object `{ name: 'Alice' }`, do not output `[object Object]`. The spec explicitly requires rendering `''`.
- **Intermediate Nullish Values:** A path `a.b.c` on `{ a: null }` must not throw a `TypeError: Cannot read properties of null`. Safely break and return `''`.

---

## ⭐ Interview Takeaway

1. **Escape Order Invariant:** Always replace `&` first when sanitizing HTML strings to prevent accidental double encoding.
2. **Brace Priority:** In regular expressions, longer fixed delimiters (`{{{`) must take precedence over shorter prefixes (`{{`).
3. **Safe Deep Lookup:** Use optional chaining or a `reduce` loop with nullish short-circuiting for reliable nested property extraction.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why must `&` be escaped before other characters like `<` and `>`?
- How do you prevent a regex from treating `{{{tag}}}` as `{{` followed by `{tag}}}`?
- What should a template engine output if a tag resolves to a nested object?

### Follow-up Questions
- How would you implement Handlebars-style helpers (e.g. `{{formatDate createdAt}}`)?
- How does React JSX prevent XSS by default compared to `dangerouslySetInnerHTML`?
- How would you support conditional rendering blocks like `{{#if user}}...{{/if}}`?

### Conceptual Questions
- What is Cross-Site Scripting (XSS) and how does automatic contextual escaping protect web applications?
- What are the performance trade-offs between compiling a template string to bytecode/AST versus running regex substitution on every render?

---

## 🔄 Variations

- **Template Engine I:** Flat keys only with no HTML escaping.
- **Section / Loop Engine:** Full Mustache engine supporting iterations and inverted sections.
- **DOM Template Parser:** Using `<template>` elements and DocumentFragments instead of string replacements.

---

## 📝 Revision Notes

- **Core idea:** Match `{{{...}}}` (raw) and `{{...}}` (escaped); resolve dotted path; format output.
- **Remember:** Escape `&` before `<`, `>`, `"`, `'`.
- **Watch out for:** Paths resolving to objects or nullish values must render as `''`.
- **Complexity:** Time: $O(T + P)$ where $T$ is template length and $P$ is total path depth traversed; Space: $O(T)$.

## Official Solution
## Template Engine II ( Official solution )

Premium
Languages

## Solution

Part II starts from the single-pass replacement model in part I, but each matched tag now carries two extra decisions: how to resolve nested data and whether the rendered output should be HTML-escaped.

The model is a tiny tokenizer plus a lookup formatter. The regex does not build a full syntax tree because this part has no sections, loops, or malformed input recovery. Each match is independent and can be treated as `{ mode, path }`: choose raw vs escaped mode from the brace count, resolve the dotted path, format the resolved value, and optionally escape the resulting string.

There are three pieces:

1. Find both `{{...}}` and `{{{...}}}` tags in one global regex pass.
2. Resolve the tag path by walking the object one dotted segment at a time.
3. Convert the resolved value to a string, then escape it only for normal double-brace tags.

The regex has one capture group for triple-brace tags and another for double-brace tags. Triple braces land in the raw-output path; double braces land in the escaped-output path. After picking the matched path and trimming it, `resolvePath()` starts at the root data object and follows each segment from `path.split('.')`.

That path walk has a simple condition: after each segment, `current` is the value at the path prefix already consumed. If the next segment is missing or `current` is no longer a non-null object, the path cannot be resolved and the replacement becomes `''`.

`formatValue()` keeps the output rules centralized: missing paths, `null`, `undefined`, and objects render as `''`, while primitive values are converted with `String(...)`. The last step is conditional escaping. Normal `{{...}}` tags escape `&`, `<`, `>`, `"`, and `'`; triple-brace `{{{...}}}` tags return the formatted value as-is.

For the same data value, the brace count decides only the final escaping step:

| Tag | Resolved value | Formatted value | Returned replacement |
| --- | --- | --- | --- |
| `{{user.name}}` | `'Alice'` | `'Alice'` | escaped `'Alice'` |
| `{{content}}` | `'<strong>Hello</strong>'` | `'<strong>Hello</strong>'` | `&lt;strong&gt;Hello&lt;/strong&gt;` |
| `{{{content}}}` | `'<strong>Hello</strong>'` | `'<strong>Hello</strong>'` | `<strong>Hello</strong>` |
| `{{missing.path}}` | `undefined` | `''` | `''` |

```jsx
const TOKEN_REGEX = /{{{\s*([^{}]+?)\s*}}}|{{\s*([^{}]+?)\s*}}/g;
const HTML_ESCAPE_MAP = {
  '&': '&amp;',
  '<': '&lt;',
  '>': '&gt;',
  '"': '&quot;',
  "'": '&#39;',
};

/**
 * @typedef {null | undefined | boolean | number | string} TemplatePrimitive
 * @typedef {TemplatePrimitive | TemplateObject} TemplateValue
 * @typedef {{ [key: string]: TemplateValue }} TemplateObject
 */
function resolvePath(path, data) {
  let current = data;

  for (const segment of path.split('.')) {
    if (
      typeof current !== 'object' ||
      current === null ||
      !Object.hasOwn(current, segment)
    ) {
      return undefined;
    }

    current = current[segment];
  }

  return current;
}

function formatValue(value) {
  if (value == null || typeof value === 'object') {
    return '';
  }

  return String(value);
}

function escapeHtml(value) {
  return value.replace(/[&<>"']/g, (character) => HTML_ESCAPE_MAP[character]);
}

/**
 * @param {string} template
 * @param {TemplateObject} data
 * @returns {string}
 */
export default function renderTemplate(template, data) {
  return template.replace(TOKEN_REGEX, (_, rawPath, escapedPath) => {
    // Triple braces land in `rawPath`; double braces land in `escapedPath`.
    const path = (rawPath ?? escapedPath ?? '').trim();
    const renderedValue = formatValue(resolvePath(path, data));

    return rawPath != null ? renderedValue : escapeHtml(renderedValue);
  });
}
```

## Common pitfalls

- **Escaping before converting to the supported output shape:** Resolve and format the value first, then escape the resulting string. This keeps missing values and objects as `''` and avoids trying to escape non-string values directly.
- **Letting path lookup continue through primitives:** If a path segment resolves to a primitive, the next segment cannot be read from it. Return `undefined` as soon as the current value is not an object, is `null`, or does not own the next segment.
- **Treating triple braces as a different lookup mode:** Triple braces change only the escaping behavior. They should resolve the same dotted paths and follow the same missing-value rules as normal double braces.
- **Dropping falsy primitives:** `0`, `false`, and `''` are resolved values and should render as strings. Only missing paths, `null`, `undefined`, and objects render as an empty string.

## Notes

- Triple braces change only the escaping behavior, not the path lookup.
- Returning `''` for objects keeps the prompt focused on primitive interpolation.
- Escaping `&`, `<`, `>`, `"`, and `'` covers the common HTML-sensitive characters for this question.
- A regex-based scan is reasonable here because tags are flat. Once sections or nesting are introduced, a parser is easier to justify than adding more capture groups.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
An escape helper replaces `<` with `&lt;`, then replaces every `&` with `&amp;`. Why does rendering `<` produce the wrong result? Explain why the solution's single replacement pass avoids this problem.

Your notes (optional)
