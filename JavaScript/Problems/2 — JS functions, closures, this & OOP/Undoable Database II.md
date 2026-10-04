---
title: Undoable Database II
aliases:
  - Undoable Database II
difficulty: Hard
source: GreatFrontEnd
url: "https://www.greatfrontend.com/questions/javascript/undoable-database-ii"
companies:
  - "[[Anthropic]]"
  - "[[Figma]]"
  - "[[Rippling]]"
pattern:
  - "[[Undo-Redo History]]"
concepts:
  - "[[Undo-Redo History]]"
  - "[[Transactions & Drafts]]"
  - "[[Object-Oriented Programming]]"
section: "2 — JS functions, closures, this & OOP"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Undoable Database II

> [!info] Problem
> Implement a class that manages draft user state and batch-based undo/redo checkpoints

## Problem

## Undoable Database II

This is a follow-up to [Undoable Database](/questions/javascript/undoable-database).

In many products, several CRUD changes should belong to one undo step. For example, a user editor may let someone create, update, and delete records freely while a dialog is open, but only commit the whole draft once the user confirms the changes.

In this question, implement a reusable `Database` class with that behavior:

- `addUser()`, `updateUser()`, and `deleteUser()` update a live draft immediately.
- `commit()` records the current draft as exactly one undo/redo history step.

Each user has an `id` and may contain arbitrary additional properties.

## Examples

```javascript
const database = new Database();

database.getUsers(); // []

database.addUser({ id: 1, name: 'Alice' });
database.updateUser(1, { role: 'admin' });

database.getUsers();
// [{ id: 1, name: 'Alice', role: 'admin' }]

database.commit();
database.getUsers();
// [{ id: 1, name: 'Alice', role: 'admin' }]

database.addUser({ id: 2, name: 'Bob' });
database.deleteUser(1);

database.getUsers();
// [{ id: 2, name: 'Bob' }]

database.undo(); // Discards the uncommitted draft.
database.getUsers();
// [{ id: 1, name: 'Alice', role: 'admin' }]

database.addUser({ id: 2, name: 'Bob' });
database.commit();

database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]

database.undo();
database.getUsers();
// [{ id: 1, name: 'Alice', role: 'admin' }]

database.redo();
database.getUsers();
// [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob' },
// ]
```

## Database API

Implement the following APIs on the `Database`:

### new Database()

Creates an instance of the `Database` class with one committed empty snapshot. User data and history are isolated within each `Database` instance.

### database.getUsers()

Returns the current visible users in insertion order.

If there is a pending draft started by `addUser()`, `updateUser()`, or `deleteUser()`, return that draft list. Otherwise return the current committed list.

Return a new array each time. User objects only need shallow copies.

### database.addUser(user)

Appends `user` to the live draft.

Repeated CRUD calls before `commit()` belong to the same batch, so they should not create multiple committed history steps.

If some committed batches had been undone before the first CRUD call in a new draft, all redo history should be discarded immediately when that new draft starts.

| Parameter | Type | Description |
| --- | --- | --- |
| `user` | `Object` | A user record with an `id` plus any additional properties. |

### database.updateUser(id, updates)

Shallow-merges `updates` into the matching user in the live draft.

Repeated CRUD calls before `commit()` belong to the same batch, so they should not create multiple committed history steps.

The user's original `id` should be preserved.

If some committed batches had been undone before the first CRUD call in a new draft, all redo history should be discarded immediately when that new draft starts.

| Parameter | Type | Description |
| --- | --- | --- |
| `id` | `string \| number` | The ID of the user to update. |
| `updates` | `Object` | Partial properties to merge into the stored user. |

### database.deleteUser(id)

Removes the matching user from the live draft.

Repeated CRUD calls before `commit()` belong to the same batch, so they should not create multiple committed history steps.

If some committed batches had been undone before the first CRUD call in a new draft, all redo history should be discarded immediately when that new draft starts.

| Parameter | Type | Description |
| --- | --- | --- |
| `id` | `string \| number` | The ID of the user to delete. |

