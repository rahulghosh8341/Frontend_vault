---
title: Implement the Holy Grail layout using just CSS
aliases:
  - Holy Grail
difficulty: Easy
time: 15 min
languages:
  - HTML
  - CSS
companies:
  - "[[Atlassian]]"
  - "[[Dropbox]]"
pattern:
  - "[[Holy Grail Layout]]"
concepts:
  - "[[CSS Flexbox]]"
  - "[[CSS Grid]]"
solved: true
solvedDate: 2026-08-27
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Implement the classic Holy Grail layout (header, footer, left nav, center content, right sidebar) using CSS.

## Problem

Implement the Holy Grail layout with:
- **Header:** horizontal, 60px tall.
- **Left & Right Columns:** fixed width of 100px.
- **Center Column:** fluid width.
- **All columns:** equal height.
- **Footer:** 100px tall, sticks to bottom of viewport even with short content.

## Companies

- [[Atlassian]]
- [[Dropbox]]

## Pattern

- [[Holy Grail Layout]]

## 🤔 Thought Process

- **Two Axes Model:** Outer vertical layout for header, middle container, and footer. Inner horizontal layout for the three columns.
- **Sticky Footer:** `#root` uses `display: flex; flex-direction: column; min-height: 100vh;`. Middle `.columns` uses `flex-grow: 1;`.
- **Equal Height Columns:** Placing all three columns inside a flex container (`.columns`) makes them stretch to equal height automatically.
- **Fixed & Fluid Widths:** Side columns use `flex-shrink: 0; width: 100px;`. Center column (`main`) uses `flex-grow: 1;` to absorb remaining space.

## 💻 Final Solution

### App.js

```jsx
export default function App() {
  return (
    <>
      <header>Header</header>
      <div className="columns">
        <nav>Navigation</nav>
        <main>Main</main>
        <aside>Sidebar</aside>
      </div>
      <footer>Footer</footer>
    </>
  );
}
```

### styles.css

```css
body {
  font-family: sans-serif;
  font-size: 12px;
  font-weight: bold;
  margin: 0;
}

#root {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

* {
  box-sizing: border-box;
}

header,
nav,
main,
aside,
footer {
  padding: 12px;
  text-align: center;
}

header {
  background-color: tomato;
  height: 60px;
}

.columns {
  display: flex;
  flex-grow: 1;
}

nav {
  background-color: coral;
  flex-shrink: 0;
  width: 100px;
}

main {
  background-color: moccasin;
  flex-grow: 1;
}

aside {
  background-color: sandybrown;
  flex-shrink: 0;
  width: 100px;
}

footer {
  background-color: slategray;
  height: 100px;
}
```

## UI Output

![[holy-grail-output.png]]

## 🤔 Why This Works

- `#root` takes up at least full viewport height (`min-height: 100vh`) and stacks children vertically (`flex-direction: column`).
- `.columns` has `flex-grow: 1`, forcing it to expand and push the footer to the bottom.
- Inside `.columns`, `nav` and `aside` have fixed widths and `flex-shrink: 0` so they don't squish, while `main` has `flex-grow: 1` to fill the remaining fluid space.

## 🐞 Bugs I Made

- None.

## Production Considerations

- Use semantic HTML tags (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) for accessibility.
- Ensure responsive behavior on mobile screens (stack columns vertically via media queries).

## ⭐ Revision Notes

### Key Facts

- Sticky footer pattern: `min-height: 100vh` on flex container + `flex-grow: 1` on middle wrapper.
- Equal height columns come naturally with Flexbox containers.
- Sidebars need `flex-shrink: 0` to prevent unintended compression.

### Common Interview Questions

- How do you make a sticky footer? → Flexbox column layout with `min-height: 100vh` and growing middle element.
- How do you achieve equal-height columns? → Flexbox (default `align-items: stretch`) or CSS Grid.

### Interview Takeaways

- Flexbox simplifies modern layout design compared to legacy floats and negative margins.

### Related

- [[Holy Grail Layout]]
- [[CSS Flexbox]]
- [[CSS Grid]]
