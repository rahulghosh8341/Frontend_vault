# CLAUDE.md — Frontend Knowledge Base Vault Assistant Instructions

This repository is an **Obsidian Knowledge Base** for frontend interview preparation (primarily based on GreatFrontend questions).

---

## 1. Vault Directory Structure

```text
Frontend_vault/
├── JavaScript/
│   ├── Companies/           # Company notes (e.g., Google.md, Meta.md) with Dataview queries
│   ├── Interview Questions/ # Conceptual quiz notes (e.g., Hoisting.md, var vs let vs const.md)
│   ├── Patterns/            # Reusable pattern notes (e.g., Array Traversal.md, DFS Recursion.md)
│   ├── Problems/            # Coding problem notes (e.g., Clamp.md, Flatten.md, Function Length.md)
│   ├── Concepts/            # Core JS concept notes
│   └── Dashboard.md         # Dataview overview dashboard
```

---

## 2. Mandatory Naming Conventions & Frontmatter Tracking

- **Filename** = Concise short alias (e.g., `Function Length.md`, `From Pairs.md`, `var vs let vs const.md`).
- **Frontmatter `title`** = Full GreatFrontend question text.
- **Backlinks**: Always use short alias WikiLinks (e.g., `[[Function Length]]`, `[[Array Traversal]]`, `[[Google]]`). Never use full questions in backlinks.
- **Mandatory Tracking Frontmatter**: Always include `solved`, `solvedDate`, and `type` in the frontmatter so `Dashboard.md` tracks progress automatically:

```yaml
# For coding challenges (JavaScript/Problems/):
---
title: <FULL QUESTION>
solved: true
solvedDate: YYYY-MM-DD
type: coding
---

# For quiz / interview questions (JavaScript/Interview Questions/):
---
title: <FULL QUESTION>
solved: true
solvedDate: YYYY-MM-DD
type: quiz
---
- **Learning Order Progress**: Whenever a question note (coding or quiz) is created or completed, automatically check it off in `Learning_Order.md` by changing `- [ ] **Question Name**` to `- [x] **Question Name**`.
```

---

## 3. Strict Rule: User Input Preservation

- **DO NOT overwrite or rewrite sections provided by the user** (such as their code, `Thought Process`, `Bugs I Made`, `Why This Works`, `Revision Notes`).
- Use the user's exact text for provided sections.
- Fill in missing required sections (e.g., frontmatter, `> [!info]` callout, `Key Facts`, `Common Interview Questions`, `Production Considerations`, backlinks) around the user's provided text.
- Treat the user's solution as the primary solution. Compare with official/alternative solutions without replacing the user's code.

---

## 4. Git & Push Rule

- **Do NOT push to GitHub automatically** unless the user explicitly asks you to push. Commit locally when asked, but wait for explicit instruction to push.

---

## 5. TWO NOTE TYPES & TEMPLATES (NEVER MERGE)

### TYPE 1 — CODING PROBLEM (Location: `JavaScript/Problems/`)

For coding challenges (e.g., `Flatten`, `Chunk`, `Clamp`, `Function Length`, `Function.prototype.apply`, polyfills).

**Rules:**
- Focus on user's thinking, solution, bugs, and revision.
- **DO NOT** create a `# Detailed Reference` section. Keep it concise (~1 page).

```md
---
title: <FULL GREATFRONTEND QUESTION>
aliases:
  - <short filename / alias>
difficulty: <Easy | Medium | Hard>
time: <duration, e.g., 15 min>
languages:
  - JavaScript
companies:
  - "[[Company]]"
pattern:
  - "[[Pattern]]"
concepts:
  - "[[Concept]]"
solved: true
solvedDate: YYYY-MM-DD
type: coding
---

> [!info]
> **Difficulty:** 🟢 Easy | **Time:** 15 min
> <One-line summary of problem>

## Problem

<Concise summary & example code>

## Companies

- [[Company]]

## Pattern

- [[Pattern]]

## 🤔 Thought Process

<User's reasoning>

## 💻 Final Solution

<User's submitted/final code>

## 🤔 Why This Works

<Explanation of JS behavior & edge cases>

## 🐞 Bugs I Made

<Actual mistakes made by user, or "None.">

## Production Considerations

<Native ES methods like Array.prototype.flat(), Object.fromEntries(), or performance notes>

## ⭐ Revision Notes

### Key Facts

- ...

### Common Interview Questions

- Question? → Answer

### Interview Takeaways

- ...

### Related

- [[Pattern]]
- [[Concept]]
- [[Related Problem]]
```

---

### TYPE 2 — INTERVIEW / QUIZ QUESTION (Location: `JavaScript/Interview Questions/`)

For theory/conceptual questions (e.g., `Hoisting`, `var vs let vs const`, `Event Loop`, `Closures`, `Execution Context`).

**Rules:**
- Quick revision top section + complete detailed reference below.

```md
---
title: <FULL GREATFRONTEND QUESTION>
aliases:
  - <short concept name>
tags:
  - javascript
  - interview
solved: true
solvedDate: YYYY-MM-DD
type: quiz
---

## TL;DR

<Very concise explanation>

## Interview Answer (30–60 sec)

<Concise verbal interview response>

## Key Takeaways

<Key points / comparison table>

## Visual Model

<Diagram or mental model if useful>

## Common Interview Traps

<Common misconceptions>

## Common Follow-ups

- ...

## My Notes

<Space for user notes>

# Detailed Reference

## <GreatFrontend section>

<Preserve GreatFrontend's complete useful explanation here for deep lookup>
```

---

## 6. Pattern Management Rules

1. When a coding problem is introduced, identify its pattern.
2. If an existing pattern fits (`[[Array Traversal]]`, `[[DFS Recursion]]`, `[[Fixed Size Grouping]]`, `[[Range Checking]]`), backlink to it.
3. If a **new reusable pattern** is introduced that does not exist in `JavaScript/Patterns/`:
   - Tell the user: `Pattern identified: [[Pattern Name]]`
   - Automatically create `JavaScript/Patterns/<Pattern Name>.md` using the Pattern Template below:

```md
---
aliases:
  - <Pattern Name>
---

## Core Idea
## Recognition
## Template
## Variations
## Complexity
## Common Mistakes
## Interview Tips
## Problems Using This Pattern
## Related Patterns
## Related Concepts
```

---

## 7. Execution Instructions for Claude Code CLI

- Write/edit files directly in the vault (`JavaScript/Problems/`, `JavaScript/Interview Questions/`, `JavaScript/Patterns/`).
- Use standard GitHub Flavored Markdown and Obsidian callouts (`> [!info]`).
- Do not run unnecessary terminal commands unless writing or updating vault files.
