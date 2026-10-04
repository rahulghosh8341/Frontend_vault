---
title: Template Engine
aliases:
  - Template Engine
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/template-engine"
pattern:
  - "[[String Search]]"
concepts:
  - "[[String Search]]"
  - "[[Regular Expressions]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Template Engine

> [!info] Problem
> Implement a basic templating engine with Mustache-like placeholders

## Problem

## Template Engine

Templating engines turn a string like `Hello {{name}}!` into rendered output by replacing placeholders with data.

In this question, implement `renderTemplate(template, data)`, a simplified Mustache-like renderer.

This question is intentionally small:

- Only simple variable tags such as `{{name}}` and `{{ name }}` need to be supported.
- `data` is a flat object of primitive values.
- Missing keys should render as an empty string.
- Inputs are guaranteed valid.

## Examples

```javascript
renderTemplate('Hello {{name}}!', { name: 'Alice' });
// 'Hello Alice!'

renderTemplate('{{greeting}}, {{name}}!', {
  greeting: 'Hi',
  name: 'Sam',
});
// 'Hi, Sam!'
```

Whitespace inside tags should be ignored.

```javascript
renderTemplate('{{ count }} items left', { count: 3 });
// '3 items left'
```

Missing keys should render as empty strings.

```javascript
renderTemplate('Hello {{name}} {{surname}}!', { name: 'Alice' });
// 'Hello Alice !'
```

## Arguments

`renderTemplate(template, data)` accepts the following arguments:

| Argument | Type | Description |
| --- | --- | --- |
| `template` | `string` | The template string containing `{{...}}` placeholders. |
| `data` | `Object` | A flat object whose values are primitive values. |

## Returns

Returns a new string with all placeholders replaced.

## Notes

- `false`, `0`, and `''` are valid values and should still render.
- Resolved `null` and `undefined` values should render as an empty string.
- You do not need to support nested paths, sections, or malformed templates in this question.

## Hints

### Hint : What does one placeholder resolve to?

## 🤔 Thought Process

- **Immediate Recognition:** String interpolation / micro-template parsing (Mustache / Handlebars style).
- **Core Problem:** Replace `{{key}}` and `{{ key }}` tags with values from a flat dictionary, rendering missing keys as empty strings.
- **Regex Strategy:** Use `template.replace(/\{\{\s*(\w+)\s*\}\}/g, ...)` (or matching `([^{}]+)`).
- **Falsy Values vs Missing:**
  - `0`, `false`, `""` are valid values and must be rendered as `String(val)`.
  - `null` and `undefined` must render as `""`.
  - Missing keys (not present on data) must render as `""`.
- **Clean Guard:** Check `if (key in data && data[key] != null) return String(data[key]); return '';`.

---

## 🧠 Mental Model

Think of **Token Substitution via Regex Callbacks**:
- The template string is static text with `{{...}}` holes.
- JavaScript's `string.replace(regex, replacerFn)` scans the string once.
- For every match, the callback extracts the identifier inside the mustache braces, strips whitespace, inspects `data`, and swaps the token for its string representation.

---

## 🔑 Key Concepts

- String interpolation & regex capturing groups
- Distinction between `null`/`undefined` vs valid falsy values (`0`, `false`)
- `String.prototype.replace` with replacer function
- Object property lookup (`in` operator / `Object.hasOwn`)

---

## ⚠️ Edge Cases / Traps

- **Treating `0` or `false` as Missing:** Using a naive truthiness check `if (data[key])` incorrectly replaces `0` or `false` with `''`. Must explicitly check `data[key] != null`.
- **Whitespace Tolerance:** Tags like `{{ count }}` with arbitrary spaces before or after the identifier must match just like `{{count}}`.
- **Missing Keys:** Keys not present in `data` must render as empty strings `''`, not `'undefined'`.
- **Special Regex Characters in Replacer:** When using `replace(regex, str)`, if the replacement string contains special characters like `$$` or `$&`, passing a string directly can cause substitution bugs. Returning the string from a replacer function `() => str` is immune to this issue.

---

## ⭐ Interview Takeaway

1. **Precision Falsy Handling:** Distinguish between missing (`!(key in data)`), nullish (`val == null`), and falsy primitives (`0`, `false`, `""`).
2. **Regex Callback Immunity:** Always use a function replacer `(match, key) => ...` rather than passing replacement strings directly to avoid regex `$` replacement tokens.
3. **Scope Bounds:** For simple flat interpolation, a regex replacer is optimal ($O(N)$); for nested or conditional templates, an AST/token parser is needed (see Template Engine II).

