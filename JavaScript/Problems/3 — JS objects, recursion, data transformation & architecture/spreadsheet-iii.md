---
title: "Spreadsheet III"
aliases:
  - "SpreadsheetIII"
  - "Spreadsheet III"
difficulty: "Hard"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Spreadsheet III

> [!info] Problem
> Implement a spreadsheet class with dependency-aware updates and cycle detection

## Problem

## Spreadsheet III

This is a follow-up to [Spreadsheet II](/questions/javascript/spreadsheet-ii).

Keep the same `Spreadsheet` API and the same formula syntax from the previous question, but now formulas may contain cycles.

Add support for:

- direct cycles
- indirect cycles
- self-references

If a cell is part of a cycle, `getCell(cellId)` should return `'#CYCLE!'`.

If a cell depends on another cell that is part of a cycle, it should also return `'#CYCLE!'`.

## Examples

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', '=A1 + 1');

sheet.getCell('A1'); // '#CYCLE!'
```

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', '=B1 + 1');
sheet.setCell('B1', '=A1 + 1');
sheet.setCell('C1', '=B1 + 2');

sheet.getCell('C1'); // '#CYCLE!'
```

Overwriting a cell should update the dependency graph too.

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', '=B1 + 1');
sheet.setCell('B1', '=A1 + 1');

sheet.getCell('A1'); // '#CYCLE!'

sheet.setCell('B1', 3);

sheet.getCell('A1'); // 4
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

Returns the evaluated numeric value of `cellId`, `0` for unset cells, or `'#CYCLE!'` when `cellId` depends on a cycle.

## Notes

- Keep the same left-to-right evaluation rules from [Spreadsheet II](/questions/javascript/spreadsheet-ii).
- Spaces inside formulas should be ignored.
- Formula operands are either cell references or unsigned numeric literals.
- You do not need ranges, functions, parentheses, precedence, or invalid-formula handling in this question.

## Hints

### Hint 1 : Is a previously visited cell necessarily cyclic?

### Hint 2 : What should arithmetic do with a cyclic operand?

## Asked at these companies

- [[OpenAI]]
- [[Discord]]

## 🤔 Thought Process

- **Immediate Recognition:** Extending the Spreadsheet formula engine with cycle detection (direct cycles, indirect cycles, self-references).
- **Core Requirements:**
  - Formula syntax and operators are identical to Spreadsheet II.
  - Cycle detection:
    - Direct cycle: `A1 = =A1 + 1` (self-reference).
    - Indirect cycle: `A1 = =B1 + 1`, `B1 = =A1 + 1`.
    - Downstream dependency: `C1 = =B1 + 2` where `B1` is cyclic.
  - If a cell is part of a cycle OR depends on a cell with a cycle, `getCell(id)` must return `'#CYCLE!'`.
  - Overwriting a cell must update the dependency graph (cycles can be resolved or introduced dynamically).
- **Cycle Detection Algorithm: DFS with Visiting Stack:**
  - Maintain a `Set` representing cells currently on the active recursion call stack (`visiting`).
  - When entering `evaluate(cellId)`:
    - If `cellId` is already in `visiting`: **Cycle detected!** Return `'#CYCLE!'`.
    - Add `cellId` to `visiting`.
    - Evaluate operands: If any dependent cell returns `'#CYCLE!'`, propagate `'#CYCLE!'`.
    - Remove `cellId` from `visiting` upon return (backtracking).

---

## 🧠 Mental Model

Think of **Graph Cycle Detection in a Directed Graph**:
```
Call Stack Path:
  getCell('A1') -> visiting.add('A1')
    └─► getCell('B1') -> visiting.add('B1')
          └─► getCell('A1') -> visiting.has('A1') === TRUE!
              ──► CYCLE DETECTED! Return '#CYCLE!'
```
Any back-edge pointing to an ancestor currently on the call stack represents a cycle.

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Set Lookup]]
- [[Recursion]]
- Graph Cycle Detection (Tarjan / 3-color DFS)
- Active call stack tracking (Backtracking)
- Error propagation across dependency chains

---

## ⚠️ Edge Cases / Traps

