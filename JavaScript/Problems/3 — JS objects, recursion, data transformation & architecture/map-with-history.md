---
title: "Map With History"
aliases:
  - "MapWithHistory"
  - "Map With History"
difficulty: "Medium"
source: GreatFrontEnd
solved: true
solvedDate: 2026-10-04
type: coding
---

# Map With History

> [!info] Problem
> Implement a class that returns the latest value at or before a timestamp

## Problem

## Map With History

Implement a `MapWithHistory` class that can store multiple string values for the same key at different timestamps and read them back later.

When reading a key, return the value stored for the greatest timestamp that is less than or equal to the query timestamp.

A key may accumulate many historical values over time, but timestamps for the same key are always added in strictly increasing order.

## Examples

```javascript
const mapWithHistory = new MapWithHistory();

mapWithHistory.set('language', 'JavaScript', 1);
mapWithHistory.set('language', 'TypeScript', 4);

mapWithHistory.get('language', 1);
// 'JavaScript'

mapWithHistory.get('language', 3);
// 'JavaScript'

mapWithHistory.get('language', 4);
// 'TypeScript'

mapWithHistory.get('language', 10);
// 'TypeScript'
```

Queries earlier than the first stored timestamp should return `''`.

```javascript
const mapWithHistory = new MapWithHistory();

mapWithHistory.set('theme', 'dark', 5);

mapWithHistory.get('theme', 4);
// ''
```

## MapWithHistory API

### new MapWithHistory()

Creates a new empty `MapWithHistory` instance.

Stored values are isolated within each instance.

### mapWithHistory.set(key, value, timestamp)

Stores `value` for `key` at the given `timestamp`.

| Parameter | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to store a value for. |
| `value` | `string` | The string value to store. |
| `timestamp` | `number` | The timestamp associated with the value. |

### mapWithHistory.get(key, timestamp)

Returns the value stored for the largest timestamp associated with `key` such that `storedTimestamp <= timestamp`.

If no such value exists, return `''`.

| Parameter | Type | Description |
| --- | --- | --- |
| `key` | `string` | The key to read from. |
| `timestamp` | `number` | The timestamp to query. |

## Notes

- Timestamps for the same key are always added in strictly increasing order.
- Different keys may reuse the same timestamp.
- Only string values are in scope for this question.
- You do not need deletion, iteration, range queries, or out-of-order inserts.
- Inputs are guaranteed valid.

## Hints

### Hint 1 : What should one key store?

### Hint 2 : Which boundary answers a historical read?

## Asked at these companies

- [[Airbnb]]
- [[Anthropic]]
- [[OpenAI]]

## 🤔 Thought Process

- **Immediate Recognition:** Time-travel key-value store / Point-in-time lookup database.
- **API Requirements:**
  - `set(key, value, timestamp)`: Stores a value for `key` at `timestamp`. Timestamps for the same key are strictly increasing.
  - `get(key, timestamp)`: Returns the value for `key` with the largest stored timestamp $\le$ queried timestamp. If no such timestamp exists, returns `undefined`.
- **Data Structure Selection:**
  - Hash map storing an array of entries per key: `Map<string, Array<{ timestamp, value }>>`.
  - Since timestamps arrive in strictly increasing order, each key's array is **pre-sorted** by timestamp.
- **Query Optimization: Binary Search:**
  - To find the largest timestamp $\le$ target timestamp in a sorted array of size $N$, linear search is $O(N)$, but Binary Search achieves $O(\log N)$ time complexity.
  - Standard rightmost binary search / bisect right.

---

## 🧠 Mental Model

Think of a **Time Machine Snapshot Log**:
```
Timeline for Key 'balance':
  [t=10: $100]  ──►  [t=20: $150]  ──►  [t=50: $200]

Query get('balance', t=25):
  Find latest event at or before t=25 ──► Returns $150 (from t=20)
Query get('balance', t=5):
  No events exist at or before t=5   ──► Returns undefined
```

