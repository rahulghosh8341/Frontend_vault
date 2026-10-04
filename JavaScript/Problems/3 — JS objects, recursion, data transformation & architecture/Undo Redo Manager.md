---
title: Undo / Redo Manager
aliases:
  - Undo Redo Manager
  - Undo / Redo Manager
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/undo-redo-manager"
pattern:
  - "[[Undo-Redo History]]"
concepts:
  - "[[Undo-Redo History]]"
  - "[[Object-Oriented Programming]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Undo / Redo Manager

> [!info] Problem
> Implement a class that manages value history with undo and redo functionality

## Problem

## Undo / Redo Manager

Undo/redo functionality is common in text editors, drawing tools, settings panels, and multi-step workflows. Instead of tying this logic to a UI, implement a reusable `UndoRedoManager` class that stores a history of values and can move backward and forward through that history.

## Examples

```javascript
const manager = new UndoRedoManager(0);

manager.getCurrent(); // 0
manager.canUndo(); // false
manager.canRedo(); // false

manager.set(1);
manager.set(2);
manager.getCurrent(); // 2

manager.undo();
manager.getCurrent(); // 1
manager.canRedo(); // true

manager.redo();
manager.getCurrent(); // 2

manager.undo();
manager.set(10);
manager.getCurrent(); // 10
manager.canRedo(); // false

manager.reset();
manager.getCurrent(); // 0
manager.canUndo(); // false
manager.canRedo(); // false
```

## UndoRedoManager API

Implement the following APIs on the `UndoRedoManager`:

### new UndoRedoManager(initialValue)

Creates an instance of the `UndoRedoManager` class with `initialValue` as the first history entry. History is isolated within each instance.

Values should be stored as-is. You do **not** need to deep clone them.

| Parameter | Type | Description |
| --- | --- | --- |
| `initialValue` | `unknown` | The initial value to seed the history with. |

### manager.getCurrent()

Returns the current value in the history.

### manager.set(value)

Adds `value` as a new current history entry.

If some values had been undone before calling `set`, all redo history should be discarded before appending the new value.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | `unknown` | The new value to append to history. |

### manager.undo()

Moves the current pointer one step backward if possible.

If the manager is already at the earliest history entry, this method should do nothing.

### manager.redo()

Moves the current pointer one step forward if possible.

If the manager is already at the latest history entry, this method should do nothing.

### manager.reset()

Restores the manager to its original `initialValue` and clears the rest of the history, including any redo history.

### manager.canUndo()

Returns `true` if there is an earlier history entry to move to, otherwise `false`.

### manager.canRedo()

Returns `true` if there is a later history entry to move to, otherwise `false`.

## Notes

- Calling `set()` with the same value as the current value should still create a new history step.
- You do not need to implement history inspection APIs such as `getHistory()` or `getIndex()`.

## Hints

### Hint 1 : Where is the current value in history?

### Hint 2 : What happens to an abandoned future?

## Asked at these companies

Anthropic
Figma
Databricks
Snowflake
Canva
Palantir

## 🤔 Thought Process

- **Immediate Recognition:** Fundamental single-value timeline manager supporting `getCurrent`, `set`, `undo`, `redo`, `reset`, `canUndo`, `canRedo`.
- **Core Problem:** Storing an ordered sequence of arbitrary values with a movable current pointer, truncating redo history on new `set` calls.
- **Design Comparison:**
  - *Option 1: Array + Index pointer (`_history = [initialValue]`, `_index = 0`)*. Easiest, most direct.
  - *Option 2: Past and Future Stacks (`_past = []`, `_current`, `_future = []`)*. Explicit stack pops/pushes.
- **Specification Rules:**
  - Values stored as-is (no deep cloning required by spec).
  - `reset()` restores the original `initialValue` and clears all history.
  - `canUndo()` is `index > 0`; `canRedo()` is `index < history.length - 1`.
  - `set(value)` drops redo entries: `_history.length = _index + 1`, pushes `value`, increments `_index`.

---

## 🧠 Mental Model

