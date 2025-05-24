---
title: SSR, CSR, SSG, PPR の整理
source: https://zenn.dev/beingish/articles/a5545ebf4f1fa7#csr
author:
  - "[[Zenn]]"
published: 2024-07-21
created: 2025-05-15
description: 
tags:
  - web
  - フロントエンド
---
[株式会社 being-ish](https://zenn.dev/p/beingish) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

140[tech](https://zenn.dev/tech-or-idea)

[PPR](https://nextjs.org/learn/dashboard-app/partial-prerendering) の登場でだいたい登場人物が出揃ったかな、というタイミングのため、一度まとめる。

![overview](https://res.cloudinary.com/zenn/image/fetch/s--Uu7u58iT--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/f4eeaadd57d428d01fe06f16.png%3Fsha%3D2ef5fc79689356243188f732c401c34eff8aaccf)

左が古いもの、右が新しいもので並べている。

## これらの目的

いたずらに登場人物を増やしているわけではなく、解決したい課題に対する解法としてこれらがある。というわけで、そもそもこれらが出てきた目的に立ち戻って考えてみる。

と言っても特に難しいことはなく、すべてユーザーからのリクエストを起点としてできるだけ速くユーザーが画面を操作できるようにするためのものだ。特に最近出てきたものほどその傾向が強い。

速度にこだわる理由は、コンバージョンや売上の減少を回避するためが大きいだろうか。ユーザーが短い時間で目的を達成できるのであればそれに越したことはないし、ともすればロイヤルティも向上する。

待ち時間は少ないほうが良いのか？

ここではどういった状況であっても待ち時間は少ないほうが喜ばしいという推測に立っている。が、すべての Web サービスでそうであるかどうかの保証はないため、あなたのサービスにもこれが当てはまるかをきちんと確かめると良い。

たとえばゲームなどではインタラクティブの際に都度待たされるよりも即座に反応が欲しい。そのため先に長めのロード時間ですべてのリソースをメモリーに載せるなどで対応しているが、この挙動はユーザーの一定の理解の上での実装だと考えられる。

## どれを選べば良いか？

インタラクティブに動く必要がなく、プライベートなデータを表示しないのであれば SSG 、そのうち、内容を更新する必要があるのであれば ISR 。

それ以外では様々な軸を考慮する必要があり、アプリの性質によって最適なものは変わってくる。次に述べることは参考程度に。

### ユーザーの待ち時間をなるべく少なくしたい場合

次の選択肢を上から検討する。 GA 前でも PPR が使えそうか見てみる、それがダメなら SSG + client fetch を調査…、という順番で検討していく。

1. PPR
2. SSG + client fetch
3. streaming SSR
4. SPA

SSG は 1 回のコンテンツ生成時間で複数ユーザーに価値を届けられるため最優先で検討すると良い。また、 SSR はサーバー内でキャッシュを効かせることで高速化できる余地が生まれるため、 SPA よりも先に検討したほうが良い。

### 運用コストを低減したい場合

次の順番となるはずだが、実際に運用したわけではないので話し半分で。

1. SPA
2. SSG + client fetch
3. PPR
4. streaming SSR

ビルド済みのアセット配布を CDN で、動的なデータ授受については API サーバーで済ませられる SPA が必要なリソースが少なくなる。 SSG は生成処理をどこでやるかによるが、 CI/CD 段階でやれば SPA とさほどコストは変わらない。 SSR はリクエストごとにリソースを消費するので、ランニングコストが膨らむ想定だ。

## 言葉

それぞれを略さずに表記すると次になる。

- SSR: Server Side **Rendering**
- CSR: Client Side **Rendering**
- SSG: Static Site **Generation**
- ISR: Incremental Static Re **generation**
- PPR: Partial Pre **rendering**

[rendering](https://www.oxfordlearnersdictionaries.com/definition/english/render?q=render#render_sng_5) とはコードからブラウザーが解釈可能なリソースへの生成処理を指す。リアルタイムなニュアンスを含んでいる。

[generation](https://www.oxfordlearnersdictionaries.com/definition/english/generate?q=generate#generate_sng_2) とはページとして意味をなす、すべてのリソース生成を指している。

やっていることそのものには両者に違いはなく、リアルタイムなニュアンスを含むかどうかの違い程度しかない。◯◯ side という表現はリアルタイム変換をどこで実施するかの補足だ。

## それぞれの説明

基本的に Web における説明を書く。

また、著しく性能が低いデバイスや速度が低いネットワークではどの手法を用いてもユーザーの待ち時間は長くなるため、特段記載しない。

### CSR

描画すべき DOM ツリーを、コードを実行することによりブラウザー上で生成する。その際に必要なデータはどこかから取ってくる。どこからというのは明確に決まっているわけではない。

![CSR diagram](https://res.cloudinary.com/zenn/image/fetch/s--EqP07vAW--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/ec250a11f2cc7657395564d7.png%3Fsha%3Ddecfe85003067cd6d7fc9ad89ed0985ac95d1e19)

DOM ツリーの一部もしくは全部を生成する。 SSR や SSG によって生成された HTML がブラウザー上で解釈されたあとに CSR が走る。

メリットは次。

- ユーザーに一番近くレイテンシーが著しく低いため、自己帰属感を高く維持できる
- ユーザーごとに内容を生成するため、プライベートな内容を含んでも問題となりづらい

デメリットは次。

- 複雑なページは描画までに時間がかかる

Web だけの概念ではなく、モバイルアプリなどは原理的に CSR となる。

### SSR

描画すべき DOM ツリーを、コードを実行することによりサーバー上で生成する。その際に必要なデータはどこかから取ってくる。こちらもどこからというのは明確に決まっていない。

![SSR diagram](https://res.cloudinary.com/zenn/image/fetch/s--_S2hz6NV--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/bd6f0638308f2b83107e1699.png%3Fsha%3De3f8a56b4637006a2ff4c95b08ed68a37d76246d)

DOM ツリーの一部もしくは全部を生成する。 SSG によって生成された HTML がブラウザー上で解釈されたあとに SSR が走ることもある。

メリットは次。

- ユーザーごとに内容を生成するため、プライベートな内容を含んでも問題となりづらい
- 生成した内容をサーバー内でキャッシュできるため、 2 回目以降のレスポンス生成を高速化できる可能性がある
- 必要なデータ取得にかかる時間を制御下におけるため、レスポンス生成を高速化できる可能性がある

デメリットは次。

- 複雑なページは描画までに時間がかかる
- ネットワークを介するため、 CSR に比べるとユーザーからの入力に対するレスポンスは時間がかかる

### SSG

HTML や CSS 、画像などページを構成するリソースをあらかじめ生成しておく。

![SSG diagram](https://res.cloudinary.com/zenn/image/fetch/s--lo7pmhrL--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/9fb3892a1a5490738d304a77.png%3Fsha%3Df7a733b605626167634a69f8ec2c50aeaee06774)

メリットは次。

- リクエスト時に何も処理が走らないため、ユーザーの待ち時間はネットワークのみに依存する
- CDN による広域キャッシュが効くため、エッジまでのレイテンシーと転送容量によってユーザーの待ち時間が決まる

デメリットは次。

- ユーザー全体を対象とするページ生成のため、個々のユーザーに向けたプライベートな内容を含むことができない。
- ユーザーからの入力を元にした動的な内容を生成できない

こういった特性から read-heavy なサービスに向いている。具体例で言えば Zenn などブログサービスで実力を発揮しやすい。

## 知っている限りの歴史

### original MVC

多分 CSR に相当する。 1970 年代後半にはあった。

ネットワークが身近でない時代に考案されたアーキテクチャーのため、手元のストレージからデータを取得し、それを元に手元で画面を生成して表示するための手法だ。

![original MVC diagram](https://res.cloudinary.com/zenn/image/fetch/s--X0hKLnn9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/91669c0f62b6c60937089243.png%3Fsha%3D659156c9850954da6081a5d425e327ca7d701cb6)

### Web MVC

SSR に分類される。 1998 年頃から。

original MVC が理解しやすく応用しやすかったため、 Web に適用してみた結果だ。

![web MVC diagram](https://res.cloudinary.com/zenn/image/fetch/s--hf8Ja3Sk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/5cc18d633809c074bb893a09.png%3Fsha%3D70e5c44e6ea32dff78f67334fbed6f0d8e0291f0)

Ruby on Rails などで花開き、世界的に受容されていった。

ユーザーからの入力を扱うにあたって Web という制約があったため、 [POST back](https://e-words.jp/w/%E3%83%9D%E3%82%B9%E3%83%88%E3%83%90%E3%83%83%E3%82%AF.html) などでゴリ押していた時期もあった。 UX という言葉がなかった時代だ。

### SPA

SPA は [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application) の略。

現在 CSR というとこのイメージ。 2004 年頃から Gmail や Google Map で Ajax の有用性が示され Web でリッチな体験を提供するには SPA 一択という状況になっていった。

![SPA diagram](https://res.cloudinary.com/zenn/image/fetch/s--9vBPiPpu--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/6fb093bc5534240f1d29c65e.png%3Fsha%3De7adcaab167819ee6fefb1764e6fed6da3b0f52a)

### SSG

もともと GUI でサイトをオーサリングするツールなどは昔からあったが、コードやデータなどから静的な web サイトを生成するツールは [Jekyll](https://jekyllrb.com/) がはしりだと思われる。 2008 年頃から存在している。

生成した HTML などを CDN で配信することで高速な表示が可能となる。

![SSG diagram](https://res.cloudinary.com/zenn/image/fetch/s--lo7pmhrL--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/9fb3892a1a5490738d304a77.png%3Fsha%3Df7a733b605626167634a69f8ec2c50aeaee06774)

最近は宣言的に UI を書けるライブラリーを用いた記述もできるものが出てきており、スキルセットの共通化のみならず SSR と SSG の切り替えが可能なものもある。

### SSR with VDOM library

レンダリングをサーバー上でやるようにしてみたらメリットがあるということで 2016 年頃から観測されている。

プライベートな情報にキャッシュをきかせやすいという点が一番のメリットだろうか。

同一ネットワーク内にデータベースを置くことが可能なため、データ取得のレイテンシーを低減しやすいというのもある。

![SSR with VDOM library diagram](https://res.cloudinary.com/zenn/image/fetch/s--ZClOpaVk--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/ee552caf53ff26ca59dc2a12.png%3Fsha%3D372f5eca6634b76775eb79588d0437b116ece1be)

登場時は検索結果で上位に表示されるための手法という側面もあったが、現在ではクローラーの進化によりその意味合いも薄れてきている。

CSR できるコードを用いて SSR しているため、容易にブラウザー上で [ハイドレーション](https://en.wikipedia.org/wiki/Hydration_\(web_development\)) できる。つまりインタラクティブなページや追加のデータ取得などが可能だ。また、 SSR / CSR で覚えるべきことがあまり変わらないこともメリットだ。

### ISR

SSG をベースに、より内容の更新をしやすくしたもの。 2020 年頃から Next.js がサポートしている。

SSG では中身を書き換えるにはリソースを再生成しなければならない。サイトが大きくなってくると全体の再生成は時間がかかるので、ページ単位で再生成をかけるようにする。さらに、リクエストされないページの再生成をしても無駄なのでリクエスト時に生成処理を走らせるようにする、という理屈の仕組みだ。

![ISR diagram](https://res.cloudinary.com/zenn/image/fetch/s--IB6rzaak--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/1c9cfb148ac3f0e917c0744a.png%3Fsha%3Dcb604cd35f2b53d1919c90256b05c20ac881a081)

### SSG + client fetch

SSG と CSR を組み合わせ、お互いのメリットを享受できるようにしたもの。

2020 年頃には Atro Islands という概念があった。

SSG 生成した HTML をベースとし、ブラウザー上でハイドレーションし追加データの取得やインタラクティブな挙動を実現する。

![SSG + client fetch](https://res.cloudinary.com/zenn/image/fetch/s--RQ3s666R--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/eb1c65a9aa4669ecae98f1d5.png%3Fsha%3D6a03dc808bcd51aa2e87e08fd518d12c7c2c2e13)

### streaming SSR

SSR の進化系。

2022 年頃から Next.js が提供している。その他のフレームワークでも出てきている可能性があるが把握できていない。

SSR は必要なデータが揃うまでレスポンスを生成できない。つまりユーザーの待ち時間がそれだけ増える。

これを防ぐため streaming SSR では先に返せるところを返し、時間がかかる部分を後から差し替えるという発想で待ち時間を最小化するものだ。

![streaming SSR diagram](https://res.cloudinary.com/zenn/image/fetch/s--zWZH6E4e--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/07bf4731e5864883c7c4d4a1.png%3Fsha%3D13bb7142c228f0ea47dbeaf1b5b6770961de3080)

### PPR

SSG + SSR という理解で良いはず。

2024 年内には GA するか？

先に作れるところだけ SSG しておき、プライベートな部分などは SSR で返す。

![PPR diagram](https://res.cloudinary.com/zenn/image/fetch/s--TlSks36e--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_1200/https://storage.googleapis.com/zenn-user-upload/deployed-images/acad0f76b735d74c42fc377e.png%3Fsha%3D58e5af5a87c48f3b53707310ff3ab5e177073615)

[GitHubで編集を提案](https://github.com/januswel/zenn-contents/blob/main/articles/a5545ebf4f1fa7.md)

140