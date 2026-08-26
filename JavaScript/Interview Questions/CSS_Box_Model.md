---
title: Explain your understanding of the box model and how you would tell the browser in CSS to render your layout in different box models.
aliases:
  - CSS Box Model
tags:
  - css
  - interview
  - box-model
---

> [!info]
> 🟢 Difficulty: Medium
> 📂 Category: CSS Interview Question
> ⏱️ Review Time: ~5 minutes

---

## TL;DR

The CSS box model describes the rectangular boxes generated for elements and how their dimensions are calculated.

Every box consists of:

```text
Content
   ↓
Padding
   ↓
Border
   ↓
Margin
```

The `box-sizing` property determines how the browser calculates an element's `width` and `height`.

- `content-box` → `width`/`height` apply to the content only.
- `border-box` → `width`/`height` include content, padding, and border.
- Margin is always outside the box.

---

## Interview Answer (30–60 sec)

> The CSS box model describes how an element's dimensions and surrounding space are calculated. Every element has a content area, padding, border, and margin. By default, CSS uses `content-box`, where the declared width and height apply only to the content, so padding and border increase the element's total size. With `border-box`, the declared width and height include the content, padding, and border, which makes layouts easier to reason about. Margins are outside the box and aren't included in either calculation. I generally use `box-sizing: border-box` globally for predictable sizing.

---

## Key Takeaways

| Concept | Meaning |
|---|---|
| Content | Actual content of the element |
| Padding | Space between content and border |
| Border | Boundary around the element |
| Margin | Space outside the border |
| `content-box` | Width/height apply to content |
| `border-box` | Width/height include content + padding + border |

### `content-box`

```text
Declared width
      ↓
   Content
      +
   Padding
      +
   Border
      ↓
Actual space occupied
```

### `border-box`

```text
Declared width
      ↓
Content + Padding + Border
      ↓
Actual space occupied
```

Margin is outside both models.

---

## Visual Model

```text
┌───────────────────────────────┐
│            Margin             │
│  ┌─────────────────────────┐  │
│  │         Border          │  │
│  │  ┌───────────────────┐  │  │
│  │  │      Padding      │  │  │
│  │  │  ┌─────────────┐  │  │  │
│  │  │  │   Content   │  │  │  │
│  │  │  └─────────────┘  │  │  │
│  │  └───────────────────┘  │  │
│  └─────────────────────────┘  │
└───────────────────────────────┘
```

---

## Common Interview Traps

❌ **"Margin is part of the element's width."**

✔ Margin is outside the box and isn't included in the element's width or height.

---

❌ **"`border-box` includes margin."**

✔ `border-box` includes:

```text
content + padding + border
```

but not margin.

---

❌ **"`content-box` includes padding in the declared width."**

✔ With `content-box`, declared width applies only to the content.

---

❌ **"Borders collapse like margins."**

✔ Borders don't collapse or overlap between adjacent elements.

---

❌ **"Margins never collapse."**

✔ Vertical margins between block-level elements can collapse.

---

## Common Follow-ups

- What is the difference between `content-box` and `border-box`?
- Why is `border-box` commonly used?
- Does `border-box` include margin?
- Does padding contribute to an element's width?
- What is margin collapsing?
- Do horizontal margins collapse?
- What happens if `box-sizing` is not specified?
- Why do CSS frameworks commonly use `border-box`?

---

## My Notes

- 

---

# Detailed Reference

The CSS box model describes the rectangular boxes that are generated for elements in the document tree and laid out according to the visual formatting model. Each box has a content area (e.g. text, an image, etc.) and optional surrounding padding, border, and margin areas.

The CSS box model is responsible for calculating:

How much space a block element takes up.
Whether or not borders and/or margins overlap, or collapse.
A box's dimensions.

Box model rules
The dimensions of a block element are calculated by width, height, padding, and border.
If no height is specified, a block element will be as high as the content it contains, plus padding (unless there are floats — see describe floats and how they work).
If no width is specified, a non-floated block element will expand to fit the width of its parent minus the padding, unless it has a max-width property set, in which case it will be no wider than the specified maximum width.
Some block-level elements (e.g. table, figure, and input) have inherent or default width values and may not expand to fill the full width of their parent container.
span is an inline-level element and does not have a default width, so it will not expand to fit.
The height of an element is calculated by the content's height.
The width of an element is calculated by the content's width.
By default (box-sizing: content-box), padding and border are not part of the width and height of an element.
Margins are not part of the box itself
margins are not counted towards the actual size of the box. They affect the total space the box takes up on the page, but only the space outside the box. The box's area stops at the border — it does not extend into the margin.

Extra
Look up the box-sizing property, which affects how the total heights and widths of elements are calculated.

box-sizing: content-box: This is the default value of box-sizing and adheres to the rules above.

For example:


.example {
  box-sizing: content-box;
  width: 100px;
  padding: 10px;
  border: 5px solid black;
}
The actual space taken by the .example element will be 130px wide (100px width + 10px left padding + 10px right padding + 5px left border + 5px right border).

box-sizing: border-box: The width and height will include the content, padding and border (but not the margin). This is a much more intuitive way to think about boxes and hence many CSS frameworks (e.g. Bootstrap, Tailwind, Bulma) set * { box-sizing: border-box; } globally, so that all elements use such a box model by default. See the question on box-sizing: border-box for more information.

For example:


.example {
  box-sizing: border-box;
  width: 100px;
  padding: 10px;
  border: 5px solid black;
}
The element will still take up 100px on the page, but the content area will be 70px wide (100px - 10px left padding - 10px right padding - 5px left border - 5px right border).

Border and margin behavior
Borders do not collapse or overlap with those of adjacent elements. Each element's border is rendered individually.
Margins can collapse, but only vertically and only between block-level elements. Horizontal margins do not collapse. This means that if one block element has a bottom margin and the next has a top margin, only the larger of the two will be used. This behavior is independent of box-sizing and is the default in CSS.

References
- [The box model | MDN](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model)