---

## 🎯 Common Interview Questions

### Direct Questions
- Why is `String(data[key])` preferred over template literal interpolation in the replacer?
- How do you distinguish between a missing key and a key with value `undefined`?
- Why does `template.replace` with a callback avoid `$` character substitution bugs?

### Follow-up Questions
- How would you extend this to support nested paths like `{{ user.profile.name }}`? (See [[Template Engine II]])
- How would you handle HTML sanitization to prevent XSS injection?
- How would you support default fallback values like `{{ name || 'Guest' }}`?

### Conceptual Questions
- What is the difference between string interpolation, template literals, and a true template engine AST?
- Why do modern frameworks like React and Vue compile templates to render functions instead of using string regex replacement?

---

## 🔄 Variations

- **Template Engine II:** Adding nested dot paths (`user.name`), HTML escaping for `{{...}}`, and raw unescaped triple braces `{{{...}}}`.
- **Section / Loop Support:** Mustache-style `{{#items}}...{{/items}}`.
- **ES6 Tagged Template Literals:** Native JS tagged template functions for interpolation.

---

## 📝 Revision Notes

- **Core idea:** Use `template.replace(/\{\{\s*(\w+)\s*\}\}/g, (_, key) => ...)` to interpolate values.
- **Remember:** `0` and `false` must render; `null`, `undefined`, and missing keys render as `''`.
- **Watch out for:** Avoid truthiness checks `if (data[key])`—use `Object.hasOwn(data, key) && data[key] != null`.
- **Complexity:** Time: $O(T)$ where $T$ is template length; Space: $O(T)$ for output string.

## Official Solution
## Template Engine ( Official solution )

Premium
Languages

## Solution

This first question is placeholder interpolation, not full template parsing. The template can be treated as plain text with simple `{{...}}` holes, so the learner-facing default is a single regex pass that finds each tag and replaces it immediately.

The replacement pass has a narrow rule: literal text outside a tag is copied unchanged, and each supported tag independently contributes exactly one output string.

The main responsibility is to preserve the difference between "missing" and "present but falsy":

- A key that is not present in the flat `data` object renders as `''`.
- A present key with `null` or `undefined` also renders as `''`.
- A present primitive value such as `false`, `0`, or `''` is valid and should be converted with `String(...)`.

The global regular expression matches every `{{...}}` tag and captures only the key inside the braces. Trimming the captured key makes `{{name}}` and `{{ name }}` behave the same way. Since this part only supports flat lookup, the captured key can be checked directly on `data` with `Object.hasOwn()`.

That independent-tag model is why a regex is acceptable here. There is no nested syntax, escaping rule, or section state to carry across matches, so a parser would add surface area without protecting additional behavior in this prompt.

For `{{count}}/{{enabled}}/{{missing}}`, the replacement decisions are:

| Tag | Data state | Replacement |
| --- | --- | --- |
| `count` | own key with `0` | `'0'` |
| `enabled` | own key with `false` | `'false'` |
| `missing` | no own key | `''` |

```jsx
const VARIABLE_REGEX = /{{\s*([^{}]+?)\s*}}/g;

/**
 * @typedef {null | undefined | boolean | number | string} TemplateValue
 * @typedef {Record<string, TemplateValue>} TemplateData
 */

/**
 * @param {string} template
 * @param {TemplateData} data
 * @returns {string}
 */
export default function renderTemplate(template, data) {
  return template.replace(VARIABLE_REGEX, (_, key) => {
    if (!Object.hasOwn(data, key)) {
      return '';
    }

    const value = data[key];
    return value == null ? '' : String(value);
  });
}
```

## Common pitfalls

- **Treating falsy values as missing:** Do not use a broad truthiness check such as `if (!data[key])`. That would erase valid values like `0`, `false`, and `''`. Check whether the object owns the key first, then only collapse `null` and `undefined` to an empty string.
- **Looking through the prototype chain:** Use `Object.hasOwn()` instead of `key in data` when deciding whether a tag is available. The renderer should only interpolate values supplied directly in the data object, not inherited properties.
- **Solving more than this part asks for:** Nested paths, sections, escaping, and malformed templates are intentionally out of scope here. A direct regex replacement is enough because every supported tag is independent.

## Notes

- `Object.hasOwn()` helps distinguish a missing key from an inherited property.
- Converting with `String(...)` preserves values such as `false` and `0`.
- Since this question only supports flat lookups, no path parsing is needed yet.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A renderer returns `String(data[key] || '')` for each tag. Which pair of assertions distinguishes the intended nullish rule from this broad fallback?
