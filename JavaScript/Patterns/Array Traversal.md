---
aliases:
  - Array Traversal
  - Linear Scan
---

> [!info]
> Process every element of an array exactly once from left to right (or right to left), optionally building a result or computing an answer.

---

## Core Idea

Visit each element sequentially while maintaining any required state.

```text
Initialize state

↓

Loop through array

↓

Process current element

↓

Update state

↓

Return answer
```

---

## Recognition

Use this pattern when you need to:

- Visit every element once
- Transform an array
- Filter elements
- Compute a running value
- Validate all elements
- Search for an element
- Count occurrences
- Build a new array
- Aggregate results

---

## Template

```js
for (let i = 0; i < array.length; i++) {
  const current = array[i];

  // Process current element

  // Update state
}

return result;
```

---

## Variations

### Build a New Array

```js
const result = [];

for (let i = 0; i < array.length; i++) {
  result.push(transform(array[i]));
}

return result;
```

Examples:

- `map()`
- `filter()`
- Polyfills

---

### Running Calculation

```js
let answer = initialValue;

for (let i = 0; i < array.length; i++) {
  answer = update(answer, array[i]);
}

return answer;
```

Examples:

- Sum
- Product
- Maximum
- Minimum

---

### Count

```js
let count = 0;

for (let i = 0; i < array.length; i++) {
  if (condition(array[i])) {
    count++;
  }
}
```

Examples:

- Count even numbers
- Count vowels
- Frequency counting

---

### Search

```js
for (let i = 0; i < array.length; i++) {
  if (condition(array[i])) {
    return array[i];
  }
}

return null;
```

Examples:

- find()
- indexOf()
- includes()

---

## Complexity

| Operation | Complexity |
|-----------|------------|
| Time | **O(n)** |
| Extra Space | **O(1)** *(unless building a new array)* |

---

## Common Mistakes

- Off-by-one errors (`i < array.length`)
- Forgetting to return the result
- Modifying the original array unintentionally
- Ignoring sparse array holes in polyfills
- Using `for...in` instead of indexed iteration
- Forgetting to update state

---

## Interview Tips

Ask yourself:

- Am I visiting every element exactly once?
- Am I computing one answer or building a new array?
- Do I need early exit?
- Should I mutate the input?
- Can I reduce memory usage?

---

## Problems Using This Pattern

```dataview
LIST
FROM "JavaScript/Problems"
WHERE contains(pattern, this.file.link)
SORT file.name ASC
```

---

## Related Patterns

- [[Fixed Size Grouping]]
- [[Two Pointers]]
- [[Sliding Window]]
- [[Prefix Sum]]

---

## Related Concepts

- [[Array]]
- [[Time Complexity]]
- [[Space Complexity]]