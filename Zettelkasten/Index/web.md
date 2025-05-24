---
aliases: ["#web"]
---

```dataview
TABLE WITHOUT ID
  file.link as "📝 ノート",
  file.tags as "🏷️ タグ"
FROM #web
SORT file.mtime DESC
```