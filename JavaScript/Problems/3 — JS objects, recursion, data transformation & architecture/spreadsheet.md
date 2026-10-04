---
title: "Spreadsheet"
aliases:
  - "Spreadsheet"
  - "spreadsheet"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Spreadsheet

> [!info] Problem
> Implement a spreadsheet class with numeric cells and addition formulas

## Problem

## Spreadsheet

Spreadsheets let each cell hold either a literal value or a formula that references other cells. In this question, implement a small `Spreadsheet` class with that behavior.

This question is intentionally limited:

- Cells only hold numbers or formula strings.
- Formula strings start with `=` and only use `+`.
- Referenced cells may themselves contain formulas.
- Inputs are guaranteed acyclic.

## Examples

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 10);
sheet.setCell('B1', '=A1 + 5');

sheet.getCell('B1'); // 15
```

Unset cells should behave like `0`.

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', '=B1 + 3');

sheet.getCell('A1'); // 3
```

Referenced cells can contain formulas too.

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 2);
sheet.setCell('B1', '=A1 + 3');
sheet.setCell('C1', '=B1 + 4');

sheet.getCell('C1'); // 9
```

## Spreadsheet API

### new Spreadsheet()

Creates a `Spreadsheet` instance with no cells.

### spreadsheet.setCell(cellId, input)

Stores a value for `cellId`.

| Parameter | Type | Description |
| --- | --- | --- |
| `cellId` | `string` | A cell reference such as `A1` or `B12`. |
| `input` | `number \| string` | Either a number or a formula string beginning with `=`. |

### spreadsheet.getCell(cellId)

Returns the evaluated numeric value of `cellId`.

If `cellId` has not been set, return `0`.

## Notes

- Cell references use uppercase A1-style labels such as `A1` and `B12`.
- Spaces inside formulas should be ignored.
- Formula operands are either cell references or unsigned numeric literals.
- You do not need subtraction, multiplication, division, cycles, ranges, or invalid-formula handling in this question.

## Hints

### Hint 1 : What should a cell store?

### Hint 2 : Is each operand a value or a reference?

## Asked at these companies

- [[OpenAI]]
- [[Discord]]

## 🤔 Thought Process

- **Immediate Recognition:** Modeling spreadsheet cell formulas with recursive dependency resolution (addition only, guaranteed acyclic in Part I).
- **Core Requirements:**
  - `setCell(cellId, value)`: Stores literal number or formula string (e.g. `10` or `'=A1 + 5 + B2'`).
  - `getCell(cellId)`: Evaluates and returns numeric value of cell.
  - Formula rules:
    - Starts with `_=`.
    - Operands can be numeric literals (`5`) or cell identifiers (`A1`, `B2`).
    - Only operator is `+`.
    - If cell is empty / unset, default to `0`.
- **Evaluation Strategy:**
  - Store raw values in a `Map<cellId, value>`.
  - When `getCell(cellId)` is called:
    - If literal number, return it.
    - If string starting with `_=`:
      - Strip `_=`.
      - Split by `+` to get tokens.
      - For each token (trimmed):
        - If numeric string (`/^\d+$/`), parse as number.
        - Otherwise, treat as cell reference and recursively call `this.getCell(token)`.
      - Sum the resolved token values and return total.

---

## 🧠 Mental Model

Think of a **Directed Acyclic Graph (DAG) Expression Tree**:
```
Cell C1: "=A1 + B1"
          │
     ┌────┴────┐
     ▼         ▼
  Cell A1   Cell B1
   (10)     ("=A2 + 5")
                │
                ▼
             Cell A2 (20)
```
Evaluating a cell recursively traverses down dependent DAG nodes and bubbles summed values up.

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Recursion]]
- [[String Search]]
- DAG Dependency Resolution
- Formula Lexing / Tokenization
- In-memory Cell Coordinate Mapping (`Map<string, any>`)

---

## ⚠️ Edge Cases / Traps

- **Unset / Empty Cells:** Referencing a cell that has not been set (e.g. `C9`) should evaluate to `0`, not `NaN` or throw.
- **Whitespace in Formulas:** Formulas like `'= A1 + 10 + B2 '` must have tokens trimmed before inspection.
- **Cell ID vs Numeric Literal:** Token `'100'` is a literal number; `'A1'` is a cell reference. Check `Number.isNaN(Number(token))` or regex.
- **Formulas Referencing Other Formulas:** Cell A depends on B, which depends on C. Depth-first recursion evaluates C, then B, then A seamlessly.

---

## ⭐ Interview Takeaway

- Store raw formulas/literals directly in `this.cells = new Map()`.
- Evaluate lazily on `getCell(id)` using DFS recursion.
- Tokenize by splitting on `+`:
  `const tokens = formula.slice(1).split('+').map(t => t.trim());`
- Return `0` for unset cells: `if (!this.cells.has(id)) return 0;`

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you distinguish cell references from numeric literals in a formula?" (Attempt `Number(token)` or test against cell identifier regex `/^[A-Z]+\d+$/`).
- "What happens if a cell is evaluated multiple times in different formulas?" (In Part I, re-evaluated via DFS; in production, memoization or reactive dependency tracking is used).

### Follow-up Questions
- "How would you handle multiple operators with left-to-right evaluation?" (Answered in Spreadsheet II).
- "What happens if cells contain circular dependencies (`A1 = B1 + 1`, `B1 = A1 + 1`)?" (Answered in Spreadsheet III).

### Conceptual Questions
- "How does Excel or Google Sheets manage cell updates under the hood?" (Maintains a directed dependency graph; modifying a cell triggers topological sort to recompute only affected dependents).

---

## 🔄 Variations

- **Spreadsheet II:** Supporting `+`, `-`, `*`, `/` left-to-right.
- **Spreadsheet III:** Cycle detection (`#CYCLE!`).
- **Reactive Formula Engine:** Event-driven dirty propagation (MobX/Vue reactivity model).

