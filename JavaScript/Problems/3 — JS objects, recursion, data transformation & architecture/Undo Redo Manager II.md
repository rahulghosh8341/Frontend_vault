---
title: Undo / Redo Manager II
aliases:
  - Undo Redo Manager II
  - Undo / Redo Manager II
difficulty: Hard
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/undo-redo-manager-ii"
pattern:
  - "[[Undo-Redo History]]"
concepts:
  - "[[Undo-Redo History]]"
  - "[[Transactions & Drafts]]"
  - "[[Object-Oriented Programming]]"
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Undo / Redo Manager II

> [!info] Problem
> Implement a class that manages draft values and batch-based undo/redo checkpoints

## Problem

## Undo / Redo Manager II

This is a follow-up to [Undo / Redo Manager](/questions/javascript/undo-redo-manager).

In many products, successive updates should be grouped into a single undo step. For example, typing three characters into a text editor often should not require three separate undo presses. Instead, the app may keep updating the visible value while only committing a history checkpoint once the typing burst is done.

In this question, implement a reusable `UndoRedoManager` class with that behavior:

- `set()` updates the live draft immediately.
- `commit()` records the current draft as one undo/redo history step.

## Examples

```javascript
const manager = new UndoRedoManager('');

manager.getCurrent(); // ''
manager.canUndo(); // false
manager.canRedo(); // false

manager.set('h');
manager.set('he');
manager.set('hel');

manager.getCurrent(); // 'hel'
manager.canUndo(); // true because undo() would discard the pending draft
manager.canRedo(); // false

manager.commit();
manager.getCurrent(); // 'hel'

manager.undo();
manager.getCurrent(); // ''
manager.canRedo(); // true

manager.redo();
manager.getCurrent(); // 'hel'

manager.set('hell');
manager.set('hello');
manager.getCurrent(); // 'hello'

manager.undo(); // Discards the uncommitted draft.
manager.getCurrent(); // 'hel'
manager.canRedo(); // false

manager.set('hello');
manager.commit();
manager.getCurrent(); // 'hello'

manager.reset();
manager.getCurrent(); // ''
manager.canUndo(); // false
manager.canRedo(); // false
```

## UndoRedoManager API

Implement the following APIs on the `UndoRedoManager`:

### new UndoRedoManager(initialValue)

Creates an instance of the `UndoRedoManager` class with `initialValue` as the first committed history entry. History is isolated within each instance.

Values should be stored as-is. You do **not** need to deep clone them.

| Parameter | Type | Description |
| --- | --- | --- |
| `initialValue` | `unknown` | The initial value to seed the history with. |

### manager.getCurrent()

Returns the current visible value.

If there is a pending draft started by `set()`, return that draft value. Otherwise return the current committed history value.

### manager.set(value)

Updates the live draft to `value`.

Repeated `set()` calls before `commit()` belong to the same batch, so they should not create multiple committed history steps.

If some committed values had been undone before the first `set()` in a new batch, all redo history should be discarded immediately when that new batch starts.

| Parameter | Type | Description |
| --- | --- | --- |
| `value` | `unknown` | The new live draft value. |

### manager.commit()

Commits the current draft as exactly one new history entry.

If there is no pending draft, this method should do nothing.

### manager.undo()

Moves backward if possible.

If there is a pending draft, `undo()` should discard that draft and restore the latest committed value instead of traversing committed history.

Otherwise, if the manager is already at the earliest committed history entry, this method should do nothing.

### manager.redo()

Moves the current pointer one committed step forward if possible.

Discarded uncommitted drafts are **not** redoable.

If the manager is already at the latest committed history entry, this method should do nothing.

### manager.reset()

Restores the manager to its original `initialValue`, clears committed history back to one entry, and discards any pending draft.

### manager.canUndo()

Returns `true` if `undo()` would change the current state.

That includes both:

- discarding a pending draft, or
- moving to an earlier committed history entry.

### manager.canRedo()

Returns `true` if there is a later committed history entry to move to, otherwise `false`.

## Notes

- Calling `commit()` after several `set()` calls should still create only one committed history step.
- A committed batch should still create a history step even if it ends on the same value it started with.
- You do not need to implement timers, nested batches, async batching, or history inspection APIs such as `getHistory()` or `getIndex()`.

## Hints

### Hint 1 : Which value is visible but not committed?

### Hint 2 : When does a new branch begin?

### Hint 3 : What should undo change first?

## Asked at these companies

Anthropic
Figma

## 🤔 Thought Process

- **Immediate Recognition:** Batched checkpoint extension of [[Undo Redo Manager]]. Multiple rapid `set()` calls update a live draft; only explicit `commit()` creates a history step.
- **Core Problem:** Avoid spamming history with granular keystrokes (e.g. typing characters into a text field) while keeping the visible value reactive and current.
- **State Model:**
  - `_history = [initialValue]`
  - `_currentIndex = 0`
  - `_hasDraft = false`
  - `_draft = null`
