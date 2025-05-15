---
title: "【2025年1月】Cursor Settings の設定と日本語訳について"
source: "https://zenn.dev/toono_f/scraps/6d2228eda9af1d"
author:
  - "[[Zenn]]"
published: 4ヶ月前にクローズ
created: 2025-05-15
description: "おとのさんのスクラップ"
tags:
  - "web"
  - "Cursor"
---
## Featuresの設定

## Cursor Tab

![](https://storage.googleapis.com/zenn-user-upload/19cdbc5d39b4-20250123.png)

### カーソルタブ（Cursor Tab）

- 複数行にわたる変更を提案できる強力なCopilotの代替ツールです。以前は「Copilot++」と呼ばれていました。

### 部分的な受け入れ（Partial accepts）

- 提案された次の単語を「⌘ + →」で受け入れることができます。

### カーソル予測（Cursor Prediction）

- カーソルタブの提案を受け入れた後、次に移動する行を予測します。タブで受け入れ、編集をタブでスムーズに切り替えることが可能です。

### コメント内の提案（Suggestions in Comments）

- コメント内でのカーソルタブ提案を有効または無効にする設定です。

### 空白のみの変更を表示（Show whitespace only changes）

- 空白のみの変更を含むカーソルタブ提案を表示する設定です。

### 自動インポート（Auto Import）

- 必要なモジュールをカーソルタブでタブ操作によりインポートできます（TypeScriptのみ対応）。

### Pythonの自動インポート（Auto Import for Python）（ベータ版）

- Pythonでの自動インポートを有効にします。

## Chat & Composer

![](https://storage.googleapis.com/zenn-user-upload/de634ec580bf-20250123.png)

### チャット & コンポーザー (Chat & Composer)

コードベースと対話しながら、複数のファイルをコンポーザーで一度に編集できます。

### 自動スクロール (Auto-scroll to bottom)

- 新しいメッセージが生成されると、コンポーザーパネルの一番下まで自動的にスクロールします。

### コンテキスト外のファイルへの自動適用 (Auto-apply to files outside context)

- コンポーザーが現在のコンテキスト外のファイルにも変更を自動的に適用することを許可します。

### YOLOモードを有効化 (Enable yolo mode)

- 確認なしでツールを実行するなど、エージェントコンポーザーが自由に動作できるようにします。

### エージェントによる編集の自動保存 (Auto save agentic edits)

- AIエージェントによる編集内容を自動的に保存します。これにより、LLMへのより正確な信号提供が可能になります。

### 入力ボックス内のピルを折りたたむ (Collapse input box pills in pane or editor)

- コンポーザーパネルやエディターの入力ボックス内のピルを折りたたんでスペースを節約します。

### ブロックの代わりにピルを表示 (Render pills instead of blocks)

- コンポーザーパネル内のコードブロックを折りたたみ、ピルとして表示します。

### リントの自動修正 (Agent composer iterate on lints)

- 有効にすると、エージェントコンポーザーがリントエラーに対して自動的に修正を試みます。

### 通常のコンポーザーによるリント修正 (Normal composer iterate on lints)（ベータ版）

- リントエラーがある場合、通常のコンポーザーも修正を試みます。

### 自動コンテキスト (Auto context)（ベータ版）

- コンポーザーに関連するコードベースのコンテキストを自動的に含めます。

### 変更のレビュー (Review changes)（ベータ版）

- コンポーザーセッション内で行われた変更をリスト化し、LLMを使用してレビューできます。

## YOLO Mode

![](https://storage.googleapis.com/zenn-user-upload/dbe03c4e91d5-20250123.png)

### YOLOモード

自動で各種コマンドを実行させることができるので、導入は慎重に考えたほうが良さそうです。

### YOLOプロンプト

- 使用しているモデルが判断する範囲で、自動的に実行されるべきコマンドの説明を入力してください。  
	例：「コンパイルコマンド、Gitコマンド、その他安全なコマンドのみ」

### コマンド許可リスト

- 特定のコマンドのみ自動的に実行されるようにしたい場合、ここに追加してください。

### コマンド禁止リスト

- 自動的に実行されてはならないコマンドをここに追加してください。

### ファイル削除保護

- 有効にすると、エージェントがファイルを自動的に削除することを防ぎます。

## Codebase Indexing

![](https://storage.googleapis.com/zenn-user-upload/568e2247f012-20250123.png)

### 概要

- 埋め込み（embeddings）は、コードベース全体の回答精度を向上させます。埋め込みとメタデータはクラウドに保存されますが、コード自体はローカルに保存されます。

### 状態

- 現在、同期済み（100%）。

### オプション

- **Resync Index** （インデックスを再同期）
- **Delete Index** （インデックスを削除）
- **Show Settings** （設定を表示）

## Docs

### 管理

- 追加したカスタムドキュメントを管理できます。
	- 現在、ドキュメントは追加されていません。
	- ドキュメントを開始するには、チャットまたは編集で「@Add」を入力してください。

### ボタン

- `+ Add new doc` （新しいドキュメントを追加）

## Editor

### Show chat/edit tooltip

- エディターでハイライトされたコード付近にチャット/編集ツールチップを表示します。

### Auto parse inline edit links

- 「/^# + K」入力時にリンクを自動的に解析します。

### Auto select for Ctrl/# + K

- インラインコード編集用に領域を自動的に選択します。

### Use themed diff backgrounds

- インライン差分用にテーマ付き背景色を使用します。

### Use character-level diffs

- インライン差分で文字単位の比較を使用します。

## Terminal

### Terminal hint

- ターミナルの下部にヒントテキストを表示します。

### Show terminal hover hint

- 「Add to chat」などのヒントをターミナルに表示します。

### Use preview box for terminal ⌘K

- 無効にすると、応答が直接シェルにストリーミングされます。

このスクラップは4ヶ月前にクローズされました

ログインするとコメントできます