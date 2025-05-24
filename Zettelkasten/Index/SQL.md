---
aliases: ["#SQL"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #SQL
SORT file.mtime DESC
```