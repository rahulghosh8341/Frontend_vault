# 🚀 Frontend Interview Dashboard

## 📊 Overall Progress

```dataviewjs
const codingTotal = 347;
const quizTotal = 283;

const pages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);

const coding = pages.where(p => p.type === "coding").length;
const quiz = pages.where(p => p.type === "quiz").length;
const total = coding + quiz;

dv.table(
  ["Category", "Solved", "Total", "Remaining", "Progress"],
  [
    ["💻 Coding", coding, codingTotal, Math.max(codingTotal - coding, 0), `${((coding / codingTotal) * 100).toFixed(1)}%`],
    ["🧠 Quiz", quiz, quizTotal, Math.max(quizTotal - quiz, 0), `${((quiz / quizTotal) * 100).toFixed(1)}%`],
    ["📚 Total", total, codingTotal + quizTotal, Math.max(codingTotal + quizTotal - total, 0), `${((total / (codingTotal + quizTotal)) * 100).toFixed(1)}%`]
  ]
);
```

## 🔥 Study Streak

```dataviewjs
const pages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);

const dateSet = new Set(
  pages.map(p => p.solvedDate.toISODate()).array()
);

const dates = [...dateSet].sort();
const moment = window.moment;
const today = moment().startOf("day");

let currentStreak = 0;
let cursor = today.clone();

while (dateSet.has(cursor.format("YYYY-MM-DD"))) {
  currentStreak++;
  cursor.subtract(1, "day");
}

let longestStreak = 0;
let runningStreak = 0;
let previous = null;

for (const date of dates) {
  const current = moment(date);

  if (previous && current.diff(previous, "days") === 1) {
    runningStreak++;
  } else {
    runningStreak = 1;
  }

  longestStreak = Math.max(longestStreak, runningStreak);
  previous = current;
}

dv.table(
  ["🔥 Current Streak", "🏆 Longest Streak", "📅 Study Days"],
  [[
    `${currentStreak} day${currentStreak === 1 ? "" : "s"}`,
    `${longestStreak} day${longestStreak === 1 ? "" : "s"}`,
    dates.length
  ]]
);
```

## 📅 Today's Progress

```dataviewjs
const today = window.moment().format("YYYY-MM-DD");

const pages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);

const todayPages = pages.where(
  p => p.solvedDate.toISODate() === today
);

const coding = todayPages.where(p => p.type === "coding").length;
const quiz = todayPages.where(p => p.type === "quiz").length;

dv.table(
  ["💻 Coding", "🧠 Quiz", "📚 Total"],
  [[coding, quiz, coding + quiz]]
);
```

## 📈 Daily Activity

```dataviewjs
const pages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);

const activity = {};

for (const page of pages) {
  const date = page.solvedDate.toISODate();

  if (!activity[date]) {
    activity[date] = { coding: 0, quiz: 0 };
  }

  if (page.type === "coding") {
    activity[date].coding++;
  } else if (page.type === "quiz") {
    activity[date].quiz++;
  }
}

const rows = Object.entries(activity)
  .sort(([a], [b]) => b.localeCompare(a))
  .map(([date, data]) => [
    date,
    data.coding,
    data.quiz,
    data.coding + data.quiz
  ]);

dv.table(
  ["Date", "💻 Coding", "🧠 Quiz", "📚 Total"],
  rows
);
```

## 🏢 Companies

```dataview
TABLE length(file.inlinks) AS Problems
FROM "JavaScript/Companies"
SORT length(file.inlinks) DESC
```

## 🧩 Patterns

```dataview
TABLE length(file.inlinks) AS Problems
FROM "JavaScript/Patterns"
SORT length(file.inlinks) DESC
```

## 🕐 Recently Solved

```dataview
TABLE solvedDate AS "Solved Date", type AS "Type"
FROM "JavaScript"
WHERE solved = true AND solvedDate
SORT solvedDate DESC
LIMIT 10
```

## 📝 Solved Question Format

Coding:
```yaml
---
solved: true
solvedDate: 2026-09-03
type: coding
---
```

Quiz:
```yaml
---
solved: true
solvedDate: 2026-09-03
type: quiz
---
```

## 📌 Tracking Rules

- Solving at least **1 question** on a calendar day counts as a study day.
- Multiple questions on the same day count as one streak day.
- Missing a day breaks the current streak.
- `solvedDate` is used for progress and streaks; `file.mtime` is not used.
- The dashboard updates automatically when question metadata is changed.
