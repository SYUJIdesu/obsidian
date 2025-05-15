---
title: "VSCode -> Cursor 移行メモ"
source: "https://zenn.dev/yusei_wy/scraps/351e4eadc8eaac"
author:
  - "[[Zenn]]"
published: 2024/09/03にコメント追加
created: 2025-05-15
description: "yusei_wyさんのスクラップ"
tags:
  - "web"
  - "Cursor"
---
1. VSCode ： `> Export Profile` でプロファイルをエクスポートする（ローカルファイル or gist）
2. Cursor ： `> Import Profile` を実行してエクスポートしたプロファイルを読み込み
3. これだけやっても拡張はインストールされない

`setting.json` やキーバインドは共有されているので、同期は諦めて個別に拡張機能をインストールすることにする

1. `code --list-extensions` で VSCode の拡張機能を列挙
2. `cursor --install-extension <name>` で拡張機能をインストール

これでようやく期待する動作に

おまけ  
shell script で自動化

```shell
#!/bin/bash

for extension in $(code --list-extensions); do
    cursor --install-extension "$extension"
done
```

ログインするとコメントできます