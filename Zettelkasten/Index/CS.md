---
aliases: ["#CS"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #CS
SORT file.mtime DESC
```