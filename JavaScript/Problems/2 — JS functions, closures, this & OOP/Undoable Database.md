---
title: Undoable Database
aliases:
  - Undoable Database
difficulty: Medium
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/undoable-database"
companies:
  - "[[Anthropic]]"
  - "[[Figma]]"
  - "[[Rippling]]"
pattern:
  - "[[Undo-Redo History]]"
concepts:
  - "[[Undo-Redo History]]"
  - "[[Object-Oriented Programming]]"
  - "[[Array.prototype.slice]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Undoable Database

> [!info] Problem
> Implement a class that manages users with CRUD operations and undo/redo history

## Problem

## Undoable Database

Undo/redo is often applied to more than a single value. In admin tools, editors, and dashboards, one history step might represent creating, editing, or deleting a full record.

In this question, implement a reusable `Database` class that stores users and supports undo/redo for CRUD operations.

Each user has an `id` and may contain arbitrary additional properties.

## Examples

```javascript
const database = new Database();

database.getUsers(); // []

database.addUser({ id: 1, name: 'Alice', role: 'admin' });
database.addUser({ id: 2, name: 'Bob' });
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]

database.updateUser(2, { role: 'editor', active: true });
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]

database.deleteUser(1);
database.getUsers();
// [{ id: 2, name: 'Bob', role: 'editor', active: true }]

database.undo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]

database.undo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]

database.redo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'editor', active: true },
// ]
```

## Database API

Implement the following APIs on the `Database`:

### new Database()

Creates an instance of the `Database` class with no users. User data and history are isolated within each `Database` instance.

### database.getUsers()

Returns the current users in insertion order.

Return a new array each time. User objects only need shallow copies.

### database.addUser(user)

Appends `user` to the database as a new history step.

| Parameter | Type | Description |
| --- | --- | --- |
| `user` | `Object` | A user record with an `id` plus any additional properties. |

### database.updateUser(id, updates)

Shallow-merges `updates` into the matching user and records exactly one new history step.

The user's original `id` should be preserved.

| Parameter | Type | Description |
| --- | --- | --- |
| `id` | `string \| number` | The ID of the user to update. |
| `updates` | `Object` | Partial properties to merge into the stored user. |

### database.deleteUser(id)

Removes the matching user and records exactly one new history step.

| Parameter | Type | Description |
| --- | --- | --- |
| `id` | `string \| number` | The ID of the user to delete. |

### database.undo()

Reverts the most recent successful CRUD change if possible.

If there is no earlier history entry, this method should do nothing.

### database.redo()

Reapplies the most recently undone CRUD change if possible.

If there is no later history entry, this method should do nothing.

## Notes

- Every successful `addUser()`, `updateUser()`, and `deleteUser()` call should create exactly one undoable history step.
- A successful CRUD call after `undo()` should clear redo history.
- Calling `updateUser()` with values identical to the current top-level values should still create a history step.
- You may assume `addUser()` is only called with a unique `id`, and `updateUser()` / `deleteUser()` are only called with existing IDs.
- You do not need to deep clone nested objects or arrays inside user records.

## Hints

### Hint 1 : What must one history entry restore?

### Hint 2 : Which copies protect saved history?

### Hint 3 : What remains stable during an update?

## Asked at these companies

Anthropic
Figma
Rippling

## 🤔 Thought Process

- **Immediate Recognition:** This is a state management problem requiring timeline navigation (`undo` / `redo`) combined with tabular CRUD operations.
- **Core Problem:** When a mutation (`addUser`, `updateUser`, `deleteUser`) happens, we must persist enough information to restore past states and discard any active "future" redo branch.
- **Before Coding Decision:** Decide between *Full Snapshots* vs *Two Stacks (Past/Future)* vs *Command Inversion*. Full snapshots with a pointer index is the fastest, least error-prone approach for interviews.
- **Simplest Approach:** Maintain `_history = [[]]` and `_currentIndex = 0`. Each mutation clones the current snapshot, applies changes, slices off future history (`slice(0, _currentIndex + 1)`), appends the new snapshot, and advances `_currentIndex`.
- **Defensive Copying:** `getUsers()` and mutations must return/store copies (`cloneUsers`) to prevent callers from corrupting the internal history timeline by mutating returned objects.
- **Edge Cases to Identify:** Same-value updates must still create a history step; updates must preserve the original `id`; undo at the earliest entry and redo at the latest entry must safely no-op; insertions must maintain order.

