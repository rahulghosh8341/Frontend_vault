---
aliases:
  - Undo-Redo History
  - Undo Redo
  - Command Pattern
---

## Core Idea

The Undo-Redo History pattern manages state transitions across time by maintaining historical snapshots or reversible operation commands. When state mutates, future redo branches are discarded, and the new state is appended to the timeline.

## Recognition

- Requirement to support `.undo()` and `.redo()`.
- State transitions must be revertible to previous points in time.
- Performing a new mutation while in an undone state invalidates any subsequent redo history.

## Template

### Approach A: Snapshot Pointer (Memento Pattern)

```js
class HistoryManager {
  constructor(initialState) {
    this.history = [initialState];
    this.currentIndex = 0;
  }

  getCurrent() {
    return this.history[this.currentIndex];
  }

  commit(nextState) {
    // Truncate redo future
    this.history = this.history.slice(0, this.currentIndex + 1);
    this.history.push(nextState);
    this.currentIndex = this.history.length - 1;
  }

  undo() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
    }
    return this.getCurrent();
  }

  redo() {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
    }
    return this.getCurrent();
  }
}
```

### Approach B: Two Stacks (Past & Future)

```js
class TwoStackHistory {
  constructor(initialState) {
    this.past = [];
    this.current = initialState;
    this.future = [];
  }

  commit(nextState) {
    this.past.push(this.current);
    this.current = nextState;
    this.future = []; // Clear redo stack on new action
  }

  undo() {
    if (this.past.length === 0) return this.current;
    this.future.push(this.current);
    this.current = this.past.pop();
    return this.current;
  }

  redo() {
    if (this.future.length === 0) return this.current;
    this.past.push(this.current);
    this.current = this.future.pop();
    return this.current;
  }
}
```

### Approach C: Reversible Commands (Inverse Operations)

```js
class CommandHistory {
  constructor() {
    this.undoStack = [];
    this.redoStack = [];
  }

  execute(command) {
    command.apply();
    this.undoStack.push(command);
    this.redoStack = [];
  }

  undo() {
    if (this.undoStack.length === 0) return;
    const command = this.undoStack.pop();
    command.invert();
    this.redoStack.push(command);
  }

  redo() {
    if (this.redoStack.length === 0) return;
    const command = this.redoStack.pop();
    command.apply();
    this.undoStack.push(command);
  }
}
```

## Variations

1. **Full Snapshots**: Simple, fast reads; high memory consumption for large states.
2. **Structural Sharing / Immutable Trie**: Shares unchanged nodes between snapshots (e.g., Immer, Immutable.js).
3. **Delta / Diff Operations (JSON Patch)**: Stores only deltas between consecutive states.
4. **Inverse Commands**: Stores discrete actions with invert functions (`add` <-> `delete`, `update(before, after)`). Memory efficient, requires index tracking.

## Complexity

- **Snapshot Approach**: Time $O(1)$ to $O(N)$ per commit (cloning cost); Space $O(S \times N)$ where $S$ is history steps.
- **Two Stacks**: Time $O(1)$ pointer/stack operation; Space $O(S \times N)$.
- **Command Inversion**: Time $O(1)$ to $O(K)$ per operation; Space $O(S \times D)$ where $D$ is changed delta size.

## Common Mistakes

- Forgetting to truncate/clear the redo future when a new mutation occurs after undoing.
- Mutating stored snapshots in-place (failing to shallow/deep clone before modification).
- Restoring deleted items at the end of a list instead of their original insertion index.
- Not bounding history size in long-lived production systems (leading to memory leaks).

## Interview Tips

- Always clarify whether full snapshots or delta/command storage is preferred.
- Mention trade-offs: Snapshot is simplest and error-resistant; Command pattern saves memory but requires strict inversion math and index stability.
- Explicitly protect data boundaries by returning defensive shallow/deep copies.

## Problems Using This Pattern

- [[Undoable Database]]
- [[Undoable Database II]]
- [[Undo Redo Manager]]
- [[Undo Redo Manager II]]
- [[Undoable Counter]]
- [[Map With History]]

## Related Patterns

- [[Observer]]
- [[Method Chaining]]

## Related Concepts

- Memento Pattern
- Command Pattern
- Object Immutability
- Stacks & Pointers
