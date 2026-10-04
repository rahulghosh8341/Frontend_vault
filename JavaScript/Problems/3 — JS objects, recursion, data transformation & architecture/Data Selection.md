---
title: "Data Selection"
aliases:
  - "selectData"
  - "Data Selection"
difficulty: "Hard"
source: GreatFrontEnd
section: "3 — JS objects, recursion, data transformation & architecture"
solved: true
solvedDate: 2026-10-04
type: coding
---

# Data Selection

> [!info] Problem
> Implement a function to filter rows of data that match specified requirements

## Problem

## Data Selection

A data set of gym sessions looks like this:

```javascript
[
  { user: 8, duration: 50, equipment: ['bench'] },
  { user: 7, duration: 150, equipment: ['dumbbell', 'kettlebell'] },
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

Implement `selectData`, a function that returns sessions from the data. It has the interface `selectData(sessions [, options])`. Available options include:

- `user`: Select only sessions with this `id`. If not specified, include all users (subject to other filters).
- `minDuration`: Select only sessions with `duration` equal to or greater than this value. If not specified, include all sessions regardless of duration (subject to other filters).
- `equipment`: Select only sessions where at least one of the specified types of equipment was used. If not specified, include all sessions regardless of equipment used (subject to other filters).
- `merge`: If set to `true`
  - Sessions from the same `user` should be merged into one object. When merging:
    - Sum up the `duration` fields.
    - Combine all the `equipment` used, de-duplicating the values and sorting alphabetically.

  - Order merged rows by each user's **last position in the input array**. For example, the user sequence `8, 7, 1, 7, 7, 2, 2` becomes `8, 1, 7, 2`: user `7`'s last session appears after user `1`'s session.
  - The other filter options should be applied to the merged data.

Notes:

- When `merge` is not `true`, the returned sessions should remain in their original input order.
- The input objects should not be modified.

## Examples

The following examples use the data set above:

```javascript
selectData(sessions);
// [
//   { user: 8, duration: 50, equipment: ['bench'] },
//   { user: 7, duration: 150, equipment: ['dumbbell', 'kettlebell'] },
//   { user: 1, duration: 10, equipment: ['barbell'] },
//   { user: 7, duration: 100, equipment: ['bike', 'kettlebell'] },
//   { user: 7, duration: 200, equipment: ['bike'] },
//   { user: 2, duration: 200, equipment: ['treadmill'] },
//   { user: 2, duration: 200, equipment: ['bike'] },
// ];

selectData(sessions, { user: 2 });
// [
//   { user: 2, duration: 200, equipment: ['treadmill'] },
//   { user: 2, duration: 200, equipment: ['bike'] },
// ];

selectData(sessions, { minDuration: 200 });
// [
//   { user: 7, duration: 200, equipment: ['bike'] },
//   { user: 2, duration: 200, equipment: ['treadmill'] },
//   { user: 2, duration: 200, equipment: ['bike'] },
// ];

selectData(sessions, { minDuration: 400 });
// [];

selectData(sessions, { equipment: ['bike', 'dumbbell'] });
// [
//   { user: 7, duration: 150, equipment: ['dumbbell', 'kettlebell'] },
//   { user: 7, duration: 100, equipment: ['bike', 'kettlebell'] },
//   { user: 7, duration: 200, equipment: ['bike'] },
//   { user: 2, duration: 200, equipment: ['bike'] },
// ];

selectData(sessions, { merge: true });
// [
//   { user: 8, duration: 50, equipment: ['bench'] },
//   { user: 1, duration: 10, equipment: ['barbell'] },
//   { user: 7, duration: 450, equipment: ['bike', 'dumbbell', 'kettlebell'] },
//   { user: 2, duration: 400, equipment: ['bike', 'treadmill'] },
// ];

selectData(sessions, { merge: true, minDuration: 400 });
// [
//   { user: 7, duration: 450, equipment: ['bike', 'dumbbell', 'kettlebell'] },
//   { user: 2, duration: 400, equipment: ['bike', 'treadmill'] },
// ];
```

## Hints

### Hint 1 : Which rows should the filters inspect?

### Hint 2 : How can a merged row keep the latest position?

## Asked at these companies

Amazon
OpenAI
Tiktok
Stripe
Meta

## 🤔 Thought Process

- **Immediate Recognition:** Multi-criteria in-memory query engine (filtering, pagination, and sorting).
- **Options to Support:**
  - `user`: Filter rows matching specific user ID.
  - `minDuration`: Filter rows where `duration >= minDuration`.
  - `equipment`: Filter rows that contain *at least one* (or all, per spec) of the specified equipment items.
  - `merge`: If `true`, merge records for the same user using `mergeData` rules.
- **Pipeline Order Matters:**
  - Step 1: Filter raw sessions according to criteria (`user`, `minDuration`, `equipment`).
  - Step 2: If `merge: true`, run the merged aggregation over the filtered results.
  - Step 3: Return result.
- **Filtering Logic:**
  - User check: `options.user === undefined || session.user === options.user`.
  - Duration check: `options.minDuration === undefined || session.duration >= options.minDuration`.
  - Equipment check: Set intersection / `some()` check against required equipment list.

---

## 🧠 Mental Model

Think of a **Composable Query Execution Pipeline**:
```
Input Records
    │
    ▼
