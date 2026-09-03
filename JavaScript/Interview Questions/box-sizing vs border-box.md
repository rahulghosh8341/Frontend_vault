---
title: "What does `* { box-sizing: border-box; }` do? What are its advantages?"
aliases:
  - box-sizing vs border-box
tags:
  - css
  - interview
  - box-model
  - box-sizing
solved: true
solvedDate: 2026-08-26
type: quiz
---

> [!info]
> 🟢 Difficulty: Medium
> 📂 Category: CSS Interview Question
> ⏱️ Review Time: ~3 minutes

---

## TL;DR

`* { box-sizing: border-box; }` makes every element on the page use the `border-box` model when calculating its `width` and `height`.

By default, elements use:

```css
box-sizing: content-box;
```

With `border-box`:

- `width` includes content + horizontal padding + horizontal border.
- `height` includes content + vertical padding + vertical border.
- `margin` is not included.

The main advantage is **more predictable and intuitive sizing**, especially when building layouts and grids.

---

## Interview Answer (30–60 sec)

> `* { box-sizing: border-box; }` applies the `border-box` sizing model to every element. By default, CSS uses `content-box`, where the declared width and height only represent the content, so adding padding or borders increases the element's total size. With `border-box`, padding and borders are included within the specified width and height, while margin remains outside. This makes sizing more intuitive and predictable, especially for grids, responsive layouts, and components where padding and borders are added.

---

## Key Takeaways

| Property | `content-box` | `border-box` |
|---|---|---|
| Content | Included | Included |
| Padding | Not included | Included |
| Border | Not included | Included |
| Margin | Not included | Not included |

### Main advantage

```text
content-box
width
  ↓
content only
  ↓
padding + border increase total size


border-box
width
  ↓
content + padding + border
  ↓
total size stays within declared width
```

---

## Visual Model

### `content-box`

```text
width: 100px

┌───────────────────────────────┐
│       padding + border        │
│   ┌───────────────────────┐   │
│   │    100px content      │   │
│   └───────────────────────┘   │
└───────────────────────────────┘

Total width > 100px
```

### `border-box`

```text
width: 100px

┌───────────────────────────┐
│      border + padding     │
│   ┌───────────────────┐   │
│   │   70px content    │   │
│   └───────────────────┘   │
└───────────────────────────┘

Total width = 100px
```

---

## Common Interview Traps

❌ **"`border-box` includes margin."**

✔ No. Margin is not included in the width or height calculation.

---

❌ **"`border-box` means padding and border are ignored."**

✔ They are included in the declared width and height.

---

❌ **"`content-box` and `border-box` calculate width the same way."**

✔ `content-box` treats the declared width as content width, while `border-box` treats it as the total content + padding + border width.

---

❌ **"`* { box-sizing: border-box; }` changes the margin behavior."**

✔ It changes width/height calculation. Margin remains outside the box.

---

## Common Follow-ups

- What is the CSS box model?
- What is the difference between `content-box` and `border-box`?
- Why is `border-box` preferred for layouts?
- Does `border-box` include margin?
- Why do CSS frameworks use `border-box` globally?
- How does `box-sizing` affect responsive layouts?
- What happens when padding is added to an element with a percentage width?

---

## My Notes

- 

---

# Detailed Reference

`* { box-sizing: border-box; }` is a CSS rule that applies the `box-sizing: border-box` property to every element on a webpage, overriding the default `content-box` model. This changes how the `width` and `height` of elements are calculated, making the box model more predictable and intuitive by including `padding` and `border` in the specified dimensions.

### Understanding the box model

The CSS box model defines how elements are rendered in terms of their dimensions and spacing. Every element is a rectangular box composed of:

  * Content: The actual content (text, images, etc.).
  * Padding: The space between the content and the border.
  * Border: The area surrounding the padding.
  * Margin: The space outside the border, separating the element from others.

The `box-sizing` property determines which parts of the box model contribute to an element’s `width` and `height`.

#### Default behavior: `box-sizing: content-box`

With `content-box` (the default), the `width` and `height` properties only account for the content area. Any `padding` or `border` added to the element increases its total size beyond the specified `width` or `height`. For example:

```css
div {
  box-sizing: content-box;
  width: 100px;
  height: 100px;
  padding: 10px;
  border: 5px solid black;
}
```

  * Content size: 100px × 100px
  * Total size: 100px (content) + 10px (left padding) + 10px (right padding) + 5px (left border) + 5px (right border) = 130px wide. Similarly, 130px tall.

This can lead to unexpected layout issues, as the element’s total size exceeds the specified dimensions.

#### `box-sizing: border-box`

With `border-box`, the `width` and `height` properties include the content, `padding`, and `border`. The content area shrinks to accommodate `padding` and `border` within the specified dimensions. Using the same example:

