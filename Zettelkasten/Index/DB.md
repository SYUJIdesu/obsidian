---
aliases: ["#DB"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #DB
SORT file.mtime DESC
```