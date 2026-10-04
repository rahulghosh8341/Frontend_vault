---
title: "Data Merging"
aliases:
  - "mergeData"
  - "Data Merging"
difficulty: "Medium"
source: GreatFrontEnd
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Data Merging

> [!info] Problem
> Implement a function to merge rows of data from the same user

## Problem

## Data Merging

A data set of gym sessions looks like this:

```javascript
[
  { user: 8, duration: 50, equipment: ['bench'] },
  { user: 7, duration: 150, equipment: ['dumbbell'] },
  { user: 1, duration: 10, equipment: ['barbell'] },
  { user: 7, duration: 100, equipment: ['bike', 'kettlebell'] },
  { user: 7, duration: 200, equipment: ['bike'] },
  { user: 2, duration: 200, equipment: ['treadmill'] },
  { user: 2, duration: 200, equipment: ['bike'] },
];
```

Each session has the following fields:

- `user`: User ID of the session's user.
- `duration`: Duration of the session, in minutes.
- `equipment`: Array of equipment used during the session, in alphabetical order. There are only five different types of equipment.

Implement `mergeData`, a function that returns a unified view of each user's activities by merging each user's sessions. It has the interface `mergeData(sessions)`. Sessions from the same `user` should be merged into one object. When merging:

- Sum the `duration` fields.
- Combine all the `equipment` used, de-duplicating the values and sorting alphabetically.

The order of the results should always remain unchanged from the original set, and in the case of merging sessions with the same user, the row should take the place of the **earliest** occurrence of that `user`. The input objects should not be modified.

## Examples

The following example uses the data set above:

```javascript
mergeData(sessions);
// [
//   { user: 8, duration: 50, equipment: ['bench'] },
//   { user: 7, duration: 450, equipment: ['bike', 'dumbbell', 'kettlebell'] },
//   { user: 1, duration: 10, equipment: ['barbell'] },
//   { user: 2, duration: 400, equipment: ['bike', 'treadmill'] },
// ];
```

The data for users 7 and 2 is merged into each user's first occurrence.

## Hints

### Hint 1 : How can lookup and output order coexist?

### Hint 2 : What can safely accumulate equipment?

## Asked at these companies

OpenAI
Stripe
Meta
Coinbase

## 🤔 Thought Process

- **Immediate Recognition:** Aggregation / Group-by-and-Reduce problem over an array of tabular objects.
- **Core Requirements:**
  - Group sessions by `user` ID.
  - Sum the `duration` for each user.
  - Union and deduplicate `equipment` arrays, sorting them alphabetically.
  - Preserve the first encountered order or consistent ordering of users if required.
- **Algorithm Strategy:**
  - Use a hash map (`Map` or `{}`) keyed by `user` ID to accumulate values in $O(N)$ time.
  - For each session:
    - If user not in map: Initialize entry with `user`, `duration`, and a `Set` for `equipment`.
    - If user exists in map: Add `duration` to accumulator; add each equipment item to the user's `Set`.
  - Transform map values to array: Map over values, converting each user's equipment `Set` to a sorted array (`Array.from(set).sort()`).

---

## 🧠 Mental Model

Think of **SQL `GROUP BY user` with Aggregate Functions**:
```sql
SELECT user, SUM(duration), ARRAY_AGG(DISTINCT equipment ORDER BY equipment)
FROM sessions
GROUP BY user;
```
In JavaScript, an in-memory Hash Map + `Set` is the standard implementation for streaming group-by aggregations.

---

## 🔑 Key Concepts

- Hash Map indexing (`Map` for insertion-order preservation)
- `Set` for $O(1)$ deduplication
- Array reduction (`reduce` pattern)
- Lexicographical string sorting (`.sort()`)

---

## ⚠️ Edge Cases / Traps

- **Sorting Equipment:** Equipment arrays must be deduplicated *and* sorted alphabetically.
- **Mutating Input Data:** Do not mutate the original objects or arrays passed into `mergeData`.
- **Empty Equipment List:** Handle sessions where `equipment` is empty `[]` gracefully.
- **Single Session Users:** Users with only one session must still follow the output format (de-duplicated, sorted equipment).

---

## ⭐ Interview Takeaway

- Using a `Map` where each entry stores `{ user, duration, equipmentSet: new Set() }` guarantees insertion order while making lookups and deduplication $O(1)$.
- In the final serialization step, map over `map.values()` and convert `Array.from(equipmentSet).sort()`.
- Overall time complexity: $O(N \cdot K \log K)$ where $N$ is total sessions and $K$ is unique equipment items per user.

---

## 🎯 Common Interview Questions

### Direct Questions
- "How do you achieve deduplication when combining arrays in JavaScript?" (Using `new Set()`).
- "Why use `Map` over a plain JavaScript `{}` object here?" (`Map` preserves arbitrary key insertion order and avoids object prototype collisions).

### Follow-up Questions
- "What if the dataset is 1,000,000 items and doesn't fit in memory?" (External merge sort or stream-based chunked processing).
- "How would you make the aggregation rules dynamic (e.g. user specifies which field to sum vs union)?"

### Conceptual Questions
- "How does `Array.prototype.sort()` behave without an argument?" (Converts elements to strings and compares UTF-16 code units; safe for equipment name strings, unsafe for numbers).

---

## 🔄 Variations

