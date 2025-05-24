---
aliases: ["#AI"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #AI
SORT file.mtime DESC
```