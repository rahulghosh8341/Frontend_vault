---
title: Build a Progress Bar component that fills up smoothly when added to the DOM
aliases:
  - Progress Bar
difficulty: Easy
time: 15 min
languages:
  - JavaScript
  - HTML
  - CSS
companies:
  - "[[Uber]]"
pattern:
  - "[[State Separation for CSS Transition]]"
concepts:
  - "[[DOM Manipulation]]"
  - "[[CSS Transitions]]"
  - "[[requestAnimationFrame]]"
  - "[[Event Loop]]"
solved: true
solvedDate: 2026-08-30
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> Build an application where clicking an "Add" button appends progress bars that smoothly animate from 0% to 100% width over 2000ms.

## Problem

Build an app where clicking the "Add" button adds progress bars to the page. The progress bars fill up gradually as soon as they are shown.

### Requirements

- Clicking on the "Add" button adds a progress bar to the page.
- Each progress bar starts filling up smoothly as soon as it's added.
- Each bar takes approximately 2000ms to completely fill up.

### Hints

- **Hint 1:** Give each bar its own lifetime
- **Hint 2:** Start from an observable empty state
- **Hint 3:** Let visual timing stay local

## Companies

- [[Uber]]

## Pattern

- [[State Separation for CSS Transition]]

## 🤔 Thought Process

1. Create a new progress bar element.
2. Give it an initial width of `0%`.
3. Append it to the DOM.
4. Let the browser render the `0%` state.
5. Change its width to `100%`.
6. Let CSS transition smoothly animate the change over 2000ms.

The important idea is to separate the initial state from the final state so the browser can animate between them.

## 💻 Final Solution

### Vanilla JavaScript Implementation

#### `index.html`

```html
<button id="add-btn">Add</button>
<div id="wrapper"></div>
```

#### `styles.css`

```css
.bar-container {
  background-color: #eee;
  border: 1px solid #ccc;
  height: 20px;
  margin-top: 10px;
  width: 100%;
}

.bar {
  background-color: green;
  height: 100%;
  width: 0%;
  transition: width 2000ms linear;
}
```

#### `index.js`

```javascript
const addBtn = document.getElementById('add-btn');
const wrapper = document.getElementById('wrapper');

addBtn.addEventListener('click', () => {
  const container = document.createElement('div');
  container.classList.add('bar-container');

  const progress = document.createElement('div');
  progress.classList.add('bar');
  container.appendChild(progress);

  wrapper.appendChild(container);

  // Schedule target state change before next repaint
  requestAnimationFrame(() => {
    progress.style.width = '100%';
  });
});
```

### React Implementation

```jsx
import { useState, useEffect } from 'react';

function ProgressBar() {
  const [filled, setFilled] = useState(false);

  useEffect(() => {
    // Separate initial render from animated state
    const timer = requestAnimationFrame(() => {
      setFilled(true);
    });
    return () => cancelAnimationFrame(timer);
  }, []);

  return (
    <div className="bar-container">
      <div className={`bar ${filled ? 'filled' : ''}`} />
    </div>
  );
}

export default function App() {
  const [bars, setBars] = useState([]);

  return (
    <div>
      <button onClick={() => setBars((prev) => [...prev, Date.now() + Math.random()])}>
        Add
      </button>
      <div className="bars-container">
        {bars.map((id) => (
          <ProgressBar key={id} />
        ))}
      </div>
    </div>
  );
}
```

## UI Output

![[progress-bar-output.png]]

## 🤔 Why This Works

### Key Concept — `requestAnimationFrame`

`requestAnimationFrame()` schedules a callback to run before the browser's next repaint.

```javascript
container.appendChild(progress);

requestAnimationFrame(() => {
  progress.style.width = '100%';
});
```

This gives the browser an opportunity to render:

```text
width: 0%
    ↓
browser renders
    ↓
width: 100%
    ↓
CSS transition animates
```

### CSS Does the Animation

JavaScript doesn't manually animate the width.

```css
.bar {
  width: 0%;
  transition: width 2000ms linear;
}
```

JavaScript only changes:

```javascript
progress.style.width = '100%';
```

The browser's CSS transition handles:

```text
0% → 100%
     2000ms
```

## 🐞 Bugs I Made

### Mistake / Common Pitfall — `setTimeout` vs `requestAnimationFrame`

- **`setTimeout`:** Run this callback after a specified delay.
- **`requestAnimationFrame`:** Run this callback in sync with the browser's next rendering cycle.

For visual DOM updates and animations, `requestAnimationFrame` is the appropriate tool.

## Production Considerations

- **GPU Acceleration:** Animating `transform: scaleX(...)` with `transform-origin: left` instead of `width` offloads calculation to the GPU and avoids layout shifts / reflows during every animation frame.
- **Accessibility:** Add `role="progressbar"`, `aria-valuenow`, `aria-valuemin="0"`, and `aria-valuemax="100"` to ensure screen readers communicate progress updates properly.
- **Memory & DOM Management:** If hundreds of bars can be added, clean up finished progress bars or virtualize the list to prevent memory leaks and DOM bloat.

## ⭐ Revision Notes

### Key Facts

- Render the initial state first, then change the property you want to animate. CSS transition performs the animation; `requestAnimationFrame` helps separate the two visual states.
- Synchronous style modifications in the same microtask batch skip visual transition states because the browser flushes pending style changes in a single paint pass.
- `requestAnimationFrame` executes immediately prior to the style and layout recalculation pass of the rendering pipeline.

### Common Interview Questions

- **Why does setting `width = '0%'` followed immediately by `width = '100%'` fail to animate?** → Browsers batch DOM mutations. Without a frame repaint boundary, the engine calculates layout only once for `width: 100%`.
- **Why is `requestAnimationFrame` preferred over `setTimeout(fn, 0)` for DOM animations?** → `requestAnimationFrame` is aligned with the display's refresh rate (60Hz/120Hz) and visual lifecycle, whereas `setTimeout` relies on macro-task timing which can skip frames or cause visual jank.

### Interview Takeaways

- Offload visual animation calculation to CSS transitions/animations; use JS only to orchestrate state changes.
- Always separate initial element mounting state from animated state using `requestAnimationFrame`.

### Related

- [[State Separation for CSS Transition]]
- [[DOM Manipulation]]
- [[CSS Transitions]]
- [[Event Loop]]
