---
title: 【個人開発】チーム開発して最新技術に乗り換えた話【Next.js Rails】
source: https://qiita.com/naruqiita/items/99668639cb3d991fc6f5
author:
  - "[[naruqiita]]"
published: 2024-04-21
created: 2025-05-22
description: 目次結論ターゲット筆者経験個人開発 vs チーム開発どう変わったか技術アップデートの背景アップデートのpros & cons技術に興味がある人向け結論結論力を付けて結果出すなら…
tags:
  - web
  - 情報
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

## 目次

1. 結論
2. ターゲット
3. 筆者経験
4. 個人開発 vs チーム開発
5. どう変わったか
6. 技術アップデートの背景
7. アップデートのpros & cons
8. 技術に興味がある人向け
9. 結論

## 結論

力を付けて結果出すなら **チーム開発** ！

## ターゲット

- 個人開発に興味があるエンジニア
- 新しい技術を学びたい人
[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/014ea758-a4d0-1a6f-cca9-62f0e434ac61.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F014ea758-a4d0-1a6f-cca9-62f0e434ac61.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=93903c806b50ded110d95d3dd0169b0a)

## 筆者経験

個人開発特化Webアプリ ideee（アイディー）の制作者

```text
アイデアのブラッシュアップができる
面白いアイデアに出会える
```

- 個人開発とチーム開発の両方の経験あり
- 技術乗り換えを行なった経験

### 🎊祝🎊新基盤デプロイ

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/4ce7d94f-4e70-ad81-767f-e80d29f6a763.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F4ce7d94f-4e70-ad81-767f-e80d29f6a763.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e4b297f189112263c15088694fc72752)

## どう変わったか(Before & After)

#### アプリ構成（Before）

[![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fcamo.qiitausercontent.com%2Fa3c9b49b868150bffac3dab5be0a21a814cd4d29%2F68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3439383730312f66306331643435372d383862662d626662622d363261332d3633663538363664353839372e706e67?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9552aaea5c94cad02bfee3b096708106)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fcamo.qiitausercontent.com%2Fa3c9b49b868150bffac3dab5be0a21a814cd4d29%2F68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3439383730312f66306331643435372d383862662d626662622d363261332d3633663538363664353839372e706e67?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9552aaea5c94cad02bfee3b096708106)

#### アプリ構成（After）

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/ebea61c7-ed2e-c781-a376-a8adf9f9ee2c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2Febea61c7-ed2e-c781-a376-a8adf9f9ee2c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9e58a3339e5746f28d52e2e14fa26dd4)

### 以前のスタック

- Rails 6系
- jQuery
- Materialize

### 更新後スタック

- Rails 7.1
- Next.js(TypeScript)
- GraphQL(Apollo + Codegen)
- Mantine

## 技術アップデートの背景

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/6369c2d2-4b31-95da-c64e-5250d18c83b4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F6369c2d2-4b31-95da-c64e-5250d18c83b4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=8a92ae243727816078432cb0c698af1a) 開発スピードの向上

```text
Cursor(AIエディター)やCopilot、CodeRabbit（AIレビュワー）を利用
AI開発時代に合わせて、コメントやTypeScript（型付）を増強
→ AIを使いこなすためのスタックに変わってきた
```

パフォーマンスの向上

```text
- Next.jsのプリフェッチ、SPAで圧倒的な速度向上
- GraphQLのApolloキャッシュにより毎回のリクエストを大幅カット
```

技術力の向上

```text
Rails ✖️ React or Next.jsのスタックで求人数は増加
（海外でもRailsの人気が再熱してるらしい。。）
GraphQLも人気があり求められるエンジニアに
```

## アップデートのpros & cons

### メリット

技術が変わるだけで人が集まる

```text
Railsのみの募集：2名〜5名の応募
新基盤での募集：3名〜16名の応募
```

圧倒的に楽しい

```text
新技術導入でコミットパフォーマンスが上がる
→　より少ない時間で大きな結果が出る
→　開発体験が良くなる
→　コミットパフォーマンスが上がる
```

### デメリット

移行の難易度

```text
新技術導入のハードル
GraphQLとdevise・画像アップロードなどの想定外ケースがあった
```

学習コスト & 時間がかかる

```text
新基盤開発期間は既存の開発がストップ
→アプリの盛り上がりに伸び悩み
```

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/7935a95a-fc89-9c35-4f11-250170b0e3b8.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F7935a95a-fc89-9c35-4f11-250170b0e3b8.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=46120fe1c5bad9e182a64868f875fb7e)

## 個人開発 vs チーム開発

### 個人開発

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/6bb5c9d5-8f3b-46c2-e2e5-f02954244774.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F6bb5c9d5-8f3b-46c2-e2e5-f02954244774.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=fba7a77f25e88a53ebea9a5eacdbafae)
- 自由度高い 🚀
- 迅速な決定 💨

### チーム開発

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/a623aaf7-2fc3-1d4b-b437-741f64802bf1.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2Fa623aaf7-2fc3-1d4b-b437-741f64802bf1.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9f6ef141ca5e2b7db0e60f365bf86d39) 品質とスキルの向上

```text
- 強みを活かした開発
  - フロントやバック専門の補完的開発
- PR確認でメンバーの知見がチームの知見になる！
- 転職に成功　
　- ５人のメンバーがチーム開発を活かして転職に成功🚀🚀🚀
```

多様性とモチベーションの維持

```text
- 個人開発だとモチベが下がりがちになる
  - 毎週のチームMTGでもワイワイ
- デザイナーやPMの視点を追加

- ペルソナがエンジニア
→ メンバーの声 ≒ ユーザーの声 を反映
```

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/0cb856ab-3dd5-ef8c-b668-1eee869fb1f7.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F0cb856ab-3dd5-ef8c-b668-1eee869fb1f7.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3fdf488cbd7a7c5b7c51d232601e9751)

## つまずきポイント

（技術興味が高い人向け）

やりたいベースで走る危なさ

```text
GraphQLのCodegenが開発体験良すぎてなんとしても使いたかったw
→　結果、GraphQLに引っ張られて開発の時間が大幅に増加💀💀💀

（GraphQL使いたかった理由）
Rails側でのコード定義からReactにHooksと型、設計書ドキュメントを自動生成！
→ Open APIを作成・メンテしたり、Hooksを自分で作成する手間がゼロ
```

deviseで時間溶かした話

## 結論

- 個人は早さ、チームは成果に強み
- チーム開発の方が仕事に対しての再現性が高く、転職の成功に繋がりやすい！

---

## 自己紹介

個人開発について発信しています！  
Twitterもフォローください👏  
[@1026NT](https://twitter.com/1026NT)  
なる

[![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/498701/3db40e7d-3213-be1f-8650-c6ad5dff69c9.jpeg)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F498701%2F3db40e7d-3213-be1f-8650-c6ad5dff69c9.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=2793b33bef75d8259cfdc7e8369e4f6f)

[0](https://qiita.com/naruqiita/items/#comments)

コメント一覧へ移動

[17](https://qiita.com/naruqiita/items/99668639cb3d991fc6f5/likers)

いいねしたユーザー一覧へ移動

12

ストックを更新しました