---

## 🧠 Mental Model

Think of undo/redo as a **linear tape head**:
- The tape contains static snapshots of user lists: `[S0, S1, S2]`.
- `currentIndex` points to the active snapshot.
- `undo()` moves the tape head left; `redo()` moves the tape head right.
- Any write operation while the tape head is in the past **slices off everything to the right** and records a new present.

---

## 🔑 Key Concepts

- [[Undo-Redo History]]
- Memento Pattern / State Snapshotting
- Defensive shallow cloning (`Object.assign` / spread `{...user}`)
- Timeline truncation on branch divergence (`Array.prototype.slice`)
- Index tracking vs pointer invalidation

---

## ⚠️ Edge Cases / Traps

- **Mutating the Active Snapshot Directly:** Mutating the existing array in `_history[_currentIndex]` mutates past states retroactively. You must clone the snapshot before applying changes.
- **External Mutation Leakage:** Returning raw user objects from `getUsers()` allows external callers to mutate history records. Return shallow copies of both the array and user records.
- **Losing Redo Branch Invalidation:** Forgetting to drop redo history when a new mutation occurs after an undo. Subsequent redos would restore an invalid parallel timeline.
- **Overwriting ID on Update:** An update payload like `{ id: 999, name: 'Alice' }` must not overwrite the user's primary key `id`.
- **Updating with Identical Values:** Even if properties are identical, the spec explicitly requires creating a new history step. Do not optimize it away.

---

## ⭐ Interview Takeaway

1. **Default to Full Snapshots in Interviews:** Unless state size is massive, snapshot arrays with an index pointer (`_history`, `_currentIndex`) eliminate complex inverse-operation bugs.
2. **Defensive Copies at the Boundaries:** Always clone when data enters via CRUD methods and when data exits via `getUsers()`.
3. **Commit Invariant:** Every commit follows the golden rule: `slice(0, currentIndex + 1)` -> `push(nextState)` -> `currentIndex++`.
4. **Command Pattern as Alternative:** Mention that in large production systems, storing reversible operations (`add` <-> `delete`, `update(before, after)`) saves significant memory.

---

## 🎯 Common Interview Questions

### Direct Questions
- Why does a new mutation after an undo clear the redo history?
- What are the tradeoffs between snapshot-based undo/redo and command-based undo/redo?
- Why does `getUsers()` need to clone user objects rather than just returning `[...this._currentUsers]`?

### Follow-up Questions
- How would you implement a history limit (e.g., maximum 50 undo steps) without unbounded memory growth?
- How would you handle transactions or batch updates where multiple user changes should count as a single undo step? (See [[Undoable Database II]])
- If user records contain deep nested objects or arrays, how would shallow copying fail and how would you resolve it?

### Conceptual Questions
- What is the Memento design pattern and how does it apply to frontend application state?
- How do modern libraries like Redux or Immer achieve structural sharing to optimize memory in historical snapshots?

---

## 🔄 Variations

- **Fixed-Capacity Undo Buffer:** Restricting history length to $K$ entries using a ring buffer or shifting from the front.
- **Batch / Checkpoint Commits ([[Undoable Database II]]):** Accumulating edits in a draft and committing only upon explicit user confirmation.
- **Command / Delta Based History:** Storing diff patches (e.g., RFC 6902 JSON Patch) instead of full state clones.
- **Time Travel Debugging:** Exposing `jumpTo(stepIndex)` to move directly to arbitrary points in history.

---

## 📝 Revision Notes

- **Core idea:** Maintain an array of snapshot states with a current pointer; truncate future snapshots on any new write.
- **Remember:** Always slice `_history.slice(0, _currentIndex + 1)` before pushing new state.
- **Watch out for:** Reference leakage—shallow clone both the array and user objects when reading or writing.
- **Complexity:** Time: $O(N)$ per CRUD operation due to cloning $N$ users; Space: $O(H \times N)$ where $H$ is number of history entries.

