---
title: Table of Contents
aliases:
  - Table of Contents
difficulty: Hard
time: 30 min
languages:
  - JavaScript
companies:
  - "[[Google]]"
pattern:
  - "[[DFS Recursion]]"
  - "[[Recursion]]"
concepts:
  - "[[DOM Manipulation]]"
  - "[[Tree Traversal]]"
  - "[[Recursion]]"
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-28
type: coding
---

> [!info]
> **Difficulty:** 🔴 Hard | **Time:** 30 min
> Given a document node, generate an HTML string representing a nested table of contents (`<ul>`/`<li>`) based on heading tags (`<h1>`–`<h6>`) in document order.

## Problem

On websites, heading tags give hierarchy to the page. Given a document node, write a function `tableOfContents(doc)` that generates an HTML string representing a table of contents based on headings in the document.

### Output Rules
- Visit headings in document order.
- Use heading levels (`h1`–`h6`), not DOM container nesting, to determine list structure.
- Render each heading's text content inside an `<li>`.
- Render child headings inside a nested `<ul>`.
- Return an empty string when the document contains no headings.

```js
const doc = new DOMParser().parseFromString(
  `<!DOCTYPE html>
  <body>
    <h1>Heading1</h1>
    <h2>Heading2a</h2>
    <h2>Heading2b</h2>
    <h3>Heading3a</h3>
    <h3>Heading3b</h3>
    <h4>Heading4</h4>
    <h2>Heading2c</h2>
  </body>`,
  'text/html',
);

tableOfContents(doc);
// Result (compact string):
// <ul><li>Heading1<ul><li>Heading2a</li><li>Heading2b<ul><li>Heading3a</li><li>Heading3b<ul><li>Heading4</li></ul></li></ul></li><li>Heading2c</li></ul></li></ul>
```

## Companies

- [[Google]]

## Pattern

- [[DFS Recursion]]
- [[Recursion]]

## 🤔 Thought Process

The problem requires thinking in two separate tree models:
1. **DOM Tree**: Traversed depth-first using `element.children` to locate headings in exact document order.
2. **Heading Hierarchy Tree**: Built using a stack based on heading levels (`h1` to `h6`), independent of container `<div>` nesting.

### Phase 1: Build the Heading Hierarchy Tree
- Initialize a dummy `rootNode` (`{ text: null, children: [] }`) on a `stack`, with `currentLevel = 0`.
- Traverse the DOM tree recursively using `element.children` (ignoring text/comment nodes).
- For every heading tag (`h1` through `h6`):
  - Parse its numeric level: `parseInt(element.tagName[1], 10)`.
  - Maintain stack invariant: pop elements while `level <= currentLevel` to retreat to the appropriate parent node.
  - Append the new heading node `{ text: element.textContent, children: [] }` to the parent at `stack[stack.length - 1]`.
  - Push the new node onto the stack and set `currentLevel = level`.

### Phase 2: Serialize to HTML
- Recursively convert the tree into HTML:
  - Each node maps to `<li>${node.text}${stringifyChildren(node.children)}</li>`.
  - Child arrays map to `<ul>...</ul>` only if non-empty (avoiding empty `<ul></ul>` tags).
- If the root has no children (no headings found), return empty string `""`.

## 💻 Final Solution

```js
function stringify(contents) {
  function stringifyNode(node) {
    return `<li>${node.text}${stringifyChildren(node.children)}</li>`;
  }

  function stringifyChildren(children) {
    return children.length > 0
      ? `<ul>${children.map(stringifyNode).join('')}</ul>`
      : '';
  }

  return stringifyChildren(contents.children);
}

const headingTags = new Set(['h1', 'h2', 'h3', 'h4', 'h5', 'h6']);

/**
 * @param {Document} doc
 * @return {string}
 */
export default function tableOfContents(doc) {
  // A dummy root lets every heading attach through the same stack logic.
  const rootNode = {
    text: null,
    children: [],
  };
  const stack = [rootNode];
  let currentLevel = 0;

  function traverse(element) {
    if (element == null || element.tagName == null) {
      return;
    }

    if (headingTags.has(element.tagName.toLowerCase())) {
      const level = parseInt(element.tagName[1], 10);
      const node = {
        text: element.textContent,
        children: [],
      };

      // Pop back up until the stack top is the parent for this heading level.
      for (let i = level; i < currentLevel + 1; i++) {
        stack.pop();
      }

      stack[stack.length - 1].children.push(node);
      stack.push(node);
      currentLevel = level;
    }

    for (const child of element.children) {
      traverse(child);
    }
  }

  traverse(doc.body);

  return stringify(stack[0]);
}
```