- **Diamond Dependencies (Not Cycles):** `A1` depends on `B1` and `C1`; both `B1` and `C1` depend on `D1`. This is a valid DAG (Directed Acyclic Graph), NOT a cycle!
  - Trap: If you use a permanent `visited` set without removing upon return, `D1` will falsely trigger a cycle on the second path.
  - Fix: Add to `visiting` on entry, and always remove in a `finally` block on exit.
- **Cascading Cycle Propagation:** If `A1` and `B1` form a cycle, any independent cell `Z1 = =A1 + 5` must also return `'#CYCLE!'`.
- **Self-Reference:** `A1 = =A1 + 1`. Handled naturally when `A1` looks up its own cell ID.
- **Cell Overwrite:** If `B1` is reassigned to a literal `10`, the cycle between `A1` and `B1` is broken, and subsequent `getCell('A1')` calls should evaluate normally.

---

## ⭐ Interview Takeaway

- Track ancestors with `visitingSet = new Set()`.
- Add before recursing, delete in `finally`:
  ```javascript
  if (visiting.has(cellId)) return '#CYCLE!';
  visiting.add(cellId);
  try {
    // evaluate formula
  } finally {
    visiting.delete(cellId);
  }
  ```
- Propagate `'#CYCLE!'` upward immediately:
  `if (val === '#CYCLE!') return '#CYCLE!';`

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why does a diamond dependency structure falsely trigger cycle detection if using a single visited set?" (A diamond reaches the same shared node via two distinct paths; cycle detection must only track active ancestors currently in the recursion stack).
- "How does Google Sheets represent cyclic dependencies in cells?" (Displays `#REF!` with a tooltip indicating circular dependency).

