---
title: "Spreadsheet II"
aliases:
  - "SpreadsheetII"
  - "Spreadsheet II"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Spreadsheet II

> [!info] Problem
> Implement a spreadsheet class with left-to-right arithmetic formulas

## Problem

## Spreadsheet II

This is a follow-up to [Spreadsheet](/questions/javascript/spreadsheet).

Keep the same `Spreadsheet` API, but now formulas can use more arithmetic operators: `+`, `-`, `*`, `/`.

This question still keeps the formula language intentionally small:

- Evaluate strictly left-to-right.
- Do not apply normal math precedence.
- Do not support parentheses.
- Inputs are still guaranteed acyclic.

## Examples

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 10);
sheet.setCell('B1', '=A1 - 2 * 3');

sheet.getCell('B1'); // 24
```

The expression is evaluated left-to-right, so it behaves like `(10 - 2) * 3`.

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 8);
sheet.setCell('B1', '=A1 / 2 + 3');

sheet.getCell('B1'); // 7
```

Referenced cells can still contain formulas.

```javascript
const sheet = new Spreadsheet();

sheet.setCell('A1', 4);
sheet.setCell('B1', '=A1 + 6');
sheet.setCell('C1', '=B1 * 2 - 3');

sheet.getCell('C1'); // 17
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
- You do not need precedence, parentheses, ranges, cycles, or invalid-formula handling in this question.

## Hints

### Hint 1 : What must tokenization preserve?

### Hint 2 : Which value becomes the next left operand?

## Asked at these companies

- [[OpenAI]]
- [[Discord]]

## 🤔 Thought Process

- **Immediate Recognition:** Extending the Spreadsheet formula evaluator with arithmetic operators `+`, `-`, `*`, `/`, evaluated strictly left-to-right without standard operator precedence and without parentheses.
- **Formula Specification:**
  - Strictly left-to-right evaluation: `1 + 2 * 3` evaluates as `(1 + 2) * 3 = 9` (NOT `7`).
  - No parentheses support required.
  - Operands can be numeric literals or cell references.
- **Lexing / Tokenization Strategy:**
  - Tokenize formula string into a stream of operands and operators.
  - Regex matcher: `tokens = formula.slice(1).match(/([A-Z0-9]+|[\+\-\*\/])/g)` or split by whitespace/operators.
- **Evaluation Loop:**
  - Evaluate first operand (literal or recursive `getCell`).
  - Loop through subsequent operator-operand pairs:
    - `op = tokens[i]`, `operand = resolve(tokens[i+1])`.
    - Apply operation to running accumulator:
      - `+`: `acc += operand`
      - `-`: `acc -= operand`
      - `*`: `acc *= operand`
      - `/`: `acc /= operand`

---

## 🧠 Mental Model

Think of a **Simple Calculator Accumulator**:
```
Formula: "=A1 + 2 * B2 - 4"
Initial: acc = resolve(A1)
Step 1:  acc = acc + 2
Step 2:  acc = acc * resolve(B2)
Step 3:  acc = acc - 4
```
A single pass stream processor where state folds continuously into the left accumulator.

---

## 🔑 Key Concepts

- [[DFS Recursion]]
- [[Array Traversal]]
- [[String Search]]
- Stream Tokenization / Lexer Regex
- Left-to-right accumulator reduction
- Operator dispatch

---

## ⚠️ Edge Cases / Traps

- **Ignoring Math Precedence:** Standard JS `eval()` or precedence rules will produce incorrect results! The problem strictly demands left-to-right execution.
- **Division by Zero:** Division by 0 evaluates to `Infinity` or `-Infinity` in standard JavaScript.
- **Multi-character Cell IDs:** Regex must match arbitrary cell names like `AA12`, not just single letters and digits.
- **Whitespace Tolerance:** Spaces around operators (`= A1 + 5 * 2`) must be handled cleanly by the tokenizer.

---

## ⭐ Interview Takeaway

- Do NOT use `eval()` or `new Function()`: they apply PEMDAS math precedence, which violates the strict left-to-right prompt constraint.
- Tokenize into a flat array of alternating `[operand, operator, operand, operator, ...]`.
- Initialize accumulator with `resolve(tokens[0])`, then loop `i += 2`:
  `acc = applyOp(acc, tokens[i], resolve(tokens[i+1]))`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why can't `eval()` be used for Spreadsheet II?" (Because `eval()` enforces standard mathematical operator precedence, whereas the prompt explicitly requires left-to-right evaluation).
- "How does the tokenizer distinguish operators from cell names?" (Operators are single characters in `['+', '-', '*', '/']`; operands are alphanumeric strings).

### Follow-up Questions
- "How would you add parentheses support `=(A1 + 2) * B2`?" (Use Shunting-Yard algorithm or recursive descent parsing).
- "How do you handle cyclic formulas?" (Answered in Spreadsheet III).

### Conceptual Questions
- "How does a full programming language interpreter parse mathematical expressions?" (Lexer generates tokens; Pratt parser or Shunting-Yard builds an AST respecting precedence and associativity).

---

## 🔄 Variations

- **Spreadsheet I:** Addition-only formulas.
- **Spreadsheet III:** Cycle detection.
- **Calculator / Expression Evaluator (LeetCode 224 / 227):** Parsing expressions with operator precedence and parentheses.

---

## 📝 Revision Notes

- Left-to-right evaluation loop:
```javascript
function evaluateFormula(formula, resolveCell) {
  const tokens = formula.slice(1).trim().match(/([A-Z0-9]+|[\+\-\*\/])/g);
  if (!tokens || tokens.length === 0) return 0;

  const resolve = (token) => {
    return isNaN(Number(token)) ? resolveCell(token) : Number(token);
  };

  let acc = resolve(tokens[0]);

  for (let i = 1; i < tokens.length; i += 2) {
    const op = tokens[i];
    const nextVal = resolve(tokens[i + 1]);

    if (op === '+') acc += nextVal;
    else if (op === '-') acc -= nextVal;
    else if (op === '*') acc *= nextVal;
    else if (op === '/') acc /= nextVal;
  }

  return acc;
}
```

---

## Official Solution

## Spreadsheet II ( Official solution )

Premium
Languages
The storage model stays the same as part 1. The main new idea is parsing a wider formula syntax while still evaluating lazily.

## Solution

The flow is still "store raw cell contents, evaluate on read". Part II only adds a richer formula parser on top of that same lazy evaluation strategy.

Keep the same basic storage model as part 1: a `Map` from cell ID to raw input, plus recursive evaluation for referenced cells. `setCell()` still just overwrites the raw value, and reads always reflect the latest cell contents because formulas are evaluated on demand.

The evaluation lifecycle is now:

1. Read the raw cell input from the `Map`.
2. Return `0` for an unset cell and return numeric literals directly.
3. For formulas, strip whitespace and remove the leading `=`.
4. Tokenize the expression into alternating operands and operators.
5. Resolve the first operand into a number.
6. Walk the remaining tokens two at a time and apply each operator immediately.

Because this question explicitly ignores normal operator precedence, the reduction step should always happen in source order. That means `=2 + 3 * 4` evaluates as `(2 + 3) * 4`, not `2 + (3 * 4)`. The code makes that behavior explicit by keeping one running `result` and applying each next operator/value pair immediately.

Within this question's valid-input constraints, any token that is not a cell reference can be treated as a numeric literal. Cell references recurse back into `evaluateCell()`, so formulas can depend on other formulas without a separate dependency graph.

Trace for `B1 = '=A1 + 3 * 4 - 5'` when `A1` is `2`:

| Step | Token(s) | Running result |
| --- | --- | --- |
| start | `A1` resolves to `2` | `2` |
| apply | `+ 3` | `5` |
| apply | `* 4` | `20` |
| apply | `- 5` | `15` |

The table shows the important behavior: operators are consumed left-to-right from the token stream, so JavaScript's own precedence rules never enter the evaluator.

```jsx
const CELL_REFERENCE_REGEX = /^[A-Z]+[1-9][0-9]*$/;
const TOKEN_REGEX = /([A-Z]+[1-9][0-9]*|\d+(?:\.\d+)?|[+\-*/])/g;

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

  /**
   * @param {string} cellId
   * @returns {number}
   */
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

  /**
   * @param {string} formula
   * @returns {number}
   */
  evaluateFormula(formula) {
    const expression = formula.replace(/\s+/g, '').slice(1);
    const tokens = expression.match(TOKEN_REGEX);

    let result = this.resolveOperand(tokens[0]);

    // Tokens alternate operand/operator, so each step consumes the next operator/value pair.
    for (let index = 1; index < tokens.length; index += 2) {
      result = this.applyOperator(
        result,
        tokens[index],
        this.resolveOperand(tokens[index + 1]),
      );
    }

    return result;
  }

  /**
   * @param {string} operand
   * @returns {number}
   */
  resolveOperand(operand) {
    if (CELL_REFERENCE_REGEX.test(operand)) {
      // Cell references recurse back through `evaluateCell`, while literals parse directly.
      return this.evaluateCell(operand);
    }

    return Number(operand);
  }

  /**
   * @param {number} left
   * @param {string} operator
   * @param {number} right
   * @returns {number}
   */
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

- Do not use JavaScript's `eval()` or normal expression parsing. Evaluation is left-to-right, not standard arithmetic precedence.
- Keep unset cells returning `0`, including when they are referenced from formulas.
- Make sure tokenization preserves operators as separate tokens. The evaluation loop depends on the pattern `operand, operator, operand, ...`.
- The code still assumes valid, acyclic formulas. Cycle handling is introduced in part 3.
- Overwriting a referenced cell should affect later reads because formulas are evaluated lazily.
- Repeated reads should return the same value unless a referenced raw cell changes.

## Notes

- Reusing the same operand resolver from part 1 keeps formulas with chained references working naturally.
- A small `applyOperator()` helper makes the left-to-right evaluation loop easier to read.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A candidate evaluates formulas using normal JavaScript arithmetic precedence. Which formula must be added to tests to distinguish it from this question's left-to-right language?
