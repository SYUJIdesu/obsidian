---
title: "社内デザインシステムをMCPサーバー化したらUI実装が爆速になった"
source: "https://zenn.dev/ubie_dev/articles/f927aaff02d618"
author:
  - "[[Zenn]]"
published: 2025-04-05
created: 2025-05-15
description:
tags:
  - "web"
  - "MCP"
---
[Ubie テックブログ](https://zenn.dev/p/ubie_dev) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

1348

457

## はじめに

こんにちは、普段 Ubie で症状検索エンジンユビー([https://ubie.app/](https://ubie.app/))の開発をしている江崎です。

最近、Cursor エディタや GitHub Copilot などのコーディングアシスタントツールが進化し続けていますが、社内固有のデザインシステムとの連携はまだまだ課題が残っていました。そこで社内エンジニアである [sosuke](https://x.com/__sosukesuzuki) とともに、Ubie Vitals というデザインシステムを MCP サーバー化することで、UI 開発の速度と精度が劇的に向上した体験を共有します。

## 目次

- デザインシステムと開発の現状課題
- MCP サーバーの登場
- Ubie UI MCP の構築
- デモ
- テキストだけで UI 実装が可能に
- デザイナーの壁打ち相手としての可能性
- 今後の展望

## デザインシステムと開発の現状課題

Ubie では「Ubie Vitals」というデザインシステムに則って UI 開発を行っています。普段の開発では Cursor エディタを使っていますが、AI にデザインシステムのコンテキストを理解させることが大きな課題でした。

Ubie Vitals: [https://vitals.ubie.life/](https://vitals.ubie.life/)

具体的には、

- Cursor Rules にデザインシステムの Web サイトを読み込ませる
- node\_modules 内のディレクトリを参照させる

などの方法を試しましたが、Props、Token、Icon 情報などを十分に理解させることができず、理想的な開発体験には遠く及びませんでした。

## MCP サーバーの登場

そんな中、MCP サーバー（Model Context Protocol）が注目を集めるようになりました。MCP サーバーは、特定の知識領域に特化した情報を AI に提供する仕組みです。これを活用すれば、デザインシステム固有の知識を AI に効率的に提供できるのではないかと考えました。

## Ubie UI MCP の構築

勉強がてら「Ubie UI MCP」を構築してみることにしました。この MCP サーバーは、

- Ubie UI のコンポーネント情報
- デザイントークン（文字サイズ、色など）
- アイコン情報

などを AI に提供します。結果として、MCP サーバー化したことで精度が飛躍的に向上し、UI 実装が驚くほど効率化されました。

## デモ

実際のデモを見てみましょう。Cursor にプロンプトを投げます。

![](https://res.cloudinary.com/zenn/image/fetch/s--FRNiqkXn--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/ad2229eebf0b14459a912f91.png%3Fsha%3Dfc8562a7bf044c602d0488ac1660d99dbab82c03)

すると、Cursor が実装計画を立ててくれます。

![](https://res.cloudinary.com/zenn/image/fetch/s--JamVZemr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/e45ac98802f1962055b729c0.png%3Fsha%3Dbf49f8f1b22d5aa6583030d0055aa5398f7f6436)

1. まず Figma MCP から Figma の情報を取得
2. 次に Ubie UI MCP が呼び出され、コンポーネント情報やトークン、アイコン情報を取得
3. Figma の情報をもとに、Ubie UI を使って実装を提案

これを実際に実装してもらうと、最終的なアウトプットはこちらです。

| Figma のデザイン | Cursor が実際に実装したコンポーネント |
| --- | --- |
| ![](https://res.cloudinary.com/zenn/image/fetch/s--KzhiFncq--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/c37bc5b3b1a147de1901eb68.png%3Fsha%3Dfac548122c68b5a1772901de59d1301c8264c64c) | ![](https://res.cloudinary.com/zenn/image/fetch/s--1Wd6grfZ--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/2e0cdcc81bbc9a137b7fbd48.png%3Fsha%3Da62df9bccb137044a55b3f34cebe1f5f1f3f9c97) |

一回のやりとりだけで、この精度で UI 実装が完成します。その時間は約 1 分で、従来の方法と比較すると開発時間はかなり短縮されます。  
ちなみに上のステッパーコンポーネントは、Ubie UI には存在しないコンポーネントです。そのようなコンポーネントは再現精度が低いこともわかります。

## テキストだけで UI 実装が可能に

テキストベースの指示だけで Ubie UI を使った実装が可能になります。例えば  
「ユーザー情報入力フォームを Ubie UI で作成して。名前、メールアドレス、年齢の入力欄と送信ボタンが必要」  
といった指示だけで、デザインシステムに準拠した UI を短時間で生成できるようになりました。

![](https://res.cloudinary.com/zenn/image/fetch/s--oxIsFd6X--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/59afb35f35c02237c697aef8.png%3Fsha%3D77c4059c3c63bce9193ccaae4eaa274f43583c22)

プロンプトを投げてみると、このように UI 実装が完成します。

![](https://res.cloudinary.com/zenn/image/fetch/s--HtzQIHkr--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/d9b984c7ba6311b0fcbbbdff.png%3Fsha%3De1135ad48cf7c5fb085ed5a6e2cd0d7b7208b9e7)

![簡易なバリデーションもしてくれている](https://res.cloudinary.com/zenn/image/fetch/s--ReLI7uiK--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/15bc254d969d5ffa27553126.png%3Fsha%3D33814104473919292236380b558b945c52c9b319)  
*簡易なバリデーションもしてくれている*

## デザイナーの壁打ち相手としての可能性

これはエンジニアだけでなく、デザイナーにとっても大きな可能性を秘めています。デザイナーが UI デザインを考える際の壁打ち相手として Ubie UI MCP を活用することで、デザイン案の素早い具現化や、デザインシステムの制約内での探索が可能になります。将来的には、デザイナーが Figma で一から描かなくても、Ubie UI MCP との対話だけで UI デザインが完成する未来も見えてきます。

## 今後の展望

この革新的な開発体験をさらに発展させるためには、Ubie Vitals デザインシステム自体の充実が不可欠です。今後は、

- コンポーネントの網羅性向上
- ユースケースの充実
- MCP の応答精度の向上

に取り組んでいく予定です。  
MCP サーバー化によって、デザインシステムの資産価値が飛躍的に高まった感覚があります。今後もデザインシステムと AI の融合による開発効率化を推進していきます。

## 最後に

今後も X で AI 情報発信していくので、ぜひフォローしてください！

[GitHubで編集を提案](https://github.com/kotaesaki/zenn_connect/blob/main/articles/f927aaff02d618.md)

1348

457

この記事に贈られたバッジ

![Thank you](https://static.zenn.studio/images/badges/paid/badge-frame-5.svg)