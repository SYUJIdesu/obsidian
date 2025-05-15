---
title: "Obsidianのフォルダ構成をIdeaverse準拠にしてみた｜illy / 入谷 聡"
source: "https://note.com/irritantis/n/nb72f11bbeb03"
author:
  - "[[note（ノート）]]"
published: 2023-10-08
created: 2025-05-15
description: "このnoteは、メモツールObsidianで、Zettelkasten（ツェッテルカステン）などのメモ術を実践している方に向けた、マニアックな解説記事です。   2020年初ごろから使っているObsidianのフォルダ構成を、\"Ideaverse\" 準拠で再構築しました。 現在のVault。テーマはSolarized Ideaverseとは、\"Linking Your Thinking (LYT)\" ブランドでObsidianのトレーニングなどを提供しているNick Milo氏が、2023年9月から無料配布している、ObsidianのVaultセットです。無料で計6回のメール講座が付属"
tags:
  - "web"
  - "Ideaverse"
---
![見出し画像](https://assets.st-note.com/production/uploads/images/118359790/rectangle_large_type_2_2d47e4cd22ac956c9723b09099f0d63e.png?width=1200)

## Obsidianのフォルダ構成をIdeaverse準拠にしてみた

[illy / 入谷 聡](https://note.com/irritantis)

> このnoteは、メモツールObsidianで、Zettelkasten（ツェッテルカステン）などのメモ術を実践している方に向けた、マニアックな解説記事です。

2020年初ごろから使っているObsidianのフォルダ構成を、"Ideaverse" 準拠で再構築しました。

![画像](https://assets.st-note.com/img/1696762759739-dahZiZLdpG.png?width=1200)

現在のVault。テーマはSolarized

**Ideaverse** とは、"Linking Your Thinking (LYT)" ブランドでObsidianのトレーニングなどを提供しているNick Milo氏が、2023年9月から無料配布している、ObsidianのVaultセットです。無料で計6回のメール講座が付属し、段階的に学ぶことで、LYTのエッセンスが学べるようになっています。

[**Get the Ideaverse for Obsidian** *Explore ways to manage your linked notes. Experiment. Learn b* *start.linkingyourthinking.com*](https://start.linkingyourthinking.com/ideaverse-for-obsidian)

Ideaverseをダウンロードすると、多数のファイルが同梱されたVaultを利用できます。これをカスタマイズしながら育てていくこともできますし、運用中のVaultの中身をそのまま、主要なフォルダ構成を足してファイルを移動させることで、Ideaverseの構造に準拠したVaultに作り替えることができます。

![画像](https://assets.st-note.com/img/1696763110694-V3zwrrBLbn.png?width=1200)

普段使いのVaultと並行してIdeaverseのVaultを開くことができる

以下、Ideaverseの要点と、手元で試みた再構築手順のポイントを紹介します。

## Atlas

"Atlas" は、整形したアイデアメモ、MOC（マップ、索引記事）、テンプレートや画像ファイルなどを格納するフォルダです。Zettelkastenにおける "Permanent Note"（永久保存版メモ）の置き場として、最も重要なフォルダと言えるでしょう。

……とはいえ、今まで貯めるだけ貯めたメモを見返して思ったのは、よっぽど日々丁寧に「走り書きメモから永久保存版メモに清書する」という手順を踏んでいない限り、Atlas > Notesに入るメモは多くありません。Permanent Noteの定義が「読めば意味がわかる程度に編集されたメモ」であるならば、意識的にメモを作る（note-takingではなく、 **note-making** ）作業を繰り返すのが、Ideaverseの前提になっています。

私は最近、日常的なメモはDaily Noteにどんどん書くことにしており、そこからデイリーで「重要なメモを切り出してAtlasに格納する」ことを重ねていけば、知識を整理することができそうです。逆に、Note-makingをしない限り、メモは埋もれるだけで生きることがない。

### Map

Ideaverseは、2023年7月に公開されたObsidian v1.4.0で実装された "Properties" を大活用し、随所でコミュニティプラグイン "Dataview" を用いてNoteを一覧抽出するしくみを入れています。

Map (MOC = Map of Contents, 索引) は、個々の記事をまとめた一覧ページ。 **1つの記事が無数のMapに紐付く** ことで、知識のリンクが広がっていくことがObsidianの醍醐味です。

手元の簡単な例では、「文章術タイプ別おすすめ本」という記事を、"Writing Mastery（文章術に習熟する）" と "Bookguide" という2つのMOCに関連付けようとしています。

![画像](https://assets.st-note.com/img/1696763744795-Z0I1fOjXn6.png?width=1200)

個別記事。Properties > up では親記事へのリンクを設置し、複数のタグをつける

![画像](https://assets.st-note.com/img/1696763877630-1CNX4aIYGA.png?width=1200)

MOC

MOCから各記事へのリンクは、  
①手動  
②Backlinkを見る  
③Dataviewで一覧生成する  
の3つの方法があります。Backlinkで特定のタグの一覧を差し込む記述はかなり簡単です。

```javascript
\`\`\`dataview
LIST
FROM #writing_mastery
\`\`\`
```

[Dataviewのオフィシャルガイドはこちら](https://blacksmithgu.github.io/obsidian-dataview/reference/expressions/) （かなりいろいろできる）

### Add

新規作成したNoteの初期配置先フォルダは「+」に設定します。Ideaverseの "Add" というページでは、このフォルダに配置されている記事を鮮度順に並べるdataviewが組まれていました。

![画像](https://assets.st-note.com/img/1696769016867-l5qpgi2kGE.png?width=1200)

デフォルトの作成先指定。いわゆるInboxがこのフォルダ

![画像](https://assets.st-note.com/img/1696768983427-romDNVeQGL.png?width=1200)

Ideaverse側のAddページ。Days aliveは作成日からの経過日数

このリストにある「未整理のnote」を処理し、Permanent Noteに書き直してAtlasに保存するか、捨てるか、その他の方針を決めて「+」を常に空に保つこと(Inbox Zero)が日常的なメンテナンスになります。

## Calendar

「時間」を司るCalendarフォルダは、原則としてDaily Note置き場です。私はCommand+Shift+Jのホットキーで当日のDaily Noteを、Command+Shift+H/Kで前後の日付に移動できるように設定し、Calendarプラグインも有効化しています。

![画像](https://assets.st-note.com/img/1696765731866-90NzNAiJxv.png?width=1200)

デイリーノート

## Efforts

Efforts（努力、活動）は、"プロジェクト"とほぼ同義（上位概念？）と説明されています。進行中のトピックを4段階の「熱さ」で整理し、個々の記事はMOCのように振る舞います。

![画像](https://assets.st-note.com/img/1696769145047-iULonTqMJJ.png?width=1200)

Effortsの一覧MOC

- On 最もホット
- Ongoing ややホット
- Simmering とろ火
- Sleeping 休眠

ここでは、 **今意識を向けるべきテーマ（仕事、プライベート、独自研究、読書など）をOn/Ongoingに並べておき、そのリストをHome画面で常に参照できる** ようにし、各Effort記事を起点にどんどん情報を膨らませていくことで、重要事項に関するアイデアがどんどん集まってくる構造にできると理解しました。逆に「終わったもの」はSleepingにどんどん置いていけばアーカイブになります。

階層型ではなくリンク型で管理するスタイルでは、個々のnoteをフォルダから直接探しに行くことは原則ありません。各MOCを起点にリンクを辿っていき、リンク構造の中でセレンディピティを見いだすことが主眼です。であるならば、Effortsをメインの入り口とし、Mapを描いていくことが、重要な日々の活動になると理解しました。

ノート術は「学ぶ」そして「アウトプットする」ための技法であり、整理することを目的としても何ら発展性はありません。そんな中、 **「いま集中するべきEffortは何なのか」** という問いは、ノートづくりの目的を強く意識させることにもつながる重要な問いです。

## 次の一手

一旦は構造を整えたものの、現状は追うべきEffortsも一部しか明確になっておらず、また貯めるべきNotes/Ideasの量も全く足りていない状況。まずはボトムアップで今関心のあるテーマや業務上アンテナを張るべきテーマを言語化し、少しずつNoteを貯めていくのが次のステップです。

[**TAKE NOTES!――メモで、あなただけのアウトプットが自然にできるようになる** *www.amazon.co.jp*](https://www.amazon.co.jp/dp/4296000411?tag=note0e2a-22&linkCode=ogi&th=1&psc=1)

[*1,870 円* (2024年07月09日 12:29時点 詳しくはこちら)](https://www.amazon.co.jp/dp/4296000411?tag=note0e2a-22&linkCode=ogi&th=1&psc=1)

[

Amazon.co.jpで購入する

](https://www.amazon.co.jp/dp/4296000411?tag=note0e2a-22&linkCode=ogi&th=1&psc=1)

[**Linking Your Thinking with Nick Milo** *This is an exciting time in "Personal Knowledge Management".**www.youtube.com*](https://www.youtube.com/@linkingyourthinking)

  

## いいなと思ったら応援しよう！

🍻

- [
	#ノート術
	](https://note.com/hashtag/%E3%83%8E%E3%83%BC%E3%83%88%E8%A1%93)
- [
	#Obsidian
	](https://note.com/hashtag/Obsidian)

Obsidianのフォルダ構成をIdeaverse準拠にしてみた｜illy / 入谷 聡