### Follow-up Questions
- "How would you optimize `getCell` if multiple cells depend on the same formula without recomputing?" (Memoize computed values; invalidate memo cache when `setCell` is called).
- "How would you find all cells involved in a cycle to highlight them in a UI?" (Tarjan's strongly connected components algorithm).

### Conceptual Questions
- "What is topological sorting, and why can it only be applied to DAGs?" (Topological sort orders nodes such that every dependency appears before its dependent; cycles create impossible ordering constraints).

---

## 🔄 Variations

- **Spreadsheet I & II:** Acyclic versions.
- **Course Schedule (LeetCode 207):** Graph cycle detection.
- **Deep Clone II:** Cycle detection with `WeakMap`.

---

## 📝 Revision Notes

- Complete cycle-safe evaluator:
```javascript
export default class Spreadsheet {
  constructor() {
    this.cells = new Map();
  }

  setCell(cellId, value) {
    this.cells.set(cellId, value);
  }

  getCell(cellId, visiting = new Set()) {
    if (!this.cells.has(cellId)) return 0;

    const raw = this.cells.get(cellId);
    if (typeof raw === 'number') return raw;

    if (typeof raw === 'string' && raw.startsWith('=')) {
      if (visiting.has(cellId)) {
        return '#CYCLE!';
      }

      visiting.add(cellId);
      try {
        const tokens = raw.slice(1).trim().match(/([A-Z0-9]+|[\+\-\*\/])/g);
        if (!tokens) return 0;

        const resolve = (token) => {
          if (!isNaN(Number(token))) return Number(token);
          return this.getCell(token, visiting);
        };

        let acc = resolve(tokens[0]);
        if (acc === '#CYCLE!') return '#CYCLE!';

        for (let i = 1; i < tokens.length; i += 2) {
          const op = tokens[i];
          const nextVal = resolve(tokens[i + 1]);
          if (nextVal === '#CYCLE!') return '#CYCLE!';

          if (op === '+') acc += nextVal;
          else if (op === '-') acc -= nextVal;
          else if (op === '*') acc *= nextVal;
          else if (op === '/') acc /= nextVal;
        }

        return acc;
      } finally {
        visiting.delete(cellId);
      }
    }

    return Number(raw) || 0;
  }
}
```

---

## Official Solution

## Spreadsheet III ( Official solution )

Premium
Languages
The big change is not parsing formulas. It is safely evaluating formulas when the dependency graph may contain cycles.

## Solution

Evaluate depth-first over the graph implied by the current formula strings. A persistent dependency graph is unnecessary for the interview-scoped version because the current graph is already implicit in the stored formulas:

- `setCell()` just overwrites the raw cell input.
- `getCell()` follows the references in the current formulas.
- If a formula is replaced, future reads naturally stop following the old references.

The evaluation lifecycle extends part 2 with one extra piece of state:

1. `getCell()` starts evaluation with a fresh `visiting` set.
2. `evaluateCell()` returns `0` for unset cells and numbers directly for literal cells.
3. Before evaluating a formula cell, add its ID to `visiting`.
4. Resolve operands recursively, applying operators left-to-right like part 2.
5. Remove the cell ID from `visiting` after its formula has finished.

The `visiting` set represents the current recursion stack:

- If evaluation reaches a cell that is already in `visiting`, a cycle was found, so return `'#CYCLE!'`.
- If a referenced cell returns `'#CYCLE!'`, propagate that value upward.
- Otherwise resolve the operands and apply operators left-to-right like part 2.

That `visiting` set is the main correctness rule. A cell ID may appear at most once in the current evaluation path. Seeing it again means the current formula chain looped back to itself.

For example, with `A1 = B1 + 1` and `B1 = A1 + 1`, reading `A1` starts with `visiting = {A1}`. Resolving `B1` adds `B1`, and resolving `A1` again detects that `A1` is already in the active stack. The evaluator returns `'#CYCLE!'` and bubbles that value back through the pending formula evaluations.

```jsx
const CELL_REFERENCE_REGEX = /^[A-Z]+[1-9][0-9]*$/;
const TOKEN_REGEX = /([A-Z]+[1-9][0-9]*|\d+(?:\.\d+)?|[+\-*/])/g;
const CYCLE_ERROR = '#CYCLE!';

/**
 * @typedef {number | string} CellInput
 * @typedef {number | '#CYCLE!'} CellValue
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
   * @returns {CellValue}
   */
  getCell(cellId) {
    return this.evaluateCell(cellId, new Set());
  }

  evaluateCell(cellId, visiting) {
    // `visiting` is the current DFS stack, so seeing the same cell again means a cycle.
    if (visiting.has(cellId)) {
      return CYCLE_ERROR;
    }

    const input = this.cells.get(cellId);

    if (input === undefined) {
      return 0;
    }

    if (typeof input === 'number') {
      return input;
    }

    // Keep the cell on the active stack only while its formula is being resolved.
    visiting.add(cellId);
    const value = this.evaluateFormula(input, visiting);
    visiting.delete(cellId);
    return value;
  }

  evaluateFormula(formula, visiting) {
    const expression = formula.replace(/\s+/g, '').slice(1);
    const tokens = expression.match(TOKEN_REGEX);

    let result = this.resolveOperand(tokens[0], visiting);

    if (result === CYCLE_ERROR) {
      return result;
    }

    for (let index = 1; index < tokens.length; index += 2) {
      const nextValue = this.resolveOperand(tokens[index + 1], visiting);

      if (nextValue === CYCLE_ERROR) {
        return nextValue;
      }

      result = this.applyOperator(result, tokens[index], nextValue);
    }

    return result;
  }

  resolveOperand(operand, visiting) {
    if (CELL_REFERENCE_REGEX.test(operand)) {
      return this.evaluateCell(operand, visiting);
    }

    return Number(operand);
  }

  applyOperator(left, operator, right) {
    switch (operator) {
      case '+':
        return left + right;
      case '-':
        return left - right;
      case '*':
        return left * right;
      default:
        return left / right;
    }
  }
}
```

## Common pitfalls

- Use a per-read `visiting` set, not one shared across all calls. It should represent only the active DFS path for the current `getCell()`.
- Remove a cell from `visiting` after evaluating its formula. Otherwise a later independent branch can be mistaken for a cycle.
- Propagate `'#CYCLE!'` immediately instead of applying arithmetic to it.
- Keep the part 2 left-to-right operator behavior. Cycle detection should not change expression semantics.
- Unset cells still evaluate to `0`; they are not cycle errors.

## Notes

- Overwrites work naturally because the graph is derived from the current formula strings, not stored separately.
- Memoization or explicit dependency bookkeeping can be added later, but they are not necessary for the core interview problem here.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A1 contains `'=A1 + 1'`. Which result must B1 produce for `'=0 * A1'`, and why?