[ Filter Stage: Predicate evaluation (User, Duration, Equipment) ]
    │
    ▼
[ Transform / Aggregate Stage: (Conditional mergeData) ]
    │
    ▼
Output Results
```

---

## 🔑 Key Concepts

- Declarative data filtering (`Array.prototype.filter`)
- Array intersection checking (`Array.prototype.some` / `Set.prototype.has`)
- Composition of data pipelines (filtering then merging)
- Optional parameter normalization

---

## ⚠️ Edge Cases / Traps

- **Falsy Option Values:** `options.minDuration === 0` is valid! Don't check `if (options.minDuration)` because `0` evaluates to falsy. Always check `options.minDuration !== undefined`.
- **Empty Options Object:** `options = {}` should return the original data unmodified (or a shallow copy).
- **Equipment Matching Semantics:** Clarify whether `equipment` option means "contains ANY specified equipment" or "contains ALL specified equipment" (GFE specification tests ANY matching item).
- **Execution Order with `merge`:** Filtering MUST take place *before* merging; merging first would alter durations and equipment sets before evaluating filter predicates.

---

## ⭐ Interview Takeaway

- Treat filter options as composable predicates. Build a list of predicate functions or combine them into a single `filter()` pass for $O(N)$ filtering.
- Always use explicit `!== undefined` guards for numeric options to avoid the falsy `0` bug.
- Re-use your `mergeData` helper if `options.merge === true`.

---

## 🎯 Common Interview Questions

### Direct Questions
- "Why must filtering happen before merging instead of after?" (Merging sums durations and unions equipment; filtering after would filter aggregated metrics rather than individual session records).
- "How do you check if two arrays have at least one common element efficiently?" (Convert one array to a `Set` and check with `arr2.some(x => set.has(x))`).

### Follow-up Questions
- "How would you support custom sort criteria (`orderBy: { field, direction }`)?"
- "How would you optimize performance if queries are executed repeatedly over the same static dataset?" (Build inverted indexes on `user` and `equipment`).

### Conceptual Questions
- "How does this compare to Linq (C#) or SQL query planner execution?"

---

## 🔄 Variations

- **Data Merging:** The aggregation sub-routine used by this question.
- **Mini ORM / Query Builder:** Fluent chaining API (`db.select().where().orderBy()`).
- **Array.prototype.filter Polyfill:** Implementing the base filter mechanic.

---

## 📝 Revision Notes

- Clean filter predicate pattern:
```javascript
function selectData(sessions, options = {}) {
  const { user, minDuration, equipment, merge } = options;

  const equipSet = equipment ? new Set(equipment) : null;

  const filtered = sessions.filter(session => {
    if (user !== undefined && session.user !== user) return false;
    if (minDuration !== undefined && session.duration < minDuration) return false;
    if (equipSet !== null) {
      const hasMatch = session.equipment.some(item => equipSet.has(item));
      if (!hasMatch) return false;
    }
    return true;
  });

  if (merge) {
    return mergeData(filtered);
  }

  return filtered;
}
```

---

## Official Solution

## Data Selection ( Official solution )

Premium
Languages
This is a data-shaping question with two separate tasks: optionally merge multiple sessions for the same user, then filter the resulting rows by the requested options without mutating the input.

## Clarification questions

- What is the expected behavior if `options` contains `equipment: []`?
  - It should be treated as if `equipment` is not specified at all, but that case is not tested.

## Solution

The code is easiest to trace in two phases:

1. Build the list of session rows that filters should evaluate.
2. Filter that list against the requested options.

The order of those phases matters. When `merge: true`, the `user`, `minDuration`, and `equipment` filters are applied to the merged rows, not to the original individual sessions.

### Build the rows to evaluate

Without merging, this phase is mostly cloning into the internal shape used by later steps. Each row stores `equipment` as a `Set` so later overlap and de-duplication logic is simple.

With `merge: true`, the subtle part is preserving the required output position. The merged row for a user should appear where that user's latest session originally appeared.

A convenient trick is to reverse the input first. In that reversed view, the latest occurrence becomes the first occurrence, which means the merge can use the first row seen for each user and still land in the correct final position after reversing back.

The code uses:

- `sessionsProcessed` to hold the cloned rows that will eventually be filtered.
- `sessionsForUser` to map each user ID to its current merged row.

When a user appears for the first time in the reversed array, clone the session and store its equipment in a `Set`. When the same user appears again, update that stored row by adding duration and unioning equipment. After reversing `sessionsProcessed` back, the rows are in the correct output order.

On the sample data, the merge-order trick for users `7` and `2` looks like this:

| Reversed encounter | Action | Final row position |
| --- | --- | --- |
| latest user `2` session | create merged row for user `2` | latest user `2` position |
| earlier user `2` session | add duration/equipment to existing row | unchanged |
| latest user `7` session | create merged row for user `7` | latest user `7` position |
| earlier user `7` sessions | add duration/equipment to existing row | unchanged |

That is why merged rows appear where each user's latest original row appeared, not where the first row appeared.

### Filter the rows according to the options

Each option becomes a small predicate:

- `user`: exact match on the `user` field.
- `minDuration`: keep sessions whose `duration` is at least that threshold.
- `equipment`: keep sessions whose equipment overlaps with the requested equipment list.

Using `Set`s for equipment makes the overlap check simple and avoids nested `includes()` calls. The public output converts the internal equipment `Set` back into a sorted array so the result shape matches the input shape.

Filters compose with logical AND. For example, `{ merge: true, user: 7, minDuration: 400, equipment: ['bike'] }` should evaluate the merged row for user `7`, then keep it only if the summed duration and combined equipment satisfy both filters.

```jsx
function setHasOverlap(setA, setB) {
  // Bundler doesn't transpile properly when doing for-of for sets.
  for (const val of Array.from(setA)) {
    if (setB.has(val)) {
      return true;
    }
  }

  return false;
}

