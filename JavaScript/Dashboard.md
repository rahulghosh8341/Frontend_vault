# 🚀 Frontend Interview Dashboard

## 📊 Overall Progress

```dataviewjs
const codingTotal = 347;
const quizTotal = 283;

function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  try {
    if (app.plugins?.plugins?.dataview?.index) {
      app.plugins.plugins.dataview.index.reinitialize();
    }
  } catch (e) {}

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: {
          path: f.path,
          name: f.basename,
          folder: f.parent ? f.parent.path : ""
        },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const pages = getPages();
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

## 📂 Coding Progress by Section

```dataviewjs
const codingSections = [
  { name: "0 — HTML-CSS & UI warm-ups", total: 12 },
  { name: "1 — JS fundamentals, arrays & utilities", total: 44 },
  { name: "2 — JS functions, closures, this & OOP", total: 24 },
  { name: "3 — JS objects, recursion, data transformation & architecture", total: 48 },
  { name: "4 — Async JavaScript, promises & concurrency", total: 27 },
  { name: "5 — DOM, events & browser APIs", total: 25 },
  { name: "6 — React hooks & state", total: 28 },
  { name: "7 — React-UI fundamentals", total: 21 },
  { name: "8 — Advanced UI, accessibility & complex state", total: 26 },
  { name: "10 — Algorithms — arrays, strings, linked lists & basic trees", total: 71 },
  { name: "11 — Algorithms — trees, graphs, grids & advanced patterns", total: 21 }
];

function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: { path: f.path, name: f.basename, folder: f.parent ? f.parent.path : "" },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const codingPages = getPages().where(p => p.type === "coding");

const rows = codingSections.map(s => {
  const solved = codingPages.where(p => 
    p.section === s.name || (p.file.folder && p.file.folder.includes(s.name))
  ).length;
  const pct = s.total > 0 ? ((solved / s.total) * 100).toFixed(1) : "0.0";
  return [s.name, solved, s.total, Math.max(s.total - solved, 0), `${pct}%`];
});

dv.table(["Section", "Solved", "Total", "Remaining", "Progress"], rows);
```

## 🧠 Quiz Progress by Section

```dataviewjs
const quizSections = [
  { name: "1 — JavaScript core", total: 86, match: p => p.section === "1 — JavaScript core" || (p.file.folder && p.file.folder.endsWith("1 — JavaScript core")) },
  { name: "1 — JavaScript core — functions, OOP & patterns", total: 26, match: p => (p.section && p.section.includes("functions, OOP")) || (p.file.folder && p.file.folder.includes("functions, OOP")) },
  { name: "2 — Async JavaScript & networking", total: 20, match: p => (p.section && p.section.includes("2 — Async")) || (p.file.folder && p.file.folder.includes("2 — Async")) },
  { name: "3 — DOM, events & browser APIs", total: 42, match: p => (p.section && p.section.includes("3 — DOM")) || (p.file.folder && p.file.folder.includes("3 — DOM")) },
  { name: "4 — HTML & CSS", total: 35, match: p => (p.section && p.section.includes("4 — HTML")) || (p.file.folder && p.file.folder.includes("4 — HTML")) },
  { name: "5 — React fundamentals", total: 37, match: p => (p.section && p.section.includes("5 — React")) || (p.file.folder && p.file.folder.includes("5 — React")) },
  { name: "6 — React advanced & internals", total: 16, match: p => (p.section && p.section.includes("6 — React")) || (p.file.folder && p.file.folder.includes("6 — React")) },
  { name: "7 — Testing, security, performance & tooling", total: 21, match: p => (p.section && p.section.includes("7 — Testing")) || (p.file.folder && p.file.folder.includes("7 — Testing")) }
];

function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: { path: f.path, name: f.basename, folder: f.parent ? f.parent.path : "" },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const quizPages = getPages().where(p => p.type === "quiz");

const rows = quizSections.map(s => {
  const solved = quizPages.where(s.match).length;
  const pct = s.total > 0 ? ((solved / s.total) * 100).toFixed(1) : "0.0";
  return [s.name, solved, s.total, Math.max(s.total - solved, 0), `${pct}%`];
});

dv.table(["Section", "Solved", "Total", "Remaining", "Progress"], rows);
```

## 🔥 Study Streak

```dataviewjs
function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: { path: f.path, name: f.basename, folder: f.parent ? f.parent.path : "" },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const pages = getPages();
const dateSet = new Set();
for (const p of pages) {
  const d = p.solvedDate?.toISODate ? p.solvedDate.toISODate() : String(p.solvedDate).slice(0, 10);
  if (d) dateSet.add(d);
}

const dates = [...dateSet].sort();
const moment = window.moment;
const today = moment().startOf("day");

let currentStreak = 0;
let cursor = today.clone();

// If today hasn't been logged yet, keep the streak alive from yesterday
if (!dateSet.has(cursor.format("YYYY-MM-DD"))) {
  cursor.subtract(1, "day");
}

while (dateSet.has(cursor.format("YYYY-MM-DD"))) {
  currentStreak++;
  cursor.subtract(1, "day");
}

let longestStreak = 0;
let runningStreak = 0;
let previous = null;

for (const date of dates) {
  const current = moment(date).startOf("day");

  if (previous && Math.round(current.diff(previous, "days", true)) === 1) {
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
function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: { path: f.path, name: f.basename, folder: f.parent ? f.parent.path : "" },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const today = window.moment().format("YYYY-MM-DD");
const pages = getPages();

const todayPages = pages.where(p => {
  const d = p.solvedDate?.toISODate ? p.solvedDate.toISODate() : String(p.solvedDate).slice(0, 10);
  return d === today;
});

const coding = todayPages.where(p => p.type === "coding").length;
const quiz = todayPages.where(p => p.type === "quiz").length;

dv.table(
  ["💻 Coding", "🧠 Quiz", "📚 Total"],
  [[coding, quiz, coding + quiz]]
);
```

## 📈 Daily Activity

```dataviewjs
function getPages() {
  const dvPages = dv.pages('"JavaScript"').where(p => p.solved === true && p.solvedDate);
  if (dvPages && dvPages.length >= 20) return dvPages;

  const files = app.vault.getMarkdownFiles().filter(f => f.path.startsWith("JavaScript/"));
  const results = [];
  for (const f of files) {
    const cache = app.metadataCache.getFileCache(f);
    const fm = cache?.frontmatter;
    if (fm && (fm.solved === true || fm.solved === "true") && fm.solvedDate) {
      results.push({
        file: { path: f.path, name: f.basename, folder: f.parent ? f.parent.path : "" },
        section: fm.section || (f.parent ? f.parent.name : ""),
        solved: true,
        solvedDate: dv.luxon.DateTime.fromISO(String(fm.solvedDate)),
        type: fm.type
      });
    }
  }
  return results.length > 0 ? dv.array(results) : dvPages;
}

const pages = getPages();
const activity = {};

for (const page of pages) {
  const date = page.solvedDate?.toISODate ? page.solvedDate.toISODate() : String(page.solvedDate).slice(0, 10);
  if (!date) continue;

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
TABLE solvedDate AS "Solved Date", section AS "Section", type AS "Type"
FROM "JavaScript"
WHERE solved = true AND solvedDate
SORT solvedDate DESC
LIMIT 10
```

## 📝 Solved Question Format

Coding:
```yaml
---
section: "1 — JS fundamentals, arrays & utilities"
solved: true
solvedDate: 2026-09-03
type: coding
---
```

Quiz:
```yaml
---
section: "1 — JavaScript core"
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