## Official Solution
## Undoable Database ( Official solution )

Premium
Languages
This question adds CRUD behavior on top of an undo/redo history model. The main design decision is what gets stored in history: full snapshots, separated past/current/future snapshots, or reversible operations.

## Solution

Several designs work for this database history. The best one depends on whether the goal is the smallest interview implementation, a more explicit snapshot model, or an operation-based history.

### Approach 1: Full snapshots + current index

This is the most interview-friendly approach.

Use:

1. A `history` array where each entry is the full list of users at that moment.
2. A `currentIndex` pointer to the active snapshot.

```javascript
history = [
  [],
  [{ id: 1, name: 'Alice' }],
  [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
  ],
];
currentIndex = 2;
```

This works well because:

- `getUsers()` is just a lookup plus shallow cloning for the return value.
- Each successful CRUD call can build exactly one next snapshot.
- `undo()` and `redo()` only move the history pointer when possible.
- A new mutation after `undo()` can clear redo history by truncating snapshots after `currentIndex`.

| Operation | History shape | `currentIndex` |
| --- | --- | --- |
| new database | `[[]]` | `0` |
| add Alice | `[[], [Alice]]` | `1` |
| add Bob | `[[], [Alice], [Alice, Bob]]` | `2` |
| undo | unchanged | `1` |
| add Cara | `[[], [Alice], [Alice, Cara]]` | `2` |

The last step shows why redo history must be truncated before committing a new snapshot: the old `[Alice, Bob]` future no longer matches the user's timeline after adding Cara.

```jsx
/**
 * @typedef {string | number} UserId
 * @typedef {{ id: UserId, [key: string]: unknown }} UserRecord
 * @typedef {Record<string, unknown>} UserUpdates
 */
function cloneUser(user) {
  return { ...user };
}

function cloneUsers(users) {
  return users.map(cloneUser);
}

export default class Database {
  constructor() {
    this._history = [[]];
    this._currentIndex = 0;
  }

  /**
   * @returns {Array<UserRecord>}
   */
  getUsers() {
    return cloneUsers(this._getCurrentSnapshot());
  }

  /**
   * @param {UserRecord} user
   * @returns {void}
   */
  addUser(user) {
    const nextUsers = cloneUsers(this._getCurrentSnapshot());
    nextUsers.push(cloneUser(user));
    this._commitSnapshot(nextUsers);
  }

  /**
   * @param {UserId} id
   * @param {UserUpdates} updates
   * @returns {void}
   */
  updateUser(id, updates) {
    const currentUsers = this._getCurrentSnapshot();
    const userIndex = currentUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(currentUsers);
    const currentUser = nextUsers[userIndex];
    nextUsers[userIndex] = {
      ...currentUser,
      ...updates,
      id: currentUser.id,
    };

    this._commitSnapshot(nextUsers);
  }

  /**
   * @param {UserId} id
   * @returns {void}
   */
  deleteUser(id) {
    const currentUsers = this._getCurrentSnapshot();
    const userIndex = currentUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(currentUsers);
    nextUsers.splice(userIndex, 1);
    this._commitSnapshot(nextUsers);
  }

  /**
   * @returns {void}
   */
  undo() {
    if (this._currentIndex === 0) {
      return;
    }

    this._currentIndex -= 1;
  }

  /**
   * @returns {void}
   */
  redo() {
    if (this._currentIndex === this._history.length - 1) {
      return;
    }

    this._currentIndex += 1;
  }

  /**
   * @returns {Array<UserRecord>}
   */
  _getCurrentSnapshot() {
    return this._history[this._currentIndex];
  }

  /**
   * @param {Array<UserRecord>} users
   * @returns {void}
   */
  _commitSnapshot(users) {
    // Any new mutation after undo should drop the redo branch first.
    this._history = this._history.slice(0, this._currentIndex + 1);
    this._history.push(users);
    this._currentIndex = this._history.length - 1;
  }
}
```

This is the smallest and clearest implementation. The control flow is direct, and the history behavior is easy to follow because every step is a full snapshot. The exchange is memory: snapshots duplicate the top-level user list and records even when only one user changes.