Think of a **Browser Back / Forward navigation bar**:
- Every URL you visit adds an entry to your history.
- Clicking "Back" moves your pointer left without deleting forward entries.
- Clicking "Forward" moves your pointer right.
- Visiting a new website while in the past **truncates all forward history** and sets the new site as the latest entry.
- Clicking "Home" (Reset) clears the stack and returns to the default homepage.

---

## 🔑 Key Concepts

- [[Undo-Redo History]]
- History cursor / index pointer navigation
- Redo branch truncation via `Array.prototype.slice` or `array.length` reassignment
- Boundary guard conditions (`canUndo`, `canRedo`)

---

## ⚠️ Edge Cases / Traps

- **Bounds Overflows:** Calling `undo()` when `_index === 0` or `redo()` when `_index === _history.length - 1` must gracefully no-op without negative indexes.
- **Forgetting Redo Invalidation on `set`:** If you undo 3 times and call `set(x)`, all 3 undone states must be discarded immediately.
- **Reset Incomplete Cleanup:** `reset()` must restore `_history = [this._initialValue]` and `_index = 0`. It must clear both past and future history.
- **Reference Mutation:** If storing mutable objects, since spec doesn't require cloning, external in-place mutation affects history unless defensively cloned.

---

## ⭐ Interview Takeaway

1. **The Classic Index-Pointer Pattern:** A simple array plus an integer index is the cleanest implementation of undo/redo.
2. **Boolean Queries:** Always derive `canUndo()` as `this._index > 0` and `canRedo()` as `this._index < this._history.length - 1`.
3. **Array Truncation Trick:** Setting `this._history.length = this._index + 1` is an $O(1)$ in-place way to truncate future redo history in JavaScript engines.

---

## 🎯 Common Interview Questions

### Direct Questions
- How does setting `array.length` in JavaScript truncate an array?
- What is the difference between storing full snapshots vs storing commands in an undo manager?
- How do you implement `canUndo()` and `canRedo()` with $O(1)$ complexity?

### Follow-up Questions
- How would you add a `limit` parameter to limit history to at most 100 items?
- How would you implement this using two stacks (`past` and `future`) instead of an array pointer?
- What happens if the values being managed are large DOM trees or canvas states?

### Conceptual Questions
- How does React's `useReducer` or Redux time-travel debugging implement this same pattern?
- Why is state immutability necessary when building reliable undo/redo managers?

---

## 🔄 Variations

- **Undo / Redo Manager II:** Adding batched draft/commit support.
- **Bounded History Buffer:** Dropping the oldest history entry when reaching a maximum capacity.
- **Undoable Database:** Applying undo/redo to an entity collection with CRUD operations.

---

## 📝 Revision Notes

- **Core idea:** Maintain an array of values with an index cursor.
- **Remember:** On `set()`, discard everything after `_index` before appending.
- **Watch out for:** `reset()` must reset to the original `initialValue` and purge redo history.
- **Complexity:** Time: $O(1)$ for `getCurrent`, `undo`, `redo`, `canUndo`, `canRedo`, and `set`; Space: $O(H)$ where $H$ is number of saved states.

## Official Solution
## Undo / Redo Manager ( Official solution )

Premium
Languages
Undo/redo is history navigation over committed values. The design question is how to represent the active position, the values behind it, and the redo branch that must disappear after a new `set()`.

## Solution

Several designs work for undo/redo state. The best choice depends on the optimization goal: implementation simplicity, conceptual separation of undo vs redo state, or data-structure fluency.

### Approach 1: Single history array + current index

This is the simplest and most interview-friendly approach.

Use:

1. A `history` array containing all recorded values.
2. A `currentIndex` pointer indicating which entry is currently active.

```javascript
history = [initialValue, nextValue, nextNextValue];
currentIndex = 1; // Current value is nextValue.
```

This works well because:

- `getCurrent()` is an O(1) array lookup.
- `undo()` and `redo()` only need to move the pointer when possible.
- `set()` can discard redo history by truncating everything after `currentIndex` before pushing the new value.
- `reset()` can restore the manager to a single-entry history containing the original initial value.