- **Key Invariants:**
  1. `getCurrent()`: returns `_hasDraft ? _draft : _history[_currentIndex]`.
  2. First `set()` while undone immediately discards redo history (don't wait for commit).
  3. `undo()` with pending draft: discards `_draft` and leaves committed history untouched.
  4. `undo()` without pending draft: moves `_currentIndex` backward if possible.
  5. `commit()`: if `_hasDraft`, appends `_draft` to `_history`, advances index, resets `_hasDraft = false`. If no draft, does nothing.
  6. `canUndo()`: true if `_hasDraft` OR `_currentIndex > 0`.

---

## 🧠 Mental Model

Think of **Text Editor Typing with Autosave / Manual Save**:
- Typing characters updates the active screen buffer (`_draft`).
- Hitting `Ctrl+Z` (Undo) while typing discards the current burst back to the last save point.
- Hitting `Ctrl+S` (`commit`) seals the buffer into the undo history stack.
- Pressing `Ctrl+Z` again after saving undoes the previous saved checkpoint.

---

## 🔑 Key Concepts

- [[Undo-Redo History]]
- Debounced / Batched Checkpointing
- Draft staging vs committed history
- Immediate branch invalidation on dirty draft initiation

---

## ⚠️ Edge Cases / Traps

- **`canUndo()` when draft is dirty at index 0:** Even if `_currentIndex === 0`, if there is an uncommitted draft, `canUndo()` must return `true` because `undo()` can discard the draft!
- **First `set()` Redo Invalidation:** Calling `set()` after undoing must clear redo entries immediately, even before `commit()` is called.
- **Multiple `commit()` calls:** Subsequent calls to `commit()` without new `set()` calls are no-ops.
- **`reset()` while draft is active:** Must clear both the committed history and the active draft, restoring to `initialValue`.

---

## ⭐ Interview Takeaway

1. **Flag + Draft Value:** Represent draft state with either an explicit boolean flag `_hasDraft` (allows storing `undefined` as valid draft value) or an object sentinel.
2. **Interactive UI Optimization:** Explain to the interviewer that batched undo/redo is essential for rich text editors, canvas painting (stroke start -> stroke end), and form wizards.
3. **Symmetry with Database Checkpoints:** This problem shares the exact same state machine as [[Undoable Database II]].

---

## 🎯 Common Interview Questions

### Direct Questions
- Why does `canUndo()` return `true` when at `_currentIndex === 0` if a draft is pending?
- Why should redo history be cleared when the draft starts rather than when it commits?
- How does `undo()` decide whether to discard a draft or decrement the history pointer?

### Follow-up Questions
- How would you implement automatic batching based on time idle (e.g. typing pause of 500ms)?
- How would you merge sequential text insertions into single undo operations without manual `commit()` calls?

### Conceptual Questions
- How does this draft/commit pattern compare to Git's index/staging area?
- Why is separating draft state from history critical for avoiding memory bloat in high-frequency input streams?

---

## 🔄 Variations

- **Auto-Debounced Undo Manager:** Automatically commits after $N$ milliseconds of inactivity.
- **Undoable Database II:** Multi-entity database implementation with draft staging.
- **Command Grouping:** Grouping commands into macro transactions.

---

## 📝 Revision Notes

- **Core idea:** Combine an array of committed states with a dirty draft buffer.
- **Remember:** `undo()` discards the draft first; `canUndo()` returns `true` if draft is dirty or `_currentIndex > 0`.
- **Watch out for:** Clear redo history on the first `set()` that initiates a draft.
- **Complexity:** Time: $O(1)$ for all operations; Space: $O(H)$ where $H$ is number of committed history entries.

## Official Solution
## Undo / Redo Manager II ( Official solution )

Premium
Languages
This question adds one extra concept on top of the original undo/redo manager: a difference between the live value the user is currently editing and the committed checkpoints that should show up in history.

That makes it a good fit for scenarios like typing, sliders, or drag interactions, where the UI should update immediately but undo/redo should only move across larger chunks.

## Solution

The state picture separates two layers:

1. Committed history that undo/redo traverses.
2. An optional draft value that reflects the current in-progress batch.

### Approach 1: Committed history array + current index + pending draft

This is the most interview-friendly model.

Use:

1. A `history` array containing committed checkpoints.
2. A `currentIndex` pointer to the active committed checkpoint.
3. A `draftValue` plus a boolean that records whether a draft currently exists.

```javascript
history = ['hello', 'hello world'];
currentIndex = 1;
draftValue = 'hello world!';
hasDraft = true;
```

This gives clean behavior:

- `getCurrent()` returns `draftValue` when a batch is in progress, otherwise `history[currentIndex]`.
- The first `set()` in a new batch truncates history after `currentIndex` so redo history is cleared immediately.
- Later `set()` calls in the same batch only replace `draftValue`.
- `commit()` appends the latest draft as exactly one new history step.
- `undo()` discards the draft first if one exists. Otherwise it moves `currentIndex` backward.

Capability checks should follow the same split. A draft means `canUndo()` is true because the draft can be discarded, while `canRedo()` is false because redo only moves across committed checkpoints.

Example state changes for typing `h`, `he`, `hel`, then committing:

| Operation | Committed history | Draft | Visible value |
| --- | --- | --- | --- |
| initial | `['']` | none | `''` |
| `set('h')` | `['']` | `'h'` | `'h'` |
| `set('he')` | `['']` | `'he'` | `'he'` |
| `set('hel')` | `['']` | `'hel'` | `'hel'` |
| `commit()` | `['', 'hel']` | none | `'hel'` |

Undo during a draft is intentionally different from undo after commit:

| State before `undo()` | Operation result | Redo available? |
| --- | --- | --- |
| `history = ['', 'hel']`, draft `'hello'` | discard draft, show `'hel'` | no |
| `history = ['', 'hel']`, no draft | move index back to `''` | yes |

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
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  /**
   * @returns {T}
   */
  getCurrent() {
    return this._hasDraft
      ? this._draftValue
      : this._history[this._currentIndex];
  }

  /**
   * @param {T} value
   * @returns {void}
   */
  set(value) {
    if (!this._hasDraft) {
      // Starting a new draft branch immediately invalidates redo history.
      this._history = this._history.slice(0, this._currentIndex + 1);
    }

    this._draftValue = value;
    this._hasDraft = true;
  }

  /**
   * @returns {void}
   */
  commit() {
    if (!this._hasDraft) {
      return;
    }

    this._history.push(this._draftValue);
    this._currentIndex = this._history.length - 1;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  /**
   * @returns {void}
   */
  undo() {
    if (this._hasDraft) {
      this._draftValue = undefined;
      this._hasDraft = false;
      return;
    }

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
    this._history = [this._initialValue];
    this._currentIndex = 0;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  /**
   * @returns {boolean}
   */
  canUndo() {
    return this._hasDraft || this._currentIndex > 0;
  }

  /**
   * @returns {boolean}
   */
  canRedo() {
    return this._currentIndex < this._history.length - 1;
  }
}
```

This is the smallest implementation, has straightforward control flow, and is usually the easiest version for candidates to derive in an interview. The main tradeoff is that it optimizes for simplicity more than explicit modeling, and array truncation is a little less expressive than a structure that separates past and future history.

It is usually the best fit for interview settings, local-only editors, and products where history is value-based and rich branching behavior is out of scope.

### Approach 2: Past / current / future stacks + pending draft

Committed state can also be split into three buckets:

- `past`: committed checkpoints before the current one
- `current`: the active committed checkpoint
- `future`: redoable committed checkpoints
- `draftValue`: the in-progress batch, if any

This can feel natural because the data model matches product language:

- `set()` updates the draft and clears `future` on the first new draft after an undo.
- `commit()` pushes `current` into `past`, promotes the draft into `current`, and clears `future`.
- `undo()` either drops the draft or moves `current` into `future` while restoring the latest `past` value.

```jsx
interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  commit(): void;
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
  _draftValue: T | undefined;
  _hasDraft: boolean;

  constructor(initialValue: T) {
    this._initialValue = initialValue;
    this._past = [];
    this._current = initialValue;
    this._future = [];
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  getCurrent(): T {
    return this._hasDraft ? (this._draftValue as T) : this._current;
  }

  set(value: T): void {
    if (!this._hasDraft) {
      // Starting a new draft branch invalidates any redoable future state.
      this._future = [];
    }

    this._draftValue = value;
    this._hasDraft = true;
  }

  commit(): void {
    if (!this._hasDraft) {
      return;
    }

    this._past.push(this._current);
    this._current = this._draftValue as T;
    this._future = [];
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  undo(): void {
    if (this._hasDraft) {
      this._draftValue = undefined;
      this._hasDraft = false;
      return;
    }

    if (!this.canUndo()) {
      return;
    }

    this._future.push(this._current);
    this._current = this._past.pop() as T;
  }

  redo(): void {
    if (!this.canRedo()) {
      return;
    }

    this._past.push(this._current);
    this._current = this._future.pop() as T;
  }

  reset(): void {
    this._past = [];
    this._current = this._initialValue;
    this._future = [];
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  canUndo(): boolean {
    return this._hasDraft || this._past.length > 0;
  }

  canRedo(): boolean {
    return this._future.length > 0;
  }
}
```

This model reads almost like the product spec itself. `past`, `current`, and `future` make undo/redo flows easy to explain and trace. That readability comes with more moving parts to keep in sync, especially once a draft layer is added, so it is a little heavier than the array-and-index approach.

It works well when clarity of the state layout matters more than minimizing fields, or when people already describe history in terms of past, current, and future.

### Approach 3: Doubly linked list + pending draft

A linked list version works too for a node-based model instead of an index-based model.

Each committed node stores:

- `value`
- `prev`
- `next`

Keep a pointer to the current committed node plus an optional draft value.

- `undo()` and `redo()` move across committed nodes.
- The first `set()` after an undo can clear redo history by detaching `current.next`.
- `commit()` inserts one new node after `current`.

```jsx
interface IUndoRedoManager<T = unknown> {
  getCurrent(): T;
  set(value: T): void;
  commit(): void;
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
  _draftValue: T | undefined;
  _hasDraft: boolean;

  constructor(initialValue: T) {
    const node = createNode(initialValue);
    this._initialValue = initialValue;
    this._head = node;
    this._current = node;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  getCurrent(): T {
    return this._hasDraft ? (this._draftValue as T) : this._current.value;
  }

  set(value: T): void {
    if (!this._hasDraft) {
      // Starting a new draft branch drops any redo chain hanging off current.
      this._current.next = null;
    }

    this._draftValue = value;
    this._hasDraft = true;
  }

  commit(): void {
    if (!this._hasDraft) {
      return;
    }

    const node = createNode(this._draftValue as T);
    node.prev = this._current;
    this._current.next = node;
    this._current = node;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  undo(): void {
    if (this._hasDraft) {
      this._draftValue = undefined;
      this._hasDraft = false;
      return;
    }

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
    this._head = node;
    this._current = node;
    this._draftValue = undefined;
    this._hasDraft = false;
  }

  canUndo(): boolean {
    return this._hasDraft || this._current.prev !== null;
  }

  canRedo(): boolean {
    return this._current.next !== null;
  }
}
```

Traversal is direct here, and the data structure naturally expresses a chain of history nodes without relying on array indices. The downside is that this version needs the most code and is the easiest to get wrong because pointer updates must stay correct. For a value-history interview problem, that extra complexity usually does not buy much.

This approach is most useful in data-structure-focused interviews, linked-list teaching scenarios, or systems that may eventually grow into richer history graphs where node relationships matter.

## Edge cases

- `commit()` with no pending draft should do nothing.
- Repeated `set()` calls before `commit()` should still produce only one committed history step.
- `undo()` during an active draft should discard the draft rather than auto-commit it.
- Starting a new draft after undo should clear redo history immediately.
- Batches that end on the same value they started with should still create a committed step.
- Values are stored by reference; later external mutation of objects and arrays is still visible.

## Techniques

- Object-oriented programming
- Separating draft state from committed checkpoints
- Managing history with either arrays, stacks, or linked structures

## Notes

The exact batching model for undo/redo is a product decision, not just an implementation decision:

- This question intentionally keeps batching synchronous and explicit with `commit()`.
- It does not require debouncing, timers, nested batches, command objects, or exposing raw history internals.

This question chooses explicit `commit()` because it is deterministic, interview-friendly, and easy to test. It keeps the focus on the state picture instead of time-based heuristics.

That said, real apps often choose other APIs:

- `batch(fn)` is convenient when the app knows a sequence of updates belongs together.
- `beginBatch()` / `endBatch()` can support more flexible transactional flows.
- `set(value, { batch: true })` keeps everything on one method, though it can make the call sites less clear.

Different products also make different choices about draft visibility:

- Some show the latest draft immediately, like this question does, because the UI should reflect live typing or dragging.
- Others keep changes hidden until a commit point, especially for forms, wizards, or expensive previews.

`undo()` behavior during a draft is another deliberate choice:

- Discarding the draft keeps the frame simple.
- Some apps auto-commit the draft before undoing.
- Others block undo until the draft is finalized.

Redo clearing is also product-dependent:

- This question clears redo history on the first new draft change after an undo.
- Some systems wait until commit time.
- Collaborative or branching editors may preserve multiple futures instead of dropping them.

Even same-value batches are debatable:

- Here, committing the same final value still creates a step for consistency with the original question.
- In production, some apps collapse no-op commits to keep history smaller.

Many real systems go further than this question:

- Timed batching with debounce windows
- Nested transactions
- Async commits
- Command-pattern undo stacks
- Multi-user or server-synced history

There is no single universal undo/redo mechanism. The right choice depends on the desired UX, the cost of storing history, whether changes are local or collaborative, and how much control the product needs over grouping behavior.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A test checks that undoing a pending draft restores the last committed value. A bug also moves the committed-history cursor back one step, but the test uses only the initial checkpoint. Design a stronger test that detects both effects independently.

Your notes (optional)