This is usually the best default for interviews, local-only admin tools, and situations where simplicity matters more than squeezing out memory usage.

### Approach 2: Past / current / future snapshot stacks

Another option is to split snapshot state into three buckets:

- `past`: snapshots before the current one
- `current`: the active snapshot
- `future`: redoable snapshots

Each successful CRUD call computes a new snapshot, pushes the old current snapshot into `past`, promotes the new one into `current`, and clears `future`.

```jsx
type UserId = string | number;
type UserRecord = {
  id: UserId;
  [key: string]: unknown;
};
type UserUpdates = Partial<Omit<UserRecord, 'id'>>;

interface IDatabase {
  getUsers(): Array<UserRecord>;
  addUser(user: UserRecord): void;
  updateUser(id: UserId, updates: UserUpdates): void;
  deleteUser(id: UserId): void;
  undo(): void;
  redo(): void;
}

function cloneUser(user: UserRecord): UserRecord {
  return { ...user };
}

function cloneUsers(users: Array<UserRecord>): Array<UserRecord> {
  return users.map(cloneUser);
}

export default class DatabaseStacks implements IDatabase {
  _past: Array<Array<UserRecord>>;
  _current: Array<UserRecord>;
  _future: Array<Array<UserRecord>>;

  constructor() {
    this._past = [];
    this._current = [];
    this._future = [];
  }

  getUsers(): Array<UserRecord> {
    return cloneUsers(this._current);
  }

  addUser(user: UserRecord): void {
    const nextUsers = cloneUsers(this._current);
    nextUsers.push(cloneUser(user));
    this._commitSnapshot(nextUsers);
  }

  updateUser(id: UserId, updates: UserUpdates): void {
    const userIndex = this._current.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(this._current);
    const currentUser = nextUsers[userIndex];
    nextUsers[userIndex] = {
      ...currentUser,
      ...updates,
      id: currentUser.id,
    };

    this._commitSnapshot(nextUsers);
  }

  deleteUser(id: UserId): void {
    const userIndex = this._current.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const nextUsers = cloneUsers(this._current);
    nextUsers.splice(userIndex, 1);
    this._commitSnapshot(nextUsers);
  }

  undo(): void {
    if (this._past.length === 0) {
      return;
    }

    // The current snapshot becomes redo-able before restoring the latest past snapshot.
    this._future.push(this._current);
    this._current = this._past.pop() as Array<UserRecord>;
  }

  redo(): void {
    if (this._future.length === 0) {
      return;
    }

    this._past.push(this._current);
    this._current = this._future.pop() as Array<UserRecord>;
  }

  _commitSnapshot(users: Array<UserRecord>): void {
    // Every successful mutation becomes the new current snapshot and drops redo history.
    this._past.push(this._current);
    this._current = users;
    this._future = [];
  }
}
```

The main benefit here is that the data structure itself mirrors how people talk about undo/redo: past state, current state, and future state. The downside is that there are more moving pieces than in the single-history-array version, even though the core behavior is the same.

This approach is a good fit when the state picture should be especially explicit or easy to explain in product language.

### Approach 3: Command / inverse-operation stacks

Instead of storing full snapshots, reversible operations can be stored:

- `add`
- `update`
- `delete`

Each command contains enough data to either apply the change or undo it later. For example, an update command can store both the `before` and `after` user records.

