---
aliases: タグ一覧
created: 2025-05-24
updated: 2025-05-24
---

```dataviewjs
// DataviewJSを使用してデータを取得
const pages = dv.pages('""').where(p => p.tags);
const tagCounts = {};
const tagLatestDates = {};
let totalTags = 0;
let maxCount = 0;

// 全てのタグをカウントし、最新の日時を記録
pages.forEach(page => {
  if (page.tags) {
    page.tags.forEach(tag => {
      // タグのカウント
      if (tagCounts[tag]) {
        tagCounts[tag]++;
      } else {
        tagCounts[tag] = 1;
      }
      
      // タグの最新日時を更新
      const fileDate = page.file.mtime;
      if (!tagLatestDates[tag] || fileDate > tagLatestDates[tag]) {
        tagLatestDates[tag] = fileDate;
      }
      
      totalTags++;
    });
  }
});

// タグとカウントの配列に変換し、カウントの降順でソート
const sortedTags = Object.entries(tagCounts)
  .sort((a, b) => b[1] - a[1])
  .map(([tag, count]) => [tag, count, tagLatestDates[tag]]);

// 最大値を取得
if (sortedTags.length > 0) {
  maxCount = sortedTags[0][1];
}

// モダンなHTMLテーブルの生成
let output = "<div class='modern-tag-table'>";
output += "<table>";
output += "<thead><tr><th>タグ</th><th>個数</th><th>最終更新</th></tr></thead>";
output += "<tbody>";

// 行を追加
sortedTags.forEach(([tag, count, latestDate]) => {
  const percentage = Math.round((count / totalTags) * 100);
  const relativePercentage = Math.round((count / maxCount) * 100);
  // タグからハッシュタグ(#)を削除
  const tagName = tag.replace(/^#/, '');
  const tagLink = `Zettelkasten/Index/${tagName}`;
  
  // 日付フォーマット
  let formattedDate = '不明';
  if (latestDate) {
    // Dataviewの標準フォーマットを使用
    formattedDate = dv.date(latestDate).toISODate();
  }
  
  output += `<tr>
    <td><span class='tag-name'><a href="${tagLink}" class="internal-link" data-href="${tagLink}">${tag}</a></span></td>
    <td>${count}</td>
    <td>${formattedDate}</td>
  </tr>`;
});

output += "</tbody></table>";

// 統計情報
output += `<div class='tag-stats'>
  <span>${pages.length}ページ中でタグが使用されています</span>
  <span>${sortedTags.length}種類 / 合計${totalTags}個のタグ</span>
</div>`;

output += "</div>";

// HTMLを出力
dv.paragraph(output);
```
