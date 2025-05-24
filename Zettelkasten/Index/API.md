---
aliases: ["#API"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #API
SORT file.mtime DESC
```