### database.commit()

Commits the current draft as exactly one new history entry.

If there is no pending draft, this method should do nothing.

### database.undo()

Moves backward if possible.

If there is a pending draft, `undo()` should discard that draft and restore the latest committed list instead of traversing committed history.

Otherwise, if the database is already at the earliest committed entry, this method should do nothing.

### database.redo()

Moves forward one committed step if possible.

Discarded uncommitted drafts are **not** redoable.

If the database is already at the latest committed entry, this method should do nothing.

## Notes

- `commit()` after several CRUD calls should still create only one committed history step.
- A committed batch should still create a history step even if it ends on the same top-level user list it started with.
- You may assume `addUser()` is only called with a unique `id`, and `updateUser()` / `deleteUser()` are only called with existing IDs.
- You do not need to deep clone nested objects or arrays inside user records.

## Hints

### Hint 1 : What belongs to the draft?

### Hint 2 : When is redo history lost?

### Hint 3 : What does each exit from a draft do?

### Hint 4 : Can callers mutate stored state?

## Asked at these companies

Anthropic
Figma

## 🤔 Thought Process

- **Immediate Recognition:** Extension of [[Undoable Database]] introducing **two-phase staging / transaction semantics** (Draft vs Committed Checkpoints).
- **Core Problem:** CRUD operations no longer commit immediately. They update a pending "draft" state. History snapshots are only recorded on explicit `commit()`.
- **Draft Lifecycle Decisions:**
  1. Starting a new draft (`addUser`, `updateUser`, `deleteUser`) after an undo must immediately drop all redo history.
  2. `undo()` while a draft is active discards the uncommitted draft without moving the committed timeline pointer backward!
  3. `undo()` when NO draft is active moves backward along the committed history checkpoints.
  4. `commit()` when no draft is pending is a no-op; when a draft is active, it appends the draft to committed snapshots and resets the draft pointer to null.
- **State Representation:** Maintain `_history` (array of committed snapshots), `_currentIndex` (pointer in `_history`), and `_draft` (`Array<UserRecord> | null`).
- **Edge Cases:** Calling `commit()` multiple times in a row; calling `undo()` immediately after starting a draft; reading `getUsers()` when draft is pending (must reflect the uncommitted draft).

---

## 🧠 Mental Model

Think of this as **Git Staging Area & Commits**:
- `addUser` / `updateUser` / `deleteUser` = **Working Directory / Staged changes** (`_draft`).
- `commit()` = **git commit** (creates a permanent snapshot in `_history` and clears `_draft`).
- `undo()` with pending draft = **git restore** (discards unstaged working changes, reverts to latest commit).
- `undo()` without pending draft = **git checkout HEAD~1** (moves backward through committed snapshots).
- Starting work on a detached HEAD immediately prunes future redo commits.

---

## 🔑 Key Concepts

- [[Undo-Redo History]]
- Two-Phase Commit / Transactional Draft State
- Optimistic UI updates with deferred persistence
- Discarding pending transactions vs rewinding committed history
- Redo invalidation upon draft initiation (not deferred until commit)

---

## ⚠️ Edge Cases / Traps

- **When Redo History Drops:** Redo history MUST be dropped the moment the **first draft mutation occurs**, NOT when `commit()` is called.
- **Undo Behavior with Active Draft:** When `_draft !== null`, calling `undo()` resets `_draft = null` and leaves `_currentIndex` unchanged! It does not pop or rewind `_history`.
- **Multiple `commit()` Invocations:** If `_draft === null` (no changes since last commit), `commit()` must be a no-op; it should never append duplicate snapshots.
- **Active View Resolution (`getUsers`):** Must check `this._draft ?? this._history[this._currentIndex]`. If a draft exists, callers must see the draft.
- **Shallow Cloning Isolation:** User objects in the draft must not mutate previous committed snapshots when modified.

---

## ⭐ Interview Takeaway