---

## 📝 Revision Notes

- Clean implementation:
```javascript
export default class Spreadsheet {
  constructor() {
    this.cells = new Map();
  }

  setCell(cellId, value) {
    this.cells.set(cellId, value);
  }

  getCell(cellId) {
    if (!this.cells.has(cellId)) return 0;

    const val = this.cells.get(cellId);
    if (typeof val === 'number') return val;

    if (typeof val === 'string' && val.startsWith('=')) {
      const tokens = val.slice(1).split('+').map(t => t.trim());
      let sum = 0;
      for (const token of tokens) {
        if (!isNaN(Number(token))) {
          sum += Number(token);
        } else {
          sum += this.getCell(token);
        }
      }
      return sum;
    }

    return Number(val) || 0;
  }
}
```

---

## Official Solution

## Spreadsheet ( Official solution )

Premium
Languages
This question works well with lazy evaluation: store each cell's raw input, then resolve formulas only when a value is requested. The trap is caching formula outputs without invalidation; a later `setCell()` can change every dependent read. The spreadsheet is a small interpreter where `setCell()` records source text and `getCell()` turns that source into a number.

## Solution

The key state-model decision is to store raw cell contents, not cached computed values. Use a `Map` from cell ID to raw input so that each cell can hold either a number or its formula string.

Treat the map as source code for the sheet, not a cache of computed answers. `getCell()` is responsible for interpreting that source at read time.

The lifecycle is:

1. `setCell(cellId, input)` overwrites the raw value for one cell.
2. `getCell(cellId)` starts evaluation from that cell.
3. `evaluateCell()` handles the three cell states: unset, numeric literal, or formula.
4. `evaluateFormula()` parses the formula body and resolves each operand.

Because formulas stay unevaluated until `getCell()` runs, overwriting `A1` automatically affects any formula that references `A1` the next time it is read. No dependency bookkeeping is needed yet.

For a cell read, the tasks are deliberately small:

- Return `0` if the cell has never been set.
- Return the number directly for literal cells.
- For formula cells, strip whitespace, remove the leading `=`, split on `+`, and resolve each operand.

Resolving an operand is straightforward: if it looks like a cell reference, recursively evaluate that cell. Otherwise treat it as a numeric literal and convert it to a number. Since this question guarantees valid inputs and acyclic formulas, a simple recursive evaluator is enough.

For example, if `A1` is `2`, `B1` is `=A1 + 3`, and `C1` is `=B1 + A1`, reading `C1` recursively evaluates `B1`, then `A1`, and sums the resolved numeric values. If `A1` is later changed to `10`, `C1` reads as `23` without updating `B1` or `C1` manually.

That read can be traced as:

| Expression | Resolution |
| --- | --- |
| `C1 = B1 + A1` | `evaluate(B1) + evaluate(A1)` |
| `B1 = A1 + 3` | `2 + 3 = 5` |
| `A1` | `2` |
| `C1` | `5 + 2 = 7` |

The tradeoff is laziness over a dependency graph. Eager recomputation can be faster for repeated reads, but it forces dependency tracking and invalidation; for this prompt, recursive reads keep the code smaller and correct after overwrites.

```jsx
const CELL_REFERENCE_REGEX = /^[A-Z]+[1-9][0-9]*$/;

/**
 * @typedef {number | string} CellInput
 */

export default class Spreadsheet {
  constructor() {
    this.cells = new Map();
  }

  /**
   * @param {string} cellId
   * @param {CellInput} input
   * @returns {void}
   */
  setCell(cellId, input) {
    this.cells.set(cellId, input);
  }

  /**
   * @param {string} cellId
   * @returns {number}
   */
  getCell(cellId) {
    return this.evaluateCell(cellId);
  }

  evaluateCell(cellId) {
    const input = this.cells.get(cellId);

    if (input === undefined) {
      return 0;
    }

    if (typeof input === 'number') {
      return input;
    }

    return this.evaluateFormula(input);
  }

  evaluateFormula(formula) {
    const expression = formula.replace(/\s+/g, '').slice(1);

    return expression
      .split('+')
      .reduce((total, operand) => total + this.resolveOperand(operand), 0);
  }

  resolveOperand(operand) {
    if (CELL_REFERENCE_REGEX.test(operand)) {
      return this.evaluateCell(operand);
    }

    return Number(operand);
  }
}
```

## Common pitfalls

- Do not cache formula results in part 1 without also implementing invalidation. Storing raw inputs avoids stale values when referenced cells change.
- Treat unset cells as `0`. This makes formulas like `=A1 + B1` work even when one side is missing.
- Remove whitespace before splitting, otherwise operands such as `' A1 '` will not match the cell-reference regex.
- This parser only needs to handle addition for part 1. Wider operator support belongs in the follow-up.
- Because the prompt guarantees formulas are acyclic, the evaluator does not need a visited stack. Cycle detection would be the first extra state to add if that guarantee were removed.

## Notes

- Since this question guarantees acyclic formulas, plain recursion is enough.
- Treating unset cells as `0` makes formulas like `=A1 + B1` work even when one side is missing.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
What is the value of `result`?

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 2);
sheet.setCell('B1', '=A1 + 3');
sheet.setCell('C1', '=B1 + A1');

const before = sheet.getCell('C1');
sheet.setCell('A1', 10);
const after = sheet.getCell('C1');

const result = [before, after];
```
