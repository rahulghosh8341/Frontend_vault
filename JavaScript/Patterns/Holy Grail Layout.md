---
aliases:
  - Holy Grail Layout
---

## Core Idea

Structuring a classic page layout (header, footer, left nav, center content, right sidebar) using modern CSS Flexbox or Grid, ensuring sticky footer and equal-height columns.

## Recognition

- Problem asks for a 3-column layout with header and footer.
- Footer must stick to bottom even on short content (`min-height: 100vh`).
- Columns must share equal height.

## Template

```css
#root {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}
.columns {
  display: flex;
  flex-grow: 1;
}
nav, aside {
  flex-shrink: 0;
  width: 100px;
}
main {
  flex-grow: 1;
}
```

## Variations

- Flexbox layout (as shown above)
- CSS Grid layout (`grid-template-rows: auto 1fr auto;`)

## Complexity

- **Time:** $O(1)$ CSS layout rendering.
- **Space:** $O(1)$ DOM elements.

## Common Mistakes

- Forgetting `min-height: 100vh` on the root container, causing footer to ride up when content is short.
- Omitting `flex-shrink: 0` on fixed-width side columns, causing them to collapse when viewport shrinks.

## Interview Tips

- Discuss historical approaches (floats, negative margins) vs modern Flexbox / CSS Grid.
- Emphasize accessibility and semantic HTML (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`).

## Problems Using This Pattern

- [[Holy Grail]]

## Related Patterns

- None

## Related Concepts

- [[CSS Flexbox]]
- [[CSS Grid]]