The important branch rule is that `set()` after an `undo()` does not append after the old tail. It replaces the redo branch with a new value, so future `redo()` calls cannot resurrect states that no longer belong to the active history.

Example state changes after `new UndoRedoManager(0); set(1); set(2); undo(); set(3)`:

| Operation | History | Current index | Current value |
| --- | --- | --- | --- |
| initial | `[0]` | `0` | `0` |
| `set(1)` | `[0, 1]` | `1` | `1` |
| `set(2)` | `[0, 1, 2]` | `2` | `2` |
| `undo()` | `[0, 1, 2]` | `1` | `1` |
| `set(3)` | `[0, 1, 3]` | `2` | `3` |

```jsx
/**
 * @template T
 */
export default class UndoRedoManager {
  /**
   * @param {T} initialValue
   */
  constructor(initialValue) {
    this._initialValue = initialValue;
    this._history = [initialValue];
    this._currentIndex = 0;
  }

  /**
   * @returns {T}
   */
  getCurrent() {
    return this._history[this._currentIndex];
  }

  /**
   * @param {T} value
   * @returns {void}
   */
  set(value) {
    // Discard everything after the current pointer so redo history is cleared
    // before appending the next committed value.
    this._history = this._history.slice(0, this._currentIndex + 1);
    this._history.push(value);
    this._currentIndex = this._history.length - 1;
  }

  /**
   * @returns {void}
   */
  undo() {
    if (!this.canUndo()) {
      return;
    }

    this._currentIndex -= 1;
  }

  /**
   * @returns {void}
   */
  redo() {
    if (!this.canRedo()) {
      return;
    }

    this._currentIndex += 1;
  }

  /**
   * @returns {void}
   */
  reset() {
    // Reset back to a single-entry history containing the original value.
    this._history = [this._initialValue];
    this._currentIndex = 0;
  }

  /**
   * @returns {boolean}
   */
  canUndo() {
    return this._currentIndex > 0;
  }

  /**
   * @returns {boolean}
   */
  canRedo() {
    return this._currentIndex < this._history.length - 1;
  }
}
```

This is the smallest and most interview-friendly implementation. The control flow is direct, `getCurrent()` is just an array lookup, and `undo()` / `redo()` only move an index. In exchange, redo truncation is handled somewhat implicitly by slicing the array, so the model is simple but a little less expressive than approaches that separate past and future history more explicitly.

This is usually the best default for interviews, local-only editors, and settings-style UIs where the history is just a linear list of values and there is no need for richer branching semantics.

### Approach 2: Past / current / future stacks

Another common undo/redo model explicitly separates the state into three buckets:

- `past`: everything before the current value
- `current`: the active value
- `future`: redo history

`set()` pushes the current value into `past`, updates `current`, and clears `future`. `undo()` moves `current` into `future` and restores the latest value from `past`. `redo()` does the opposite.

This approach is useful when the model itself should mirror how people describe undo/redo in product discussions: "past states", "current state", and "future states".

```jsx
interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

export default class UndoRedoManagerStacks<
  T = unknown,
> implements IUndoRedoManager<T> {
  _initialValue: T;
  _past: Array<T>;
  _current: T;
  _future: Array<T>;

  constructor(initialValue: T) {
    this._initialValue = initialValue;
    this._past = [];
    this._current = initialValue;
    this._future = [];
  }

  getCurrent(): T {
    return this._current;
  }

  set(value: T): void {
    // Setting a new value promotes the current value into past history and
    // invalidates any redo path.
    this._past.push(this._current);
    this._current = value;
    this._future = [];
  }

  undo(): void {
    if (!this.canUndo()) {
      return;
    }

    // The current value becomes redo-able, and the latest past value becomes
    // the new current value.
    this._future.push(this._current);
    this._current = this._past.pop() as T;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    // Redo is the reverse transfer: current goes back into past, and the most
    // recent future value becomes current again.
    this._past.push(this._current);
    this._current = this._future.pop() as T;
  }

  reset(): void {
    this._past = [];
    this._current = this._initialValue;
    this._future = [];
  }

  canUndo(): boolean {
    return this._past.length > 0;
  }

  canRedo(): boolean {
    return this._future.length > 0;
  }
}
```