- **Data Selection:** Filtering and sorting rows (SQL `WHERE` + `ORDER BY`).
- **Group By Utility:** Writing a general-purpose `groupBy(array, iteratee)` function.
- **Full Outer / Inner Join:** Merging two distinct datasets on a common foreign key.

---

## 📝 Revision Notes

- Clean and performant pattern:
```javascript
function mergeData(sessions) {
  const userMap = new Map();

  for (const session of sessions) {
    const { user, duration, equipment } = session;
    if (!userMap.has(user)) {
      userMap.set(user, {
        user,
        duration: 0,
        equipment: new Set(),
      });
    }
    const entry = userMap.get(user);
    entry.duration += duration;
    for (const item of equipment) {
      entry.equipment.add(item);
    }
  }

  return Array.from(userMap.values()).map(entry => ({
    user: entry.user,
    duration: entry.duration,
    equipment: Array.from(entry.equipment).sort(),
  }));
}
```

---

## Official Solution

## Data Merging ( Official solution )

Premium
Languages

## Solution

The task is grouping by `user` while preserving first-seen order. A `Map` gives fast lookup for users already seen, but `results` still owns the final order.

Maintain two structures while scanning the sessions from left to right:

1. `results`: the merged rows in output order.
2. `sessionsForUser`: a map from `user` to the same merged row that already lives inside `results`.

This keeps the main correctness rule simple: there is one output row per user, stored at the position of that user's first appearance.

When a user appears for the first time, clone the session, convert its `equipment` list into a `Set`, store that cloned row in the map, and append it to `results`:

```javascript
const clonedSession = {
  ...session,
  equipment: new Set(session.equipment),
};

sessionsForUser.set(session.user, clonedSession);
results.push(clonedSession);
```

Cloning matters because the input objects should not be modified. Using a `Set` internally makes later equipment duplicates easy to ignore while merging.

When the same user appears again, look up the existing row in O(1) time and update only the cloned accumulator:

```javascript
const userSession = sessionsForUser.get(session.user);
userSession.duration += session.duration;
session.equipment.forEach((equipment) => {
  userSession.equipment.add(equipment);
});
```

Because the map and `results` both point at the same merged object, updates automatically land in the earliest row for that user. Once the scan finishes, convert each equipment set back into a sorted array before returning.

For users `8, 7, 8`, the output rows are created when `8` and `7` first appear. The second `8` updates the first row in place through the map, so the final order stays `[8, 7]` even though user `8` appears again later.

With sessions `[{ user: 8, duration: 50, equipment: ['bench', 'dumbbell'] }, { user: 7, duration: 150, equipment: ['kettlebell'] }, { user: 8, duration: 50, equipment: ['bench'] }]`, the scan looks like this:

| Session | Map hit? | Result order | User row after step |
| --- | --- | --- | --- |
| user `8`, duration `50` | no | `[8]` | `{ duration: 50, equipment: {'bench', 'dumbbell'} }` |
| user `7`, duration `150` | no | `[8, 7]` | `{ duration: 150, equipment: {'kettlebell'} }` |
| user `8`, duration `50` | yes | `[8, 7]` | user `8` becomes duration `100`, same equipment set |

```jsx
/**
 * @param {Array<{user: number, duration: number, equipment: Array<string>}>} sessions
 * @return {Array<{user: number, duration: number, equipment: Array<string>}>}
 */
export default function mergeData(sessions) {
  const results = [];
  // Point each user id at the cloned session already stored in `results`.
  const sessionsForUser = new Map();

  sessions.forEach((session) => {
    if (sessionsForUser.has(session.user)) {
      const userSession = sessionsForUser.get(session.user);
      userSession.duration += session.duration;
      session.equipment.forEach((equipment) => {
        userSession.equipment.add(equipment);
      });
    } else {
      const clonedSession = {
        ...session,
        // Use a Set internally so repeated equipment is deduplicated while merging.
        equipment: new Set(session.equipment),
      };
      sessionsForUser.set(session.user, clonedSession);
      results.push(clonedSession);
    }
  });

  // Convert the internal Set back to the sorted array shape expected by callers.
  return results.map((session) => ({
    ...session,
    equipment: Array.from(session.equipment).sort(),
  }));
}
```

## Common pitfalls

- **Mutating the input sessions:** Avoid changing `session.duration` or pushing into `session.equipment`. The solution mutates only the cloned accumulator stored in `results`.
- **Reordering the output:** Sorting users or rebuilding the answer from sorted map keys loses the required first-appearance order. Append a row to `results` only when that user is first seen.
- **Returning `Set`s:** `Set` is an internal representation for merging. The returned objects still need `equipment` as arrays.
- **Forgetting the final sort:** Each input `equipment` array is already sorted, but equipment from later sessions can be inserted after larger values. Sort each merged equipment array before returning.

## Notes

- Empty input naturally returns an empty array.
- The final sort is cheap for this problem because there are only five equipment types. More generally, the sorting work depends on the size of each user's unique equipment set.
- Durations are accumulated by addition. The merge does not choose the longest session, average sessions, or preserve individual session rows.

## Techniques

- Familiarity with JavaScript data structures like `Array`s, `Map`s, and `Set`s.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
A refactor stores merged sessions in an ordinary object keyed by numeric user IDs, then returns `Object.values(byUser)`. The sums and equipment are correct. Which sequence of user IDs exposes an ordering bug?