## 🤔 Why This Works

- **Document Order Traversal**: Depth-first recursion using `element.children` guarantees elements are encountered in natural document order, regardless of deep semantic wrapper elements (`<article>`, `<section>`, `<div>`).
- **Stack-Based Hierarchy**:
  - Going deeper (`h1` -> `h2`): `level > currentLevel`, loop condition `level < currentLevel + 1` does not run. Node pushes directly under previous node.
  - Sibling heading (`h2` -> `h2`): loop pops once, attaching under the same parent.
  - Going higher (`h4` -> `h2`): loop pops multiple times, rewinding the stack to find the correct ancestor.
- **Clean Serialization**: Recursive `stringifyChildren` checks `children.length > 0`, ensuring leaf headings do not emit empty nested `<ul></ul>` tags.
- **Empty Document Handling**: If `contents.children` is empty, `stringifyChildren` immediately evaluates to `""`.

## 🐞 Bugs / Pitfalls

- **Using `childNodes` instead of `children`**: `childNodes` includes `#text` and comment nodes, which do not have `tagName` and cause runtime errors.
- **Coupling hierarchy to DOM nesting**: Heading level alone dictates TOC nesting. An `<h2>` inside a `<div>` and an `<h2>` outside are siblings in the TOC.
- **Forgetting stack adjustment on equal/lower heading levels**: Fails to pop ancestors, incorrectly nesting subsequent sibling headings deeper and deeper.
- **Emitting empty `<ul></ul>`**: Not checking `children.length > 0` before wrapping child nodes in `<ul>`.
- **Case Sensitivity of `tagName`**: HTML `tagName` properties return uppercase (e.g. `"H1"`), so lowercase conversion (`toLowerCase()`) is necessary.

## Production Considerations

- Real-world TOC generators generate slugified anchor IDs on headings (`<a href="#heading-slug">`) to make the TOC interactive.
- In documents with non-standard heading jumps (e.g. `<h1>` followed directly by `<h3>`), the stack loop still handles parent selection gracefully without crashing.
- For extremely large DOM trees, `TreeWalker` or iterative DFS avoids recursion call stack limits.

## ⭐ Revision Notes

### Key Facts

- `element.tagName.toLowerCase()` normalizes HTML uppercase tag strings.
- Two-tree decoupling: DOM tree dictates visit order; heading numbers dictate tree structure.
- Dummy root pattern (`stack = [rootNode]`) simplifies stack operations by avoiding empty-stack edge cases.
- Nested HTML lists require strict `<ul><li>...<ul><li>...</li></ul></li></ul>` nesting (the nested `<ul>` belongs inside the parent `<li>`).

### Common Interview Questions

- Why not use `querySelectorAll('h1, h2, h3, h4, h5, h6')`? In modern browsers `querySelectorAll` returns elements in document order; however, recursive DFS demonstrates core tree traversal principles and handles custom scoping or filtering during walk.
- How are skipped heading levels handled? The stack pops down to the highest available parent with a lower level, maintaining valid tree hierarchy.
- Why is the dummy root necessary? Provides a persistent anchor at `stack[0]` so top-level headings attach cleanly without special-case null checks.

### Interview Takeaways

- Deconstruct complex DOM generation into two discrete phases: **Extraction/Structuring** into intermediate tree, then **Serialization**.
- Use monotonic stack patterns when processing hierarchical tokens of variable depth in linear order.

### Related

- [[DFS Recursion]]
- [[Recursion]]
- [[HTML Serializer]]
- [[Rich Text to HTML]]