The main strength here is conceptual clarity. `past`, `current`, and `future` map closely to how people naturally explain undo/redo, which can make the approach easier to follow once the state transfers are clear. That clarity costs more moving pieces to keep in sync than in the array-and-index version, so the code is a bit heavier even though the model is clean.

This approach is a good fit when readability of the state layout matters, such as collaborative discussions with teammates, product-oriented interviews, or codebases where people already talk about history in terms of past, current, and future state.

### Approach 3: Doubly linked list

A more data-structure-oriented solution is to represent history as a doubly linked list and keep a pointer to the current node.

Each node stores:

- `value`
- `prev`
- `next`

`undo()` moves to `prev`, `redo()` moves to `next`, and `set()` attaches a new node after the current node and makes it the new tail of the active branch.

This is useful for teaching pointer-based navigation and node relationships without relying on array indices.

```jsx
interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  undo(): void;
  redo(): void;
  reset(): void;
  canUndo(): boolean;
  canRedo(): boolean;
}

type HistoryNode<T> = {
  value: T;
  prev: HistoryNode<T> | null;
  next: HistoryNode<T> | null;
};

function createNode<T>(value: T): HistoryNode<T> {
  return {
    value,
    prev: null,
    next: null,
  };
}

export default class UndoRedoManagerLinkedList<
  T = unknown,
> implements IUndoRedoManager<T> {
  _initialValue: T;
  _head: HistoryNode<T>;
  _current: HistoryNode<T>;

  constructor(initialValue: T) {
    const node = createNode(initialValue);
    this._initialValue = initialValue;
    this._head = node;
    this._current = node;
  }

  getCurrent(): T {
    return this._current.value;
  }

  set(value: T): void {
    const node = createNode(value);
    node.prev = this._current;
    // Overwriting `next` drops any redo branch that used to hang off the
    // current node, then the new node becomes the active tail.
    this._current.next = node;
    this._current = node;
  }

  undo(): void {
    if (!this.canUndo()) {
      return;
    }

    this._current = this._current.prev as HistoryNode<T>;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    this._current = this._current.next as HistoryNode<T>;
  }

  reset(): void {
    const node = createNode(this._initialValue);
    // Rebuild the list from scratch so both undo and redo history are cleared.
    this._head = node;
    this._current = node;
  }

  canUndo(): boolean {
    return this._current.prev !== null;
  }

  canRedo(): boolean {
    return this._current.next !== null;
  }
}
```

The advantage of this version is that navigation is very direct: each history entry knows its neighbors, so `undo()` and `redo()` become pointer moves instead of index arithmetic. It takes the most code, requires careful pointer updates, and is easier to get wrong than the other two approaches. For a simple value-history question, that extra complexity often does not buy much.

This approach is most useful in data-structure-focused interviews, linked-list teaching scenarios, or systems where node relationships matter more rather than treat history as a plain sequence in an array.

## Edge cases

- Calling `undo()` on the first history entry should do nothing.
- Calling `redo()` on the last history entry should do nothing.
- Calling `set()` after `undo()` should clear the redo branch before adding the new value.
- Setting the same value multiple times should still create distinct history steps.
- Values are stored by reference; later external mutation of an object/array value should still be visible through the manager.
- `reset()` should return to the original constructor value, not merely the earliest remaining value after previous edits.

## Techniques

- Object-oriented programming
- Using arrays as history logs
- Managing state with an index pointer

## Notes

- This question intentionally does not require deep cloning, value comparison, or exposing the raw history array.
- Using a single `history` array plus `currentIndex` is a common interview-friendly model for state time travel.
- A command-pattern-based undo/redo system is related, but is intentionally out of scope here because this API stores values directly rather than reversible command objects.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
An array-based manager copies the retained history prefix on every `set()`, including when already at the latest entry. For a long sequence of sets without undo, what cumulative copying cost does this introduce? Suggest a change that preserves branch behavior.

Your notes (optional)