```css
div {
  box-sizing: border-box;
  width: 100px;
  height: 100px;
  padding: 10px;
  border: 5px solid black;
}
```

  * Total size: 100px × 100px
  * Content size: 100px - 10px (left padding) - 10px (right padding) - 5px (left border) - 5px (right border) = 70px wide. Similarly, 70px tall.

The `margin` is not included in the `width` or `height` calculation for either `box-sizing` value.

#### Comparison Table

Property  | `box-sizing: content-box` (default)  | `box-sizing: border-box`
--- | --- | ---
Content  | Included  | Included
Padding  | Excluded  | Included
Border  | Excluded  | Included
Margin  | Excluded  | Excluded

### Advantages of `box-sizing: border-box`

  1. Intuitive sizing: Designers often think of an element’s size holistically, including its padding and border. `border-box` aligns with this mental model, making it easier to create layouts where elements fit within predefined grid systems or containers.

  2. Simplified layout calculations: With `border-box`, you don’t need to manually subtract padding and border values to calculate the content size. This reduces errors in responsive designs and complex layouts.

  3. Consistency across elements: Applying `* { box-sizing: border-box; }` globally ensures all elements (e.g. divs, inputs, images) behave consistently, preventing unexpected size increases when styling forms, buttons, or other components with padding or borders.

  4. Better integration with CSS frameworks: Popular frameworks like Bootstrap and Tailwind CSS use `border-box` by default to streamline development and ensure predictable layouts.

  5. Easier responsive design: In responsive layouts where percentages or viewport units (e.g. `vw`, `vh`) are used, `border-box` prevents elements from overflowing their containers due to added padding or borders.

  6. Improved form styling: Form elements like `input` and `select` often have browser-specific padding and borders. `border-box` ensures consistent sizing across browsers, making it easier to align form fields in a layout.

### Practical example

Consider a layout with three columns, each intended to be 33.33% wide within a 900px container:

```css
.container {
  width: 900px;
  display: flex;
}

.column {
  width: 33.33%;
  padding: 15px;
  border: 2px solid black;
}
```

  * With `content-box`: Each column's total width becomes 33.33% + 15px (left padding) + 15px (right padding) + 2px (left border) + 2px (right border). This exceeds 33.33%, causing the columns to overflow or wrap unexpectedly.
  * With `border-box`: Each column's total width remains 33.33% (approximately 300px in a 900px container), with padding and borders fitting within that size. The content area adjusts to accommodate these additions.

```css
* {
  box-sizing: border-box;
}
```

This ensures the columns fit perfectly within the container without manual recalculations.

### Additional Considerations

#### Performance implications

Using `* { box-sizing: border-box; }` has negligible performance impact, as it's a simple CSS property applied during rendering. However, the universal selector (`*`) can be slightly less performant on very large DOM trees due to its broad application. For optimization, you can apply `box-sizing: border-box` to specific elements or use a more targeted selector like:

```css
html {
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: inherit;
}
```

This approach sets `border-box` as the default for all elements but allows specific elements to inherit or override it (e.g., revert to `content-box` if needed).

#### Browser support

The `box-sizing` property is universally supported across all modern browsers, including Chrome, Firefox, Safari, Edge, and Internet Explorer 8+. No vendor prefixes are required, making it safe for production use.

#### Edge cases and gotchas

  1. Legacy browser issues: While rare, older browsers like IE6-7 may not support `box-sizing`. If supporting these browsers is necessary, test layouts thoroughly or use fallbacks.
  2. Third-party components: Some third-party libraries or widgets may assume `content-box`. Applying `border-box` globally could break their styling, requiring overrides.
  3. Flexbox and Grid: In modern layouts using Flexbox or CSS Grid, `border-box` simplifies alignment by ensuring elements respect their container's constraints without unexpected overflows.
  4. Percentage-based padding: When padding is defined in percentages, it's calculated relative to the element's width (even for vertical padding). With `border-box`, this can reduce the content area more than expected, so test carefully.

### When to use `content-box`

While `border-box` is generally preferred, `content-box` may be useful in specific scenarios:

  * Precise content sizing: When you need the content area to be exactly the specified `width` or `height`, regardless of padding or borders (e.g. for image galleries or canvas elements).
  * Legacy codebases: If a project was built with `content-box` assumptions, switching to `border-box` globally could break existing layouts.

### Best practices

  1. Apply globally early: Set `* { box-sizing: border-box; }` at the start of your CSS to ensure consistency across your project.
  2. Use resets or Normalize.css: Many CSS resets (e.g. Normalize.css) include `border-box` by default. Verify your reset to avoid conflicts.
  3. Test with dynamic content: Ensure `border-box` works as expected with dynamically sized elements, such as those using `min-width`, `max-width`, or percentage-based dimensions.
  4. Document overrides: If you need to use `content-box` for specific elements, document the reason clearly to avoid confusion for other developers.

### References

  * MDN Web Docs: box-sizing
  * CSS-Tricks: Box Sizing
  * W3C CSS Box Model Specification
  * Can I Use: box-sizing