1. **Separate Draft State from Committed History:** Model `_draft` as nullable (`null` when clean, `Array` when dirty). This cleanly separates pending changes from committed history checkpoints.
2. **Dual-Role `undo()`:** Be crystal clear with the interviewer: `undo()` first acts as a "Cancel Draft" button; only when clean does it act as "Step Backward in History".
3. **Early Redo Invalidation:** A common failure mode is waiting until `commit()` to truncate redo history. The requirement states redo is pruned as soon as the first edit of a new draft begins.
4. **Real-World Parallel:** This is the exact pattern behind modal edit forms, shopping cart checkouts, and spreadsheet cell staging.

---

## 🎯 Common Interview Questions

### Direct Questions
- How does `undo()` behave differently when a draft is in progress versus when the database is clean?
- Why must redo history be discarded on the first draft edit rather than waiting for `commit()`?
- What happens if `commit()` is called when no mutations have occurred?

### Follow-up Questions
- How would you implement an explicit `rollback()` or `discardDraft()` method?
- How would you support named checkpoints or savepoints (e.g. `savepoint('v1')`)?
- What would change if individual draft operations needed their own fine-grained undo within the dialog before committing?

### Conceptual Questions
- What is the difference between an optimistic draft pattern and an event-sourced aggregate?
- How does transactional state management relate to database ACID properties (specifically Atomicity and Isolation) in the frontend?

---

## 🔄 Variations

- **Nested Transactions / Savepoints:** Allowing drafts within drafts with hierarchical rollback.
- **Auto-Commit Timer (Debounced Checkpoints):** Automatically committing drafts after 1000ms of inactivity (common in collaborative editors).
- **Undo / Redo Manager II:** Applying this exact draft/commit model to single-value state managers.

---

## 📝 Revision Notes

- **Core idea:** Split state into committed history array + nullable pending draft. `undo()` cancels draft if present, else rewinds committed history.
- **Remember:** Invalidate redo history immediately on the first mutation that starts a draft (`_history = _history.slice(0, _currentIndex + 1)`).
- **Watch out for:** `undo()` must NOT decrement `_currentIndex` if a draft was discarded.
- **Complexity:** Time: $O(N)$ for mutations and commits due to cloning; Space: $O(C \times N)$ where $C$ is number of committed checkpoints.

## Official Solution
## Undoable Database II ( Official solution )

Premium
Languages
This follow-up adds one extra concept on top of the original undoable database: a difference between the live draft users the caller is editing right now and the committed checkpoints that should show up in undo/redo history.

That makes it a good fit for flows like edit dialogs, batched settings changes, or multi-step admin tools where the UI should update immediately but undo/redo should only traverse explicit commits.

## Solution

The database becomes much easier to follow when state is separated into:

1. Committed history that undo/redo traverses.
2. An optional draft user list that reflects the current in-progress batch.

### Approach 1: Committed snapshot history + current index + pending draft

This is the most interview-friendly model.

Use:

1. A `history` array containing committed user-list snapshots.
2. A `currentIndex` pointer to the active committed snapshot.
3. A `draftUsers` value plus a boolean that records whether a draft currently exists.

```javascript
history = [[], [{ id: 1, name: 'Alice' }]];
currentIndex = 1;
draftUsers = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
];
hasDraft = true;
```

This gives clean behavior:

- `getUsers()` returns `draftUsers` when a batch is in progress, otherwise `history[currentIndex]`.
- The first CRUD call in a new batch truncates history after `currentIndex`, so redo history is cleared immediately.
- Later CRUD calls in the same batch only update `draftUsers`.
- `commit()` appends the latest draft as exactly one new history step.
- `undo()` discards the draft first if one exists. Otherwise it moves `currentIndex` backward.

Redo only traverses committed snapshots. Once a draft exists, the visible users may differ from the committed checkpoint, but redo should not try to move through that draft layer.

The main transitions in the snapshot model are:

