# Range Checking

## Core Idea

Ensure a value stays within defined lower and upper bounds.

```
value < lower → lower

value > upper → upper

otherwise → value
```

---

## Recognition

Use when a problem involves:

- Minimum / Maximum limits
- Validation
- Numeric boundaries
- Coordinates
- Percentages
- Progress bars
- Pagination

---

## Template

```js
if (value < lower) return lower;

if (value > upper) return upper;

return value;
```

or

```js
return Math.max(lower, Math.min(value, upper));
```

---

## Complexity

- Time: **O(1)**
- Space: **O(1)**

---

## Common Mistakes

- Forgetting bounds are inclusive
- Swapping lower and upper
- Using `<` instead of `<=` when required
- Not handling invalid ranges (`lower > upper`)

---

## Production Considerations

- Validate inputs if ranges can be invalid.
- `Math.max(Math.min())` is concise and idiomatic.
- Explicit `if` statements are often easier to read in business logic.

---

## Questions Using This Pattern

- [[Clamp]]