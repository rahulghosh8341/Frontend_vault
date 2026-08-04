# 🚀 Frontend Interview Dashboard

## Companies

```dataview
TABLE length(file.inlinks) AS Problems
FROM "JavaScript/Companies"
SORT length(file.inlinks) DESC
```

## Patterns

```dataview
TABLE length(file.inlinks) AS Problems
FROM "JavaScript/Patterns"
SORT length(file.inlinks) DESC
```

## Recently Solved

```dataview
LIST
FROM "JavaScript/Problems"
SORT file.mtime DESC
LIMIT 10
```