| Action | Draft exists? | History effect | Visible users afterward |
| --- | --- | --- | --- |
| First CRUD after commit | No | Truncate redo entries, clone current snapshot into draft | Draft |
| Later CRUD before commit | Yes | No new history entry | Updated draft |
| `commit()` | Yes | Append draft as one committed snapshot | New committed snapshot |
| `undo()` | Yes | Do not move history | Latest committed snapshot |
| `undo()` | No | Move `currentIndex` back if possible | Earlier committed snapshot |
| `redo()` | No | Move `currentIndex` forward if possible | Later committed snapshot |

```jsx
/**
 * @typedef {string | number} UserId
 * @typedef {{ id: UserId } & Record<string, unknown>} UserRecord
 * @typedef {Partial<Omit<UserRecord, 'id'>>} UserUpdates
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
    this._draftUsers = undefined;
    this._hasDraft = false;
  }

  /**
   * @returns {Array<UserRecord>}
   */
  getUsers() {
    return cloneUsers(this._getVisibleUsers());
  }

  /**
   * @param {UserRecord} user
   * @returns {void}
   */
  addUser(user) {
    const draftUsers = this._startDraft();
    draftUsers.push(cloneUser(user));
  }

  /**
   * @param {UserId} id
   * @param {UserUpdates} updates
   * @returns {void}
   */
  updateUser(id, updates) {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    const currentUser = draftUsers[userIndex];
    draftUsers[userIndex] = {
      ...currentUser,
      ...updates,
      id: currentUser.id,
    };
  }

  /**
   * @param {UserId} id
   * @returns {void}
   */
  deleteUser(id) {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    draftUsers.splice(userIndex, 1);
  }

  /**
   * @returns {void}
   */
  commit() {
    if (!this._hasDraft) {
      return;
    }

    this._history.push(this._draftUsers);
    this._currentIndex = this._history.length - 1;
    this._draftUsers = undefined;
    this._hasDraft = false;
  }

  /**
   * @returns {void}
   */
  undo() {
    // Undo during an active batch just discards the draft instead of committing it.
    if (this._hasDraft) {
      this._draftUsers = undefined;
      this._hasDraft = false;
      return;
    }

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

  _getVisibleUsers() {
    return this._hasDraft
      ? this._draftUsers
      : this._history[this._currentIndex];
  }

  _startDraft() {
    if (!this._hasDraft) {
      // Starting a new draft branch immediately invalidates redo history.
      this._history = this._history.slice(0, this._currentIndex + 1);
      this._draftUsers = cloneUsers(this._history[this._currentIndex]);
      this._hasDraft = true;
    }

    return this._draftUsers;
  }
}
```

This is the smallest implementation, has direct control flow, and is usually the easiest version to derive in an interview. In exchange, it stores full committed snapshots, which duplicates the top-level user list and user records each time a batch is committed.

It is usually the best fit for interviews, local-only admin tools, and products where history is user-list based and richer branching behavior is out of scope.

### Approach 2: Past / current / future committed snapshots + pending draft

Committed state can also be split into three buckets:

- `past`: committed snapshots before the current one
- `current`: the active committed snapshot
- `future`: redoable committed snapshots
- `draftUsers`: the in-progress batch, if any

This can feel natural because the data model matches the product language:

