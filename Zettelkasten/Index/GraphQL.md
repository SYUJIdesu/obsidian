---
aliases: ["#GraphQL"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #GraphQL
SORT file.mtime DESC
```