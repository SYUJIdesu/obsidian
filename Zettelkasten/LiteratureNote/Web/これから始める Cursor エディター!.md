---
title: これから始める Cursor エディター!
source: https://zenn.dev/owayo/articles/cbca6b558918bd
author:
  - "[[Zenn]]"
published: 2025-03-02
created: 2025-05-15
description: 
tags:
  - web
  - AI
---
124

43[tech](https://zenn.dev/tech-or-idea)

社内でCursor Businessを導入したので、利用方法や機能をまとめておきます。  
なお、Cursorは発展途上で、UI周りの機能変更が多いため、あくまでも参考程度にしてください。

## Cursorエディターとは

Cursorエディターは、AIを活用してソフトウェア開発をスピードアップしてくれる次世代エディターです。チャット感覚でコードの修正や提案を行ったり、複数行にわたるコード補完などをガッツリやってくれます。さらに、Cursor独自の **エージェント機能** で自動的にコードベースを解析してくれるので、ちょっとしたバグ修正からプロジェクト全体のビルドまでグイッとサポートしてくれます。

## Cursorのプラン

Cursorには、大きく3つのプランがあります。いずれも「 **プライバシーモード** 」を有効にすれば、コードは自分のマシンにしか保存されず、外部への共有や学習には使用されません。ここではざっくりと紹介します。

- **Hobby (無料)**  
	Hobbyプランは、基本無料で始められます。最初の2週間はPro版のトライアルも使えますし、補完を2000回、プレミアムモデルを50回まで使えます。とりあえず触ってみたい方にオススメです。
- **Pro ($20/月)**  
	Hobbyプランに加えて、無制限の補完と月500回の **高速プレミアムリクエスト** が利用できます。もし上限に達しても **低速リクエスト** などで継続してプレミアムモデルを使えるので、がっつりコーディングする人にぴったりです。
- **Business ($40/ユーザー/月)**  
	Proプランのすべての機能に加えて、組織全体での強制プライバシーモード適用やチームの利用状況の可視化、SSOなどの **エンタープライズ機能** が使えます。チーム開発でしっかり利用したいときに最適です。

## ダウンロード

以下のサイトから利用のOSにあったものをダウンロードしインストールしてください。

macOSの人は `brew` でもインストールできます

```
brew install --cask cursor
```

## 初回起動

初回起動時、キーバインド、AIで応答させる言語、コードベース(ソースコードの学習)に関する設定を行います。  
キーバインドはお好みで、 **Language for AI** だけ **日本語** に変更し、他はそのままで良いです。  
![](https://storage.googleapis.com/zenn-user-upload/527cd21b8b80-20250303.png)

## 日本語化

`F1` を押してコマンドパレットを開き、 `language` と入力して **Configure Display Language** を選択し、 **日本語** を選択します。  
(日本語が出てこない場合、少し待つと表示されます)  
Cursorを再起動することで反映されます。  
ただし、UIが完全に日本語化されるわけではありません。

## 基本設定

CursorはVSCode互換のエディターなのでVSCodeと同様の設定を適用できます。  
以下のサイトなどを参考に基本的な設定を済ませましょう。

## ショートカットキー

キーバインドは初回起動時に選んだエディターと同じなので、Cursorの特徴的なキーのみ記載します。  
VS Codeと同様ショートカットキーはカスタマイズできるので、使いにくい場合はカスタマイズしてください。

| ショートカット | 説明 |
| --- | --- |
| `⌘ + K` | インライン編集ウィンドウを開きます |
| `⌘ + Shift + K` | インライン編集ウィンドウの入力フォーカスを切り替えます   (**VS Codeの行削除と重複**) |
| `⌘ + L` | Chatウィンドウ(Ask)を開きます |
| `⌘ + I` | Chatウィンドウ(Agent)を開きます |
| 選択範囲して `⌘ + Shift + L` | 選択範囲を新しいChatに追加します |
| 選択範囲して `⌘ + L` | 選択範囲をChat(Ask)に追加します |
| 選択範囲して `⌘ + I` | 選択範囲をChat(Agent)に追加します |
| Chatウィンドウで `⌘ + N` | 新しいChatを作成します |

`⌘ + Shift + K` はVS Codeの行の削除と重複するため、VS Codeのキーバインドに慣れている方は変更したほうが使いやすいです。  
`Cursor` → `基本設定` → `キーボードショートカット` で、 `Open Composer as Bar` のキーバインドを削除するか他のキーに変更すると良いです。

## オススメ設定

### Cursor→基本設定→Cursor Settings

- **プライバシーモードOn**  
	General→Privacy mode  
	**enabled** にすると、Cursorにプロンプトの内容やシステムの情報が収集されないように設定できます。  
	Businessプランでは強制的にプライバシーモードが **enabled** に固定されるため考慮不要です。  
	![](https://storage.googleapis.com/zenn-user-upload/22b4a76e9fa3-20250303.png)
- **コードベースの自動インデックスOn**  
	Features→Codebase indexing  
	**Index new folders by default** にチェックを入れると自動でコードベースをインデックスするようになります（詳細後述）。
- **プロンプトの定型文**  
	Rules→User Rules  
	AIに投げる際のプロンプトの定型文を設定することができます。  
	AIでゴリゴリ書かれて自分の知識として吸収されない場合などは `生成したコードについて詳しく解説して` などと入れておけば詳細な解説を出してくれるようになります。  
	![](https://storage.googleapis.com/zenn-user-upload/dbfcf6168c5b-20250317.png)

### Cursor→基本設定→設定

- **アクティビティバーの縦表示** (VS Codeっぽい表示)  
	検索バーに `activity bar` と入力し、 **Workbench > Activity Bar: Orientation** の設定を探します。  
	**horizontal** から **vertical** に変更してCursorを再起動するとアクティビティバーが縦に表示されます。  
	VS Codeに慣れた人にオススメの設定です。

![](https://storage.googleapis.com/zenn-user-upload/446e0c32ab2a-20250305.png)

![](https://storage.googleapis.com/zenn-user-upload/f357f937581d-20250305.png)

### 万人にオススメしないけど個人的にオススメ設定

- Features→Chat  
	**Enable auto-run mode** にチェックを入れると、Agentでのコマンド実行に対して確認を行わず実行するようになります。

### 追加課金額設定 (Pro/Business限定)

この設定は以下の契約をしている方のみ設定変更可能です。

- Proプランで契約しているアカウント
- BusinessプランのAdmin権限をもったアカウント

[https://www.cursor.com/ja/settings](https://www.cursor.com/ja/settings) の **Usage-Based Pricing** にて、 **Monthly spending limit** に毎月追加課金で許可する金額を設定できます。  
Cursorは一部のプレミアムモデルや機能で追加課金が発生するため、追加課金をされたくない場合は$0にしておくと良いです。

![](https://storage.googleapis.com/zenn-user-upload/c760909ad71a-20250305.png)

### Claude 3.7 Sonnetが使い放題になる設定 (Pro/Business限定)

この設定は以下の契約をしている方のみ設定変更可能です。

- Proプランで契約しているアカウント
- BusinessプランのAdmin権限をもったアカウント

通常は **高速プレミアムリクエスト** を使い切ると **プレミアムモデル** は従量課金となりますが、現在 [https://www.cursor.com/ja/settings](https://www.cursor.com/ja/settings) の **Usage-Based Pricing** から、 **Enable for usage-based for premium models** を **Off** にするとClaude 3.7 Sonnetが使い放題になります。  
ただし、Slowリクエストとなり、そこそこの確率で失敗しリトライが必要になります。  
快適に使いたい人は従量制課金で使いましょう。

![](https://storage.googleapis.com/zenn-user-upload/e92509c53594-20250305.png)

## オススメ拡張

- [Cursor Stats](https://marketplace.cursorapi.com/items?itemName=Dwtexe.cursor-stats)  
	Cursorでのプレミアムリクエスト回数のステータスを画面下に表示してくれます。  
	管理画面で見ないとわからない値なので気軽に見れるのは便利！

## モデル

Cursorでは、いくつかのモデルを切り替えて利用できます。 **プレミアムモデル** はProやBusinessプランで毎月500回まで（高速リクエストとして）無料利用でき、超過した分は **従量課金** で追加購入が可能です。  
Hobbyプランの場合は **最大50回** までプレミアムモデルを無料トライアルとして利用できます（低速リクエストも含む）。  
プレミアムモデルには、 **GPT-4系（gpt-4o など） **や** Claude 3.7 Sonnet / 3.5 Sonnet** などが該当し、より高度な推論を扱えるのが特徴です。  
また、 **Claude 3.5 Haiku** はプレミアムモデル扱いですが、1回リクエストすると「1/3回」分としてカウントされるなど、ちょっとお得に使えるというイメージです。

下記は代表的なモデルの一覧と、その概要をまとめた表です。

| モデル名 | プロバイダー | プレミアム | Agent対応 | 料金 | 備考 |
| --- | --- | --- | --- | --- | --- |
| [`claude-3.7-sonnet`](https://www.anthropic.com/claude/sonnet) | [Anthropic](https://www.anthropic.com/) | **✓** | **✓** | $0.04 | 高度な推論が可能 |
| [`claude-3.5-haiku`](https://www.anthropic.com/claude/haiku) | [Anthropic](https://www.anthropic.com/) | **✓** |  | $0.01 | 1回のリクエストで「1/3回」分とカウント |
| `cursor-small` | Cursor |  |  | Free | 無料モデル（小〜中規模のコードにオススメ） |
| [`gpt-4o`](https://openai.com/index/hello-gpt-4o/) | [OpenAI](https://openai.com/) | **✓** | **✓** | $0.04 | 大きなコンテキストウィンドウに対応 |
| [`o1-mini`](https://openai.com/index/openai-o1-mini-advancing-cost-efficient-reasoning/) | [OpenAI](https://openai.com/) | **✓** |  | $0.10 | 1日10回の無料リクエスト付き |
| [`claude-3-opus`](https://www.anthropic.com/news/claude-3-family) | [Anthropic](https://www.anthropic.com/) | **✓** |  | $0.10 | 高速だが1日あたり10回まで |

モデルは、Cursor→Cursor Settings→Modelsで切り替えることができます。  
また、 [Models](https://docs.cursor.com/settings/models) のページにあるモデルは、Modelsの画面から **Add model** で追加して利用できます。  
Modelsで有効にしたモデルはChatウィンドウで切り替えて使用できます。  
基本的に常時 **claude-3.7-sonnet** で良いです。claude 3.7はダントツでソフトウェアエンジニアリングのスコアが高いので。

![](https://storage.googleapis.com/zenn-user-upload/6d324c34958b-20250321.png)  
[Claude 3.7 Sonnet and Claude Code](https://www.anthropic.com/news/claude-3-7-sonnet) から引用

### プレミアムモデルの500回リクエストを使い果たした場合

**Enable for usage-based for premium models** が **On** の場合は利用料に応じて追加課金が発生します。  
**Enable for usage-based for premium models** が **Off** の場合は、低速になりますがプレミアムモデルが利用できます。  
まれに、何回も再リクエストしないと使えないくらいになるので、快適に使用したい場合は追加課金で使えるようにしたほうが良いです。  
こんな感じに、 **Slow request** と表示されます。  
![](https://storage.googleapis.com/zenn-user-upload/68a7733b9308-20250316.png)

再リクエストが必要な場合はこんな感じになります。 **Try again** を何回クリックしてもリクエストが通らない時もあります。  
![](https://storage.googleapis.com/zenn-user-upload/a354f6f435b5-20250318.png)

オススメ拡張に書いた **Cursor Stats** をインストールしておけばステータスバーにプレミアムモデルリクエスト数が表示されるので便利です。  
![](https://storage.googleapis.com/zenn-user-upload/1b6b9e26d6d2-20250316.png)

## 機能

### Tab

Cursorのネイティブな **オートコンプリート機能** です。  
GitHub Copilotとの主な違いは:

- **マルチキャラクター編集**: Copilotはカーソル位置にテキストを挿入するだけですが、Cursorはカーソル周辺のコードを編集したり、テキストを削除したりすることもできます。
- **指示ベースの編集**: コメントや指示に基づいて編集提案をしてくれます。
- **履歴の活用**: あなたの最近の変更履歴をコンテキストとして使用し、次に何をしようとしているかを理解します。

`Tab` キーを押すと提案を受け入れ、 `Esc` キーで拒否できます。

![](https://storage.googleapis.com/zenn-user-upload/b9adeb3bc469-20250302.png)

### Ask

Cursorでは、コードやプロジェクト全体に対してチャットができる **チャット機能** があります。  
「このバグどこにあるの？」「どのように書き直すのが良い？」といった質問を投げかけると、AIがコードベースを理解したうえで答えてくれます。@や `⌘ + Shift + L` でコードを引用すると、ピンポイントで質問できるのでとても効率的です。さらに@Web(後述)を使うとウェブを検索して最新情報を考慮したりもできます。  
簡単な質問や単発のタスクに適しています。  
利用シーン:

- 簡単なQ&A
- コードの説明
- 小さな修正

![](https://storage.googleapis.com/zenn-user-upload/bfd38d44c22a-20250319.png)

### Agent

**Agent機能** を使うと、Cursorがエンドツーエンドでタスクを実行してくれます。  
実行されるコマンドや修正内容は確認してから適用できるので安心です。  
コードベースの変更やターミナルコマンドまで、まるっとカバーしてくれるので、とても開発が捗ります。  
複雑なタスクに適しています。  
利用シーン:

- 複数のファイルにまたがる変更
- 新機能の実装
- リファクタリング
- 完全なアプリケーションの作成

![](https://storage.googleapis.com/zenn-user-upload/5f16675a150e-20250319.png)

### Edit

Agentでは、コードの修正以外にターミナルの操作なども行われますが、コード修正に絞って実行してほしい場合に **Edit** を利用します。  
こちらも複雑なタスクに適しています。  
利用シーン:

- 複数のファイルにまたがる変更
- 新機能の実装
- リファクタリング

![](https://storage.googleapis.com/zenn-user-upload/a702bf839065-20250319.png)

### Command + K

エディター内で `⌘ + K` を押すと、AIによるコード作成や編集ができます。  
たとえば、コードを一部選択して `Edit` を押し、  
「ここの変数名をもう少しわかりやすい名前に変更して」  
のように書くと、Cursorが一発で修正案を提示してくれます。  
新しいコードを生成したいときは、何も選択せず `⌘ + K` を押して、やりたいことを説明すればOKです。

### Terminal Command + K

エディター内のターミナル(表示→ターミナル)でも `⌘ + K` を使うと、自然言語で端末コマンドを書けます。  
たとえば、

```bash
npmでReactプロジェクトを新規作成して
```

と入力すると、

```bash
npx create-react-app my-app
```

のようなコマンドを自動生成してくれます。いちいち調べなくても簡単にコマンドを書いてくれるので地味に助かります。  
以下のようにターミナルで `⌘ + K` で呼び出して指示するとコマンド生成してくれます！ `xargs` などを駆使した複雑なコマンドも作ってくれるのでかなり便利です。

![](https://storage.googleapis.com/zenn-user-upload/6c096a1e1282-20250305.png)  
![](https://storage.googleapis.com/zenn-user-upload/23e75ba696dc-20250305.png)

### Bug Finder

修正内容にバグがないか確認できます。(現在ベータ機能)  
ベータ機能のためかスキャン結果はイマイチです。今後のアップデートに期待。  
アクティビティバーから **Bug Finder** をクリックし、 `New Run...` をクリックすることでバグのスキャンを行えます。

![](https://storage.googleapis.com/zenn-user-upload/956d4cc7e463-20250303.png)

ここで注意が必要なのですが、 **Run First** は無料ですが、 **Run Smart** は追加課金の機能となります。  
**Run Smart** をクリックしてしまうと確認無しでBug Finderが動きます。  
スクショの通り、どんなに小さいファイルを選択しても、$2.7ほどかかります。

![](https://storage.googleapis.com/zenn-user-upload/c1311d357834-20250303.png)

使い方としては、レビュー時などに対象リポジトリをチェックアウトし、 **Advanced Configration** で、 **Base Ref** にターゲットブランチ名やcommitのHashを入れてバグチェックするという感じですかね。

![](https://storage.googleapis.com/zenn-user-upload/a68c8efd97a2-20250303.png)

ちなみに **Run Smart** を実行してしまうと **追加課金設定で、上限を $0 に設定してあっても課金されます** 。

![](https://storage.googleapis.com/zenn-user-upload/48a90b3affe8-20250321.png)

### MCP

Cursorは、Anthropicが提唱したMCPにも対応しています。  
Cursor SettingsのMCPから `Add new MCP server` をクリックしてMCPごとに設定することでMCPサーバを追加できます。  
![](https://storage.googleapis.com/zenn-user-upload/fffc2b60b5f4-20250304.png)

プロジェクトごとにMCP設定を使用する場合は、`.cursor/mcp.json` に、Claude Desktopの `claude_desktop_config.json` と同様の記法で定義できます。  
なお、 **MCPはAgentでのみ利用できます** 。

### @シンボル

Cursorでは、チャットやエージェントへの指示の中で `@` シンボルを使うと、いろんな対象を指定してAIに教えたり、コードを参照してもらったりできます。以下が主な使い方です。

- **@Files**  
	ファイル名を直接指定して、「@App.tsx」のように書くと、特定のファイルの内容をコンテキストに含めてAIに処理させることができます。たとえば「@App.tsx の関数fetchDataをもっと高速化できる？」と聞くと、AIがそのファイルを参照して具体的な改善案を提案してくれます。  
	Cursorの左のファイルツリーでファイルを右クリックして、 `Add Files to Cursor Chat` や `Add Files to New Cursor Chat` でもチャットに追加できます。
- **@Code**  
	直接的にコードブロックを示したいときに使います。エディター上でコードを選択して `⌘ + Shift + L` または `⌘ + L` を押すと、選択範囲を@Codeでチャットに追加してくれます。
- **@Doc**  
	ライブラリのリファレンスやAPIドキュメントを参照したいときに使うと便利です。「@React xxxについて教えて」のように書くと、あらかじめ登録しておいたドキュメントやライブラリ情報をAIが参照できます。メジャーなリファレンスはすでに登録されていますが、必要があればプロジェクトごとに `@Docs` → `Add new doc` でライブラリを登録しておくと、会話の中でササッと呼び出せるようになります。
- **@Web**  
	インターネット上の検索結果を活用した質問やリファレンス取得ができるので、「@Web starが一番多いxxxをするライブラリを調べて」と書くだけでウェブ検索の内容を考慮して回答を提示してくれます。
- **@Link**  
	明示的にURLを示しておきたいときに使います。URLを貼り付けると自動で @Link となり、AIがそのページを参照してアドバイスしたりできます。  
	貼り付けたURLを@Linkにしたくないときは、@Linkをクリックして **Unlink** を選択してください。

### コードベース

Cursorでは、自分のプロジェクト全体を **コードベース** として扱うことができます。チャット機能やエージェント機能がコードベースを自動で解析してくれるので、「どのファイルで何やってるの？」とか「このバグの原因はどこ？」といった感じで気になるところを質問できます。  
初回のみコードベースは手動で作成する必要があります。コードベースは、Cursor→Cursor Settings→Features→Codebase indexingにて、Compute Indexをクリックすると自動でインデックスが作成されます。  
**Index new folders by default** にチェックを入れると、フォルダーを開くと自動でその配下のインデックスが作成されるようになります。とりあえずチェックしておくのをオススメします。

![](https://storage.googleapis.com/zenn-user-upload/44fcf94fb1c5-20250302.png)

具体的には、

- 「この関数、もっと簡潔に書ける？」
- 「このバグはどのファイルのどこで起きてる？」
- 「この処理を実行すると何が返る？」  
	といった質問を投げると、Cursorが自動的にコードを検索して回答してくれます。

また、エージェント機能で自動的にコードを修正したり、ターミナルコマンドを叩いたりしてくれるので、開発の流れがスムーズに進みます。自分で調べる手間が減るのはとても便利です。

### ルール

Cursorでは、AIとの対話をより効果的にするための **ルール** を設定できます。これらのルールは、AIがコードを生成したり、質問に回答したりする際の指針となります。

記述すると良いルールの内容：

- **プロジェクト固有のルール**: プロジェクトのコーディング規約やアーキテクチャに関するガイドラインをAIに伝えることができます。
- **コードスタイルの指定**: インデントのスタイル、命名規則、コメントの書き方などを定義できます。
- **技術スタックの指定**: 使用しているフレームワークやライブラリ、バージョンなどを指定できます。
- **禁止事項の設定**: 特定のAPIや手法の使用を避けるよう指示できます。
- **テスト要件**: コード生成時にテストも含めるかどうかなどを指定できます。

ルールは、プロジェクトのルートディレクトリに`.cursor/rules` 配下にファイルを作成して設定できます。

たとえば、次のようなルールを設定できます：

- 「すべてのReactコンポーネントは関数コンポーネントとして実装し、TypeScriptを使用すること」
- 「コメントは日本語で書き、各関数には必ずJSDocを付けること」
- 「テストはJestを使用し、各コンポーネントには少なくとも基本的なレンダリングテストを含めること」
- 「依存性の注入やサービスコンテナーを優先して利用すること」
- 「重複を避け、反復処理やモジュール化を優先すること」
- 「Python 3.12を使用すること」

適切なルールを設定することで、AIからより一貫性のある、プロジェクトに適したコードを生成してもらうことができます。

**Cursor 0.47.5にてRule Type機能が実装されました**

- Always：常に適用
- Auto Attached：globでのマッチングでルールを適用
- Agent Requested：Agentが適用を判断
- Manual：手動  
	![](https://storage.googleapis.com/zenn-user-upload/bf9caee45637-20250317.png)

私は全体的なコーディングルールは **Always** にしておき、特定のパスや拡張子に紐づくようなフレームワークの情報等は **Auto Attached** にするという感じで作っています。  
またリファクタリング時など、コーディング規約を改めて正確に適用してほしいときなどは `@` シンボルを利用して、ルールファイルを明示して指示するようにしています。

なお、`.cursorrules` ファイルによるルール指定が以前は可能でしたが、現在は非推奨となっていますので、`.cursor/rules` を使用してください。

### .cursorignore

Cursorでは、`.cursorignore` ファイルを使用して、Cursorのコード生成やChatの対象外にするファイルを指定できます。  
デフォルトで`.gitignore` ファイルは無視されるため、`.gitignore` ファイルに記載した内容は`.cursorignore` ファイルに記載する必要はありません。  
`.cursorignore` ファイルは、プロジェクトのルートディレクトリに配置します。  
`.gitignore` と同じシンタックスで記載します。

### Git

VS Code同様、git操作が可能です。

#### git がローカルマシンにセットアップされていない場合

ローカルマシンに git が入っていない場合は git をインストールします。

macOSの人は brew でもインストールできます

```
brew install git
```

インストール後、git の設定を行います

```
git config --global user.name 'username'
git config --global user.email 'useremail@example.com'
```

#### 操作方法

左のアクティビティバーから、gitのアイコンをクリックするとgit操作ができる画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/24687e4097a0-20250407.png)

修正をステージするには、対象のファイルの **+** をクリックします。

![](https://storage.googleapis.com/zenn-user-upload/380488b95be0-20250407.png)

ステージされた状態はこちら。

![](https://storage.googleapis.com/zenn-user-upload/94453a28541a-20250407.png)

星のアイコンをクリックするとステージの内容からコミットメッセージを自動生成してくれます。

![](https://storage.googleapis.com/zenn-user-upload/6c6887941b89-20250407.png)

冗長なメッセージになりがちなので適宜調整したほうが良いです。

![](https://storage.googleapis.com/zenn-user-upload/09c28478f51f-20250407.png)

コミットボタンを押すとコミットログが追加されます。  
ソース管理グラフで、右から2つ目のアイコンをクリックするとpushできます。

![](https://storage.googleapis.com/zenn-user-upload/56ab3cab73f5-20250407.png)

リベースなどは、**...** のメニューから行うことができます。

![](https://storage.googleapis.com/zenn-user-upload/ef9731283d9c-20250407.png)

## Ask / Agent / Edit どれつかったらいいの？

個人的には常時Agentで問題無いと思っています。  
Agentが自動で判断して適用するルールがあったり、MCPがAgentでしか使えないというのもありますし、常時Agentで困ったことが無いです。  
社内で他の人にも聞いてみましたが、以下のようなアプローチがあるとのことでした。

- Askは使用しない。基本的にAgentを利用。現在のライブラリのままリファクタリングしてほしいときなど、Agentにすると勝手にライブラリを追加されるので、意図してコードだけを修正してほしいときはEditを使用する

一応Cursorのサイトにある画像を用いたユースケースはこんな感じです

![](https://storage.googleapis.com/zenn-user-upload/567516601e7b-20250319.png)

## AIによるコーディングのやりかた

AskまたはAgentにてコーディングの依頼を出し、コードの生成が終わったら以下のいずれかにて修正を適用します。

- ① Chatウィンドウの **Accept** (すべての修正を受け入れる)  
	また、 **Reject** ですべての修正を拒否できます。
- ② エディタウィンドウの **Accept file** (ファイル単位で修正を受け入れる)  
	また、 **Revert file** でファイル単位で修正を拒否できます。
- ③ エディタウィンドウの修正ブロックの **Accept** (ブロック単位で修正を受け入れる)  
	また、 **Reject** でブロック単位で修正を拒否できます。

![](https://storage.googleapis.com/zenn-user-upload/3d8a57e02c11-20250302.png)

なお、 **AIが行った修正は画面からAccept/Rejectを選択できますが、実際のファイルにはすでに反映されています** 。  
そのため、Accept/Reject前に一度動かしてみて、うまく動かなければRejectするといった使い方ができます。

## AIとのコーディングの付き合い方

AIと一緒にコーディングするとなると、とても便利な反面、思わぬデグレや余計な変更が加わることもあります。以下のポイントを押さえておくと、AIとの付き合いがスムーズになります。

1. **こまめなコミット**  
	AIにコードをいじらせる前に、必ず `git commit` などで現在の状態を保存しておくと良いです。万が一大きく壊れたり違う方向へ進んでしまっても、すぐに元に戻せます。Agent機能でごそっと変更された場合も安心です。
2. **"おまかせ"しすぎない**  
	AIは便利ですが、 **100%信頼** してポンとコードを適用してしまうと、問題が混入することもあります。生成されたコードはしっかりレビューしたり、小さな単位で `Accept` していくことをオススメします。 **ブロック単位のAccept/Reject** ができるので、使いすぎない程度に適度なチェックが大事です。
3. **指示は具体的に**  
	「テストコードも含めて書いて」とか「React 19に対応して」など、 **ルール** や要望を詳しく書くほど、AIが意図を汲み取ってくれやすいです。`.cursor/rules` フォルダーにルールファイルを作って **コーディング規約やアーキテクチャの方針** をAIに示しておくのもオススメです。
4. **必要に応じて無視設定**  
	`.cursorignore` を使って、AIに読み込んでほしくないファイルやフォルダーを指定しておくと良いです。機密っぽい設定ファイルなどは除外しておく方が無難です。`.gitignore` で無視しているファイルはデフォルトで無視されるので、重複して書かなくて大丈夫です。
5. **AIの提案を学ぶ姿勢**  
	AIが提案してくれる修正やコマンドは、自分が知らなかったテクニックや最適化を含むことが多いです。なんとなくでAcceptするだけではなく、「なぜこう書いたのか？」と疑問を持ちつつ受け入れていくと、自分のスキルアップにもつながります。

## まとめ

ざっくりCursorを紹介してきましたが、AIを使ってソフトウェア開発を効率化できるのが最大の魅力です。  
**Tab** 機能でのコード補完や **Ask** での手軽な問い合わせ、 **Agent** での自動修正・ターミナル操作など、開発の流れをまるっとサポートしてくれます。ちょっとしたコード修正や複数ファイルにまたがるタスクも、スムーズに処理してくれるのでとても助かります。  
プランに関しては、無料のHobbyからビジネス向けのBusinessまで豊富に用意されているので、個人の試用からチーム全体の導入まで幅広く使えます。

というわけで、Cursorを活用して開発ライフをレベルアップさせてみてはいかがでしょうか。

124

43