- The first CRUD call after an undo starts a draft and clears `future`.
- `commit()` pushes `current` into `past`, promotes `draftUsers` into `current`, and clears draft state.
- `undo()` either drops the draft or moves `current` into `future` while restoring the latest `past` snapshot.

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
  commit(): void;
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
  _draftUsers: Array<UserRecord> | undefined;
  _hasDraft: boolean;

  constructor() {
    this._past = [];
    this._current = [];
    this._future = [];
    this._draftUsers = undefined;
    this._hasDraft = false;
  }

  getUsers(): Array<UserRecord> {
    return cloneUsers(this._getVisibleUsers());
  }

  addUser(user: UserRecord): void {
    const draftUsers = this._startDraft();
    draftUsers.push(cloneUser(user));
  }

  updateUser(id: UserId, updates: UserUpdates): void {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    const currentUser = draftUsers[userIndex];
    draftUsers[userIndex] = {
      ...currentUser,
      ...updates,
      id: currentUser.id,
    };
  }

  deleteUser(id: UserId): void {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    draftUsers.splice(userIndex, 1);
  }

  commit(): void {
    if (!this._hasDraft) {
      return;
    }

    // Commit seals the whole draft as one history step.
    this._past.push(this._current);
    this._current = this._draftUsers as Array<UserRecord>;
    this._future = [];
    this._draftUsers = undefined;
    this._hasDraft = false;
  }

  undo(): void {
    if (this._hasDraft) {
      this._draftUsers = undefined;
      this._hasDraft = false;
      return;
    }

    if (this._past.length === 0) {
      return;
    }

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

  _getVisibleUsers(): Array<UserRecord> {
    return this._hasDraft
      ? (this._draftUsers as Array<UserRecord>)
      : this._current;
  }

  _startDraft(): Array<UserRecord> {
    if (!this._hasDraft) {
      this._draftUsers = cloneUsers(this._current);
      // A fresh draft after undo starts a new branch, so redo snapshots are dropped.
      this._future = [];
      this._hasDraft = true;
    }

    return this._draftUsers as Array<UserRecord>;
  }
}
```

This model reads almost like the product spec itself. `past`, `current`, `future`, and `draftUsers` make the behavior easy to explain and trace. The downside is that there are more moving parts to keep in sync than in the single-history-array version.

It works well when clarity of the state picture matters more than minimizing fields, or when people already describe undo/redo flows in terms of past, current, and future.

### Approach 3: Committed command batches + pending draft snapshot

Instead of storing full committed snapshots, committed batches of reversible operations can be stored:

- `add`
- `update`
- `delete`

Each batch contains the commands performed since the draft started. The live draft is still easiest to represent as a pending snapshot, but committed history is stored as grouped command batches.

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
  commit(): void;
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
type HistoryBatch = Array<HistoryCommand>;

function cloneUser(user: UserRecord): UserRecord {
  return { ...user };
}

function cloneUsers(users: Array<UserRecord>): Array<UserRecord> {
  return users.map(cloneUser);
}

function cloneCommand(command: HistoryCommand): HistoryCommand {
  switch (command.type) {
    case 'add':
      return {
        type: 'add',
        user: cloneUser(command.user),
        index: command.index,
      };

    case 'update':
      return {
        type: 'update',
        before: cloneUser(command.before),
        after: cloneUser(command.after),
        index: command.index,
      };

    case 'delete':
      return {
        type: 'delete',
        user: cloneUser(command.user),
        index: command.index,
      };
  }
}

function cloneBatch(batch: HistoryBatch): HistoryBatch {
  return batch.map(cloneCommand);
}

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
  _draftUsers: Array<UserRecord> | undefined;
  _hasDraft: boolean;
  _draftCommands: HistoryBatch;
  _undoStack: Array<HistoryBatch>;
  _redoStack: Array<HistoryBatch>;

  constructor() {
    this._users = [];
    this._draftUsers = undefined;
    this._hasDraft = false;
    this._draftCommands = [];
    this._undoStack = [];
    this._redoStack = [];
  }

  getUsers(): Array<UserRecord> {
    return cloneUsers(this._getVisibleUsers());
  }

  addUser(user: UserRecord): void {
    const draftUsers = this._startDraft();
    const command: AddCommand = {
      type: 'add',
      user: cloneUser(user),
      index: draftUsers.length,
    };

    this._applyCommand(command, draftUsers);
    this._draftCommands.push(cloneCommand(command));
  }

  updateUser(id: UserId, updates: UserUpdates): void {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    const before = cloneUser(draftUsers[userIndex]);
    const command: UpdateCommand = {
      type: 'update',
      before,
      after: {
        ...before,
        ...updates,
        id: before.id,
      },
      index: userIndex,
    };

    this._applyCommand(command, draftUsers);
    this._draftCommands.push(cloneCommand(command));
  }

  deleteUser(id: UserId): void {
    const visibleUsers = this._getVisibleUsers();
    const userIndex = visibleUsers.findIndex((user) => user.id === id);

    if (userIndex === -1) {
      return;
    }

    const draftUsers = this._startDraft();
    const command: DeleteCommand = {
      type: 'delete',
      user: cloneUser(draftUsers[userIndex]),
      index: userIndex,
    };

    this._applyCommand(command, draftUsers);
    this._draftCommands.push(cloneCommand(command));
  }

  commit(): void {
    if (!this._hasDraft) {
      return;
    }

    // Draft commands only become undo history after the batch is committed.
    this._users = cloneUsers(this._draftUsers as Array<UserRecord>);
    this._undoStack.push(cloneBatch(this._draftCommands));
    this._draftUsers = undefined;
    this._hasDraft = false;
    this._draftCommands = [];
  }

  undo(): void {
    if (this._hasDraft) {
      this._draftUsers = undefined;
      this._hasDraft = false;
      this._draftCommands = [];
      return;
    }

    if (this._undoStack.length === 0) {
      return;
    }

    const batch = this._undoStack.pop() as HistoryBatch;
    // Replay the batch backwards so stored indices still line up.
    for (let index = batch.length - 1; index >= 0; index -= 1) {
      this._applyCommand(invertCommand(batch[index]), this._users);
    }

    this._redoStack.push(cloneBatch(batch));
  }

  redo(): void {
    if (this._redoStack.length === 0) {
      return;
    }

    const batch = this._redoStack.pop() as HistoryBatch;
    batch.forEach((command) => {
      this._applyCommand(command, this._users);
    });

    this._undoStack.push(cloneBatch(batch));
  }

  _getVisibleUsers(): Array<UserRecord> {
    return this._hasDraft
      ? (this._draftUsers as Array<UserRecord>)
      : this._users;
  }

  _startDraft(): Array<UserRecord> {
    if (!this._hasDraft) {
      this._draftUsers = cloneUsers(this._users);
      // A fresh draft after undo starts a new branch, so redo batches are dropped.
      this._redoStack = [];
      this._draftCommands = [];
      this._hasDraft = true;
    }

    return this._draftUsers as Array<UserRecord>;
  }

  _applyCommand(command: HistoryCommand, users: Array<UserRecord>): void {
    switch (command.type) {
      case 'add':
        users.splice(command.index, 0, cloneUser(command.user));
        return;

      case 'update':
        users[command.index] = cloneUser(command.after);
        return;

      case 'delete':
        users.splice(command.index, 1);
        return;
    }
  }
}
```

