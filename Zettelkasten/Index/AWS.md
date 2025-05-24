---
aliases: ["#AWS"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #AWS
SORT file.mtime DESC
```