---

## 🔑 Key Concepts

- [[Undo-Redo History]]
- [[Range Checking]]
- [[Array Traversal]]
- Binary Search on monotonically increasing timestamps ($O(\log N)$)
- Time-series data modeling & Point-in-time queries

---

## ⚠️ Edge Cases / Traps

- **Query Timestamp Prior to First Entry:** If query timestamp is smaller than the earliest recorded timestamp for that key, return `undefined`.
- **Query Timestamp Exactly Matching an Entry:** Returns that exact entry's value.
- **Key Does Not Exist:** Return `undefined` immediately.
- **Monotonic Guarantee:** The problem states timestamps are added in strictly increasing order. If they weren't, `set` would need insertion sort or binary insert ($O(N)$).

---

## ⭐ Interview Takeaway

- Store entries as `Map<string, Array<{ timestamp, value }>>`.
- Always leverage the sorted timestamp property to implement Binary Search:
  - `get` complexity: $O(\log N)$ instead of $O(N)$.
  - `set` complexity: $O(1)$ amortized push.
- Classic question asked at top infrastructure and state-heavy companies (OpenAI, Anthropic, Airbnb).

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why can we use binary search in `get(key, timestamp)`?" (Because timestamps for any given key are strictly increasing, guaranteeing a sorted array).
- "What is the time complexity of `set` and `get`?" (`set` is $O(1)$; `get` is $O(\log K)$ where $K$ is the number of historical versions for that key).

### Follow-up Questions
- "What if timestamps can arrive out of order?" (Use binary search to find the insertion point, or store entries in a balanced binary search tree).
- "How would you implement range queries: `getRange(key, startTimestamp, endTimestamp)`?"

### Conceptual Questions
- "How does this relate to database systems like Git or MVCC (Multi-Version Concurrency Control) in PostgreSQL?" (MVCC stores historical row versions alongside transaction IDs/timestamps so transactions read a consistent snapshot).

---

## 🔄 Variations

- **Undo Redo Manager:** Tracking state history with rollback/redo pointers.
- **Time Map (LeetCode 981):** Identical problem statement in algorithmic format.
- **Snapshot Array:** Array supporting point-in-time snapshots.

---

## 📝 Revision Notes

- Binary search implementation:
```javascript
export default class MapWithHistory {
  constructor() {
    this.store = new Map();
  }

  set(key, value, timestamp) {
    if (!this.store.has(key)) {
      this.store.set(key, []);
    }
    this.store.get(key).push({ timestamp, value });
  }

  get(key, timestamp) {
    if (!this.store.has(key)) return undefined;

    const history = this.store.get(key);
    let low = 0;
    let high = history.length - 1;
    let result = undefined;

    while (low <= high) {
      const mid = Math.floor((low + high) / 2);
      if (history[mid].timestamp <= timestamp) {
        result = history[mid].value;
        low = mid + 1; // Try to find a later valid timestamp
      } else {
        high = mid - 1;
      }
    }

    return result;
  }
}
```

---

## Official Solution

## Map With History ( Official solution )

Premium
Languages

## Solution

The whole problem becomes much simpler once each key gets its own ordered history of timestamped values.

Because timestamps for the same key arrive in strictly increasing order, a good representation is:

```javascript
[
  { timestamp: 1, value: 'JavaScript' },
  { timestamp: 4, value: 'TypeScript' },
];
```

That sorted-history rule is what makes the API efficient:

- `set()` just appends to the end of the correct history array.
- `get()` only needs to find the latest entry whose timestamp is less than or equal to the query.

Both approaches below use the same state layout. The only real choice is how `get()` searches the history. For the best default solution to internalize, use the binary-search version.

The store itself should live on the instance, not in module scope, so separate `MapWithHistory` objects can hold different histories for the same key without interfering.

### Approach 1: Linear search from the end