This version can be more memory-efficient for committed history because it stores only the operations needed to replay or undo a batch, not a full committed snapshot every time. The bookkeeping tradeoff: each batch must preserve enough information to be replayed in order and inverted in reverse order.

This approach is a strong fit when history is naturally action-based, when committed snapshots would be large, or when practicing reversible-command modeling with batch semantics.

## Edge cases

- `commit()` with no pending draft should do nothing.
- Repeated CRUD calls before `commit()` should still produce only one committed history step.
- `undo()` during an active draft should discard the draft rather than auto-commit it.
- Starting a new draft after undo should clear redo history immediately.
- Batches that end on the same top-level user list they started with should still create a committed step.
- `updateUser()` should preserve the original `id` and the user's position in the list.
- Only shallow cloning is required, so nested object and array values can remain shared references.
- `getUsers()` should return fresh top-level arrays and user objects so callers cannot mutate stored history through a returned value.

## Techniques

- Object-oriented programming
- Separating draft state from committed checkpoints
- Managing history with snapshots or grouped reversible commands

## Notes

- This question intentionally keeps batching synchronous and explicit with `commit()`.
- It does not require timers, nested transactions, command exposure, or raw history inspection APIs.
- Showing the draft immediately is a product choice. Real systems sometimes keep pending changes hidden until commit time.

Check your understanding
Exercise 1 of 3
Beta
Check your understanding Exercise 1 of 3
A database starts every CRUD call by cloning the current committed snapshot, even when a draft already exists. Which workflow most directly exposes the lost draft state?
