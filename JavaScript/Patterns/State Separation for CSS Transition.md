---
aliases:
  - State Separation for CSS Transition
---

## Core Idea

When animating DOM properties with CSS transitions dynamically created or appended via JavaScript, the browser requires an initial state to be calculated and rendered in the paint pipeline before updating to the target state. If properties are modified synchronously in the same frame as element creation, the browser collapses the style recalculations into a single paint pass, skipping the visual animation.

Using `requestAnimationFrame()` (or forcing reflow via element geometry reading) forces a frame boundary separation:
1. Append element with initial CSS properties (e.g., `width: 0%`).
2. Schedule state change in `requestAnimationFrame()` to update properties (e.g., `width: 100%`) before next frame paint.
3. CSS `transition` smoothly interpolates between initial and final values over specified duration.

## Recognition

Use this pattern when:
- Creating elements dynamically via DOM manipulation or state addition in UI components.
- Needing CSS transitions or keyframe animations to start immediately upon element mounting.
- Animating progress bars, dynamic notifications/toast popups, dropdown disclosures, or modal entrances without manual JS frame-by-frame loops.

## Template

### Vanilla JavaScript

```javascript
// 1. Create and set initial un-animated state
const element = document.createElement('div');
element.classList.add('animatable-element');
container.appendChild(element);

// 2. Separate rendering phase via requestAnimationFrame
requestAnimationFrame(() => {
  // 3. Set target state to trigger CSS transition
  element.style.width = '100%';
});
```

### CSS

```css
.animatable-element {
  width: 0%;
  transition: width 2000ms linear;
}
```

### React Component

```jsx
function AnimatedItem() {
  const [active, setActive] = useState(false);

  useEffect(() => {
    const timer = requestAnimationFrame(() => {
      setActive(true);
    });
    return () => cancelAnimationFrame(timer);
  }, []);

  return <div className={`item ${active ? 'active' : ''}`} />;
}
```

## Variations

1. **`requestAnimationFrame` Double Frame:** For browsers with aggressive paint batching, wrapping in nested `requestAnimationFrame(() => requestAnimationFrame(...))` ensures double-frame layout calculation safety.
2. **Forced Synchronous Layout / Reflow (Legacy Trick):** Reading `element.offsetWidth` or `element.getBoundingClientRect()` forces immediate layout flush synchronously before setting target style (`element.offsetWidth; element.style.width = '100%'`), though `requestAnimationFrame` is preferred as non-blocking.

## Complexity

- **Time Complexity:** $O(1)$ setup time per element; animation execution runs at hardware 60/120 FPS offloaded to GPU/compositor.
- **Space Complexity:** $O(1)$ auxiliary JS heap overhead per DOM node added.

## Common Mistakes

- **Synchronous Style Mutation:** Setting `width = '0%'` then `width = '100%'` in same synchronous JS block results in element immediately rendering at 100% without animation.
- **Using `setTimeout(fn, 0)` instead of `requestAnimationFrame`:** `setTimeout` runs via macro-task queue which is not aligned with screen repaint lifecycle and may cause visual stutter or skip frames.
- **Forgetting CSS `transition` property:** Expecting JS state change to animate automatically without CSS transition rules.

## Interview Tips

- Explain *why* the browser ignores back-to-back style changes: style calculation and paint batching optimize DOM rendering into single paint passes.
- Contrast `requestAnimationFrame` (aligned with browser repaint rate, usually 60Hz/120Hz) vs `setTimeout` (timer queue, inaccurate for visual rendering).
- Mention hardware acceleration: animating CSS properties like `transform` (scale, translate) or `opacity` avoids layout recalculations compared to `width`.

## Problems Using This Pattern

- [[Progress Bar]]

## Related Patterns

- [[HTML Form Submission]]

## Related Concepts

- [[Event Loop]]
- [[CSS Transitions]]
- [[DOM Manipulation]]