/**
 * @param {Array<{user: number, duration: number, equipment: Array<string>}>} sessions
 * @param {{user?: number, minDuration?: number, equipment?: Array<string>, merge?: boolean}} [options]
 * @return {Array}
 */
export default function selectData(sessions, options = {}) {
  // Merge from the end so reversing again restores the original user order.
  const reversedSessions = sessions.slice().reverse();
  const sessionsForUser = new Map();
  const sessionsProcessed = [];

  reversedSessions.forEach((session) => {
    if (options.merge && sessionsForUser.has(session.user)) {
      // Update the cloned session already tracked for this user.
      const userSession = sessionsForUser.get(session.user);
      userSession.duration += session.duration;
      session.equipment.forEach((equipment) => {
        userSession.equipment.add(equipment);
      });
    } else {
      const clonedSession = {
        ...session,
        equipment: new Set(session.equipment),
      };

      if (options.merge) {
        sessionsForUser.set(session.user, clonedSession);
      }

      sessionsProcessed.push(clonedSession);
    }
  });

  sessionsProcessed.reverse();

  const results = [];
  // Membership checks become O(1) regardless of how many equipment filters were passed in.
  const optionEquipments = new Set(options.equipment);
  sessionsProcessed.forEach((session) => {
    // Skip the session as soon as it fails any requested filter.
    if (
      (options.user != null && options.user !== session.user) ||
      (optionEquipments.size > 0 &&
        !setHasOverlap(optionEquipments, session.equipment)) ||
      (options.minDuration != null && options.minDuration > session.duration)
    ) {
      return;
    }

    results.push({
      ...session,
      // Convert the internal Set back to the public array shape.
      equipment: Array.from(session.equipment).sort(),
    });
  });

  return results;
}
```

## Common pitfalls

- Filtering before merging. With `merge: true`, filters must evaluate the summed duration and combined equipment for each user.
- Placing a merged row at the first occurrence of the user instead of the latest occurrence.
- Mutating the original session objects or their `equipment` arrays while merging.
- Returning `Set` objects in the final output instead of sorted equipment arrays.

## Notes

- `equipment: []` is treated like no equipment filter because the option `Set` has size `0`.
- The helper uses `Array.from(set)` instead of iterating a `Set` directly because the bundler does not transpile `for...of` over sets properly.

## Techniques

- Familiarity with JavaScript data structures like `Array`s and `Set`s.

Check your understanding
Exercise 1 of 2
Beta
Check your understanding Exercise 1 of 2
What does this call return?

```javascript
selectData(
  [
    { user: 1, duration: 100, equipment: ['bike'] },
    { user: 2, duration: 450, equipment: ['bench'] },
    { user: 1, duration: 350, equipment: ['rower'] },
  ],
  { merge: true, minDuration: 400 },
);
```