Because the history is sorted, `get()` can scan backward from the latest entry until it finds the first timestamp that is less than or equal to the query.

```jsx
type TimeEntry = {
  timestamp: number;
  value: string;
};

interface IMapWithHistory {
  set(key: string, value: string, timestamp: number): void;
  get(key: string, timestamp: number): string;
}

export default class MapWithHistory implements IMapWithHistory {
  _store: Map<string, Array<TimeEntry>>;

  constructor() {
    this._store = new Map();
  }

  set(key: string, value: string, timestamp: number): void {
    let history = this._store.get(key);

    if (history == null) {
      history = [];
      this._store.set(key, history);
    }

    history.push({ timestamp, value });
  }

  get(key: string, timestamp: number): string {
    const history = this._store.get(key);

    if (history == null) {
      return '';
    }

    // History is append-only, so the first match scanning backward is the latest valid value.
    for (let index = history.length - 1; index >= 0; index -= 1) {
      const entry = history[index];

      if (entry.timestamp <= timestamp) {
        return entry.value;
      }
    }

    return '';
  }
}
```

Time complexity:

- `set()`: `O(1)` amortized
- `get()`: `O(n)` in the worst case for a key with `n` historical values
- Space: `O(n)` per key

This is a good first version to derive in an interview because it keeps the control flow obvious.

### Approach 2: Binary search

Since each key's timestamps are ordered, reads can be faster by binary-searching for the rightmost entry whose timestamp is less than or equal to the query timestamp.

For a history `[(1, 'draft'), (5, 'review'), (8, 'published')]`, the query result is always the value attached to the rightmost timestamp that does not exceed the query:

| Query timestamp | Rightmost timestamp `<= query` | Result |
| --- | --- | --- |
| `0` | none | `''` |
| `5` | `5` | `'review'` |
| `7` | `5` | `'review'` |
| `10` | `8` | `'published'` |

```jsx
export default class MapWithHistory {
  constructor() {
    this._store = new Map();
  }

  /**
   * @param {string} key
   * @param {string} value
   * @param {number} timestamp
   * @returns {void}
   */
  set(key, value, timestamp) {
    let history = this._store.get(key);

    if (history == null) {
      history = [];
      this._store.set(key, history);
    }

    history.push({ timestamp, value });
  }

  /**
   * @param {string} key
   * @param {number} timestamp
   * @returns {string}
   */
  get(key, timestamp) {
    const history = this._store.get(key);

    if (history == null) {
      return '';
    }

    let left = 0;
    let right = history.length - 1;
    let result = '';

    // Keep searching right after a match to find the latest entry at or before timestamp.
    while (left <= right) {
      const middle = Math.floor((left + right) / 2);
      const entry = history[middle];

      if (entry.timestamp <= timestamp) {
        result = entry.value;
        left = middle + 1;
      } else {
        right = middle - 1;
      }
    }

    return result;
  }
}
```

Time complexity:

- `set()`: `O(1)` amortized
- `get()`: `O(log n)` for a key with `n` historical values
- Space: `O(n)` per key

This is the recommended default implementation because it keeps the same simple write path while making lookups scale much better for large histories.

## Edge cases

- Missing keys and queries before the first timestamp return `''`.
- Exact timestamp matches should return that timestamp's value, not the previous one.
- Each key owns a separate history array, so writes to one key must not affect another key.
- Separate instances own separate `Map` objects, so identical keys in different instances can resolve to different values.
- `get()` should not mutate the stored history; repeated reads should behave identically.

## Notes

- Both approaches rely on the same core data model: one ordered history array per key.
- The only difference is how `get()` searches within that per-key history.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
The store appends each key's entries and uses binary search for reads because timestamps for that key arrive in strictly increasing order. A new ingestion source can deliver delayed entries out of order.

Would appending those entries and keeping the same read algorithm remain correct? Describe the invariant that must change or be restored.

Your notes (optional)
