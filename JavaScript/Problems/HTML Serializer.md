---
title: Serialize an object resembling a DOM tree into a formatted HTML string
aliases:
  - HTML Serializer
difficulty: Medium
time: 20 min
languages:
  - JavaScript
companies:
  - "[[TikTok]]"
pattern:
  - "[[DFS Recursion]]"
concepts:
  - "[[Recursion]]"
  - "[[String Manipulation]]"
---

> [!info]
> **Difficulty:** 🟡 Medium | **Time:** 20 min
> Serialize an object resembling a DOM tree into a formatted HTML string with proper indentation (one tab `\t` per nesting level) and one tag per line.

## Problem

Given an object that resembles a DOM tree, implement a function that serializes the object into a formatted string with proper indentation (one tab (`\t` character) per nesting level) and one tag per line.

### Examples

```javascript
const tree = {
  tag: 'body',
  children: [
    { tag: 'div', children: [{ tag: 'span', children: ['foo', 'bar'] }] },
    { tag: 'div', children: ['baz'] },
  ],
};

serializeHTML(tree);
// Output:
`<body>
	<div>
		<span>
			foo
			bar
		</span>
	</div>
	<div>
		baz
	</div>
</body>`;
```

## Companies

- [[TikTok]]

## Pattern

- [[DFS Recursion]]

## 🤔 Thought Process

- **Tree Structure:** The input is a tree. Each node is either a string (leaf) or an object (container).
- **Core Logic:** For every object node:
  1. Output the opening tag at the current level of indentation.
  2. For every child inside `children`:
     - If child is a string, format and append at `level + 1` indentation.
     - If child is an object, recursively serialize it at `level + 1`.
  3. Output the closing tag at the current level of indentation.
- **Indentation Tracking:** Pass a depth/level parameter (`level`) through recursive calls to dynamically generate indentation using `'\t'.repeat(level)`.
- **Formatting:** Append `\n` after tags and text elements to ensure exactly one element/tag per line. Finally, remove the trailing newline using `.trimEnd()`.

## 💻 Final Solution

```javascript
export default function serializeHTML(root) {
  function serialize(node, level) {
    let result = '';

    // Opening tag
    result += '\t'.repeat(level) + `<${node.tag}>\n`;

    // Process children
    for (let child of node.children) {
      if (typeof child === 'string') {
        result += '\t'.repeat(level + 1) + child + '\n';
      } else {
        result += serialize(child, level + 1);
      }
    }

    // Closing tag
    result += '\t'.repeat(level) + `</${node.tag}>\n`;

    return result;
  }

  return serialize(root, 0).trimEnd();
}
```

## 🤔 Why This Works

- **Base Case Processing:** Strings don't trigger recursion; they are formatted directly using `level + 1` indentation and appended.
- **Recursive Branching:** Object children are passed to `serialize(child, level + 1)` which correctly shifts the nesting context down, maintaining logical scope.
- **Accumulation of State:** Intermediate recursive results are returned and concatenated onto the parent `result` string.
- **Separation of Concerns:** Each call handles only its own structural level (its own tag, children wrapper, and closing tag), letting recursion assemble the entire tree left-to-right.

## 🐞 Bugs I Made

### 1. Used `root.tag` inside recursion

You initially had:

```
root.tag
```

Inside `serialize()`, you need:

```
node.tag
```

because `node` represents the **current recursive node**.

---

### 2. Didn't capture the recursive result

You had:

```
serialize(child, level + 1);
```

Correct:

```
result += serialize(child, level + 1);
```

---

### 3. Missing newline after closing tags

You had:

```
`</${node.tag}>`
```

Correct:

```
`</${node.tag}>\n`
```

This prevents nested output from being concatenated together.

## Production Considerations

- **Self-Closing Tags:** The current solution assumes all tags are container tags requiring closing nodes. Production engines must handle void elements/self-closing tags (e.g., `<img>`, `<input>`, `<br>`) which do not contain children and close immediately (`<img />`).
- **Data Escaping:** Special characters in string children (e.g., `<`, `>`, `&`, `"`, `'`) should be escaped to prevent cross-site scripting (XSS) or invalid markup generation.
- **Maximum Call Stack Limit:** For extremely deep trees (e.g., thousands of nested divs), direct recursion could trigger `RangeError: Maximum call stack size exceeded`. An iterative tree traversal using a manual stack could be used to mitigate this, though it is rare to see DOM structures that deep in practice.

## ⭐ Revision Notes

### Key Facts

- HTML serialization is an application of Depth-First Search (DFS) Tree Traversal.
- Level indentation is generated dynamically via `'\t'.repeat(level)`.
- Use `.trimEnd()` at the final execution entry to remove the extra trailing newline.

### Common Interview Questions

- **How would you support self-closing/void elements like `<img>` or `<input>`?** → Maintain a Set of void elements (e.g., `const VOID_TAGS = new Set(['img', 'input', 'br', 'hr', 'link', 'meta'])`). If `node.tag` is in this set, render `<tag />\n` immediately without looking at children or appending a closing tag.
- **How do you handle attributes (e.g., `<div id="app" class="container">`)?** → Extract attributes from the node object, map them into `key="value"` pairs, and join them to insert inside the opening tag: `<tag attr1="val1" attr2="val2">`.

### Interview Takeaways

- DFS Recursion is clean and easy to read. Carry parent state (like indentation/formatting levels) down through function arguments.
- Always check for primitive base cases (like string children) vs compound structural nodes.

### Related

- [[DFS Recursion]]
- [[Flatten]]