```jsx
type UserId = string | number;
type UserRecord = {
  id: UserId;
  [key: string]: unknown;
};
type UserUpdates = Partial<Omit<UserRecord, 'id'>>;

interface IDatabase {
  getUsers(): Array<UserRecord>;
  addUser(user: UserRecord): void;
  updateUser(id: UserId, updates: UserUpdates): void;
  deleteUser(id: UserId): void;
  undo(): void;
  redo(): void;
}

type AddCommand = {
  type: 'add';
  user: UserRecord;
  index: number;
};

type UpdateCommand = {
  type: 'update';
  before: UserRecord;
  after: UserRecord;
  index: number;
};

type DeleteCommand = {
  type: 'delete';
  user: UserRecord;
  index: number;
};

type HistoryCommand = AddCommand | UpdateCommand | DeleteCommand;

function cloneUser(user: UserRecord): UserRecord {
  return { ...user };
}

function cloneUsers(users: Array<UserRecord>): Array<UserRecord> {
  return users.map(cloneUser);
}

// Undo works by replaying the opposite operation at the same recorded index.
function invertCommand(command: HistoryCommand): HistoryCommand {
  switch (command.type) {
    case 'add':
      return {
        type: 'delete',
        user: cloneUser(command.user),
        index: command.index,
      };

    case 'update':
      return {
        type: 'update',
        before: cloneUser(command.after),
        after: cloneUser(command.before),
        index: command.index,
      };

    case 'delete':
      return {
        type: 'add',
        user: cloneUser(command.user),
        index: command.index,
      };
  }
}

export default class DatabaseCommands implements IDatabase {
  _users: Array<UserRecord>;
  _undoStack: Array<HistoryCommand>;
  _redoStack: Array<HistoryCommand>;

  constructor() {
    this._users = [];
    this._undoStack = [];
    this._redoStack = [];
  }

  getUsers(): Array<UserRecord> {
    return cloneUsers(this._users);
  }

  addUser(user: UserRecord): void {
    const command: AddCommand = {
      type: 'add',
      user: cloneUser(user),
      index: this._users.length,
    };

    this._applyCommand(command);
    this._undoStack.push(command);
    this._redoStack = [];
  }

  updateUser(id: UserId, updates: UserUpdates): void {
    const userIndex = this._users.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const before = cloneUser(this._users[userIndex]);
    const after = {
      ...before,
      ...updates,
      id: before.id,
    };

    const command: UpdateCommand = {
      type: 'update',
      before,
      after,
      index: userIndex,
    };

    this._applyCommand(command);
    this._undoStack.push(command);
    this._redoStack = [];
  }

  deleteUser(id: UserId): void {
    const userIndex = this._users.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const command: DeleteCommand = {
      type: 'delete',
      user: cloneUser(this._users[userIndex]),
      index: userIndex,
    };

    this._applyCommand(command);
    this._undoStack.push(command);
    this._redoStack = [];
  }

  undo(): void {
    if (this._undoStack.length === 0) {
      return;
    }

    const command = this._undoStack.pop() as HistoryCommand;
    this._applyCommand(invertCommand(command));
    this._redoStack.push(command);
  }

  redo(): void {
    if (this._redoStack.length === 0) {
      return;
    }

    const command = this._redoStack.pop() as HistoryCommand;
    this._applyCommand(command);
    this._undoStack.push(command);
  }

  _applyCommand(command: HistoryCommand): void {
    // Commands carry enough data to rebuild the live list in either direction.
    switch (command.type) {
      case 'add':
        this._users.splice(command.index, 0, cloneUser(command.user));
        return;

      case 'update':
        this._users[command.index] = cloneUser(command.after);
        return;

      case 'delete':
        this._users.splice(command.index, 1);
        return;
    }
  }
}
```

This version can be more memory-efficient because it stores only the changed operation payloads instead of a full user list snapshot each time. The bookkeeping tradeoff: every command must carry enough information to be replayed and inverted correctly.

This approach is a strong fit when history is naturally action-based, when snapshots would be large, or when practicing reversible-command modeling.

## Edge cases

- `undo()` should be a no-op when there is no earlier history entry.
- `redo()` should be a no-op when there is no later history entry.
- A new CRUD change after `undo()` should clear redo history.
- `updateUser()` should preserve the original `id` and the user's position in the list.
- Same-value updates should still create distinct history steps.
- Only shallow cloning is required, so nested object and array values can remain shared references.

## Techniques

- Object-oriented programming
- Snapshot-based history
- Reversible command modeling

## Notes

- This question intentionally assumes valid IDs and unique user creation.
- Full snapshots are the best default for most interviews because they minimize branching logic.
- Command-based undo/redo can be more memory-efficient, but only if the extra bookkeeping is worth the added complexity.
- `getUsers()` returns shallow copies, so callers can mutate the returned array or user objects without rewriting the database's current snapshot.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
In a command-based implementation, a delete command stores only the deleted user. Its inverse calls ordinary `addUser()`. Explain two problems with that inverse and what information or internal operation it needs instead.

Your notes (optional)
