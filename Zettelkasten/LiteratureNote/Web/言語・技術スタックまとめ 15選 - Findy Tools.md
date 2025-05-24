---
title: 言語・技術スタックまとめ 15選 - Findy Tools
source: https://findy-tools.io/articles/2024-tech-stack/1
author: 
published: 2024-01-23
created: 2025-05-15
description: 急成長中のスタートアップからメガベンチャーまで15社の技術スタックを伺いました。技術選定の意思決定の裏側にある背景や意図はどのようなものがあるのか。本記事では、各社が用いている言語やフレームワーク、モニタリングやCI/CDなどで活用している技術やツールなどを取り上げ、開発チームの方々がなぜその技術を選定したのかの意図や背景を紹介させていただきます。<small>※掲載している技術スタックは各社からご提供頂いたものを掲載しております</small>
tags:
  - web
  - 情報
---
[![Findy Tools](https://findy-tools.io/img/findy_tools.svg)](https://findy-tools.io/)

[ログイン](https://findy-tools.io/sign_in?r=%2Farticles%2F2024-tech-stack%2F1) [新規登録](https://findy-tools.io/sign_up?r=%2Farticles%2F2024-tech-stack%2F1)

目次

![言語・技術スタックまとめ 15選](https://findy-tools.io/public_images/article/202401_tech_stack/kv.techstackpng.png)

公開日 更新日

## 言語・技術スタックまとめ 15選

企業の規模や業種によって採用される技術スタックは様々異なります。それは事業やプロダクトの特徴、過去に採用してきた技術などの要因に大きく影響されています。  
  
この記事では、スタートアップからメガベンチャーの技術スタックの事例をまとめ、各社がそれぞれどのような意図や背景で現在の技術構成を作っているか、その設計思想を紐解いていきます。  
  
他社の事例から学ぶことで、技術選定における考え方のヒントを見つけられると思いますので、ぜひ参考にしてください。  

※掲載している技術スタックは各社からご提供頂いたものを掲載しております  
  

## 株式会社10X

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド: Flutter / Nuxt.js</td></tr><tr><td>バックエンド：Dart / gRPC</td></tr><tr><td>モバイル：Flutter</td></tr><tr><th>インフラ</th><td>Google Cloud</td></tr><tr><th>コンテナ管理</th><td>Kubernetes</td></tr><tr><th>DB</th><td>DWH： BigQuery<br>DB：Firestore</td></tr><tr><th>監視・モニタリング</th><td>Datadog</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions, CircleCI</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

10Xでは「10xを創る」をミッションとし、小売向けECプラットフォーム「Stailer」の提供を通じて、スーパーやドラッグストア等のオンライン事業立ち上げ・運営支援を行っています。Stailerでは業務構築におけるコンサルティングから、必要な商品マスタやお客様アプリ・スタッフ向けのオペレーションシステム等の提供、配達システムの提供、販売促進の支援など、一気通貫での支援を行っています。  
  

**技術選定で意識していること**

常に事業のあるべき姿から逆算を行い、意思決定を行うことを心がけています。そのため特定の技術にこだわりはなく、事業と組織にとって最適な技術を選定するようにしています。 結果として、事業の状況により大胆な意思決定を行うこともありますし、一方で流行ってるという理由だけで採用するといったこともありません。  
  
10X Product Blogは [こちら](https://product.10x.co.jp/)

  
  

## 株式会社アンドパッド

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：JavaScript / TypeScript / Vue.js / Nuxt.js / React / Next.js</td></tr><tr><td>バックエンド：Ruby /Ruby on Rails / Go</td></tr><tr><td>モバイル：Swift / SwiftUI / Kotlin / Jetpack Compose / Flutter</td></tr><tr><th>インフラ</th><td>AWS / GCP / nginx</td></tr><tr><th>DB</th><td>MySQL / Redis / Firebase / DynamoDB / Neptune</td></tr><tr><th>監視・モニタリング</th><td>Datadog / Sentry / Amazon CloudWatch Logs / BugSnag</td></tr><tr><th>環境構築</th><td>Kubernetes / Docker / Terraform</td></tr><tr><th>CI/CD・テスト</th><td>CircleCI / GitHub Actions / AWS CodeBuild</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

ANDPAD は建築・建設 DX を推進する、現場から経営までをつなぐバーティカル SaaS です。マルチプロダクト戦略のもと、施工管理、チャット、図面、受発注など様々な製品を開発・提供し、 18.7 万社 47.5 万人の毎日の業務を支えています。  
  

**技術選定で意識していること**

アンドパッドには、多数のプロダクトが存在します。各プロダクトは固有の要件とニーズを持っています。そして、それぞれ特色のあるプロダクトを牽引してくれるエンジニアがいます。 立ち上げ段階でエンジニア 1 人で始めることもあれば、グロースフェーズで複数人が協力することもありますが、チームメンバーのスキルセットと適合する技術選定を行い、開発プロセスがスムーズに進行することがまず必要です。加えて、新しい技術を採用するなど、挑戦的な選定を織り交ぜることもまた重要なことです。

アンドパッドでは、意志と責任を持って事業を前に進めてくれるエンジニアの考えが、技術選定に反映されています。統一的な技術スタックで揃えることの利点を理解しつつも、責任を持って推進してくれる多くのエンジニアの、自由な発想で意思決定できる環境であることを重視しています。 （株式会社アンドパッド VpoT 秒速）  
  
株式会社アンドパッドのTechblogは [こちら](https://tech.andpad.co.jp/)

  
  

## キャディ株式会社

<table><tbody><tr><th rowspan="3">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：TypeScript / React / Next.js / WebGL / WebAssembly</td></tr><tr><td>バックエンド：<br>- 言語：Rust / TypeScript / Python<br>- フレームワーク：Rust (axum) / Node.js (Express / Fastify /NestJS) /Python (PyTorch)</td></tr><tr><th>インフラ</th><td>Google Cloud / Google Kubernetes Engine / Anthos Service Mesh</td></tr><tr><th>DB/DWH</th><td>CloudSQL(PostgreSQL) / AlloyDB /Firestore / BigQuery</td></tr><tr><th>監視・モニタリング</th><td>Datadog / Sentry</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions</td></tr><tr><th>その他</th><td>- 検索エンジン:Elastic Cloud<br>- API： GraphQL, REST, gRPC<br>- 認証: Auth0<br>- 開発ツール: GitHub, GitHub Copilot, Figma, Storybook, Miro</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

独自の画像解析アルゴリズムを搭載した図面データ活用クラウド「CADDi DRAWER」を提供しています。2022年6月に正式ローンチされた、まだまだ若いプロダクトですが、数十人規模の加工会社さんから数万人規模のエンタープライズの会社まで、製造業における様々な業界・規模の顧客にご活用いただいており、グローバル展開にも着手しています。  
  

**技術選定で意識していること**

- ドメイン駆動設計に基づく設計手法を取り入れており、型システムを活かした開発を行うため、静的型言語を中心とした技術選定を行っています
- およそ10チームで「CADDi DRAWER」という1つのプロダクトを開発していく上で、各チームが疎結合に動けるようにするため、マイクロサービスアーキテクチャを採用しています
- チームの疎結合化を目指す一方で、車輪の再発明を防ぎ、全体最適を実現するため、チーム横断での標準化にも取り組んでいます
- 数万人規模のエンタープライズ企業にもご利用いただいており、グローバル展開にも着手していることから、セキュリティやスケーラビリティをはじめとする非機能要件の重要性も高まっています。今後の事業展開で求められる高い非機能要件も見据え、技術スタックの見直しを随時行っていく予定です
  
  
キャディ株式会社のテックブログは [こちら](https://caddi.tech/)  
  

## Classi株式会社

<table><tbody><tr><th rowspan="3">言語<br>フレームワーク</th></tr><tr><td>プログラミング言語: Ruby / PHP / Go /TypeScript</td></tr><tr><td>フレームワーク: Ruby on Rails / Angular / FuelPHP / Laravel</td></tr><tr><th>インフラ</th><td>AWS / Redis / Aurora / OpenSearch</td></tr><tr><th>分析</th><td>Google BigQuery, Tableau, Redash, Google Analytics</td></tr><tr><th>監視・モニタリング</th><td>Datadog / Sentry</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>CircleCI / GitHub Actions / Autify</td></tr><tr><th rowspan="4">その他</th></tr><tr><td>開発者ツール：GitHub / Slack / Confluence / esa / Asana, JIRA</td></tr><tr><td>モバイル：Swift / RxSwift / Kotlin /Objective-C / Java / Firebase / fastlane / RxJava</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

Classiは学校向けの教育プラットフォームです。学校の先生・生徒・保護者が利用しています。主な機能としてコミュニケーション機能や学習機能などがあり、現在は個別最適な学びの実現に向けた学習機能に注力しています。これまで2400校を超える学校で利用され、累計で210万人以上の生徒の学びを支援してきました。  
  

**技術選定で意識していること**

結論から言えば、「この技術スタックを必ず使ってください」という制限や制約はかけていません。理由は健全な新陳代謝の機会を奪わないこと、複数の技術に触れ成長機会を創出したいためです。 ひとつの技術を極めていくことにも良い点がありますが、「変化のための余白」を残しておくことも重要です。しかし、これはあくまで現時点でのスタンスであるため、今後もこのスタンスであり続ける保証はありません。また、ガバナンスを効かせる意味では、新しい技術を導入したい場合にはVPoTやEMなどに事前に相談するようなゆるいガバナンスを効かせています。  
  
詳細はVPoT丸山が書いたブログを見てください [https://tech.classi.jp/entry/2022/07/05/100000](https://tech.classi.jp/entry/2022/07/05/100000)  
  
Classiの開発者ブログは [こちら](https://tech.classi.jp/)

  
  

## dely株式会社

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>iOS：Swift / SwiftUI / Swift Package Manager</td></tr><tr><td>Android：Kotlin / Jetpack Compose</td></tr><tr><td>Serverside： Ruby 3.0 / Rails 7/ Node.js / React</td></tr><tr><th>インフラ</th><td>AWS (ECS, RDS, ElastiCache, DynamoDB, SQS, S3, Lambda), GCP (Bigquery), Vercel</td></tr><tr><th rowspan="4">DB</th></tr><tr><td>DWH：BigQuery / Snowflake</td></tr><tr><td>ETL： dbt / Embulk /Digdag</td></tr><tr><td>BI Tool：: Redash / Looker</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions / CodePipeline / CodeBuild / XcodeCloud</td></tr><tr><th>その他</th><td>GitHub Copilot for Business</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

クラシルリワードは、日常生活をよりお得にするアプリケーションです。通常の移動、特売情報の閲覧、レシートの送信でアプリ内のコインを得られます。貯まったコインは、他社ポイントやデジタルギフト、電子マネーに交換可能です。  
  

**技術選定で意識していること**

チーム内では技術的な議論が活発ですが、流行りの技術をとりあえず採用するようなことはありません。なぜその技術を導入するのか、どのような効果が期待できるのか、運用コストはどうか、代替案は存在するのかなど多角的な議論を踏まえて意思決定を行います。特定の人に技術が依存することなく、多くのメンバーが運用・保守できるよう配慮しています。  
  
dely株式会社のテックブログは [こちら](https://tech.dely.jp/)

  
  

## フリー株式会社

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：TypeScript / JavaScript (Flow) / Babel / webpack / Vite / React</td></tr><tr><td>バックエンド：Ruby / Go / Python / Rails / gRPC</td></tr><tr><td>モバイル：Swift / SwiftUI / Kotlin / Jetpack Compose</td></tr><tr><th>インフラ</th><td>AWS / Kubernetes</td></tr><tr><th>DB</th><td>Aurora MySQL / Redis / Elasticsearch / DynamoDB</td></tr><tr><th>監視・モニタリング</th><td>Datadog / BugSnag</td></tr><tr><th>環境構築</th><td>Terraform / Helm</td></tr><tr><th>CI/CD・テスト</th><td>Argo CD / CircleCI / GitHub Actions</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

freeeは会計・人事労務・販売管理を中心とした統合型クラウドERPを提供しています。有料課金ユーザー企業数は約45万事業所です。  
  

**技術選定で意識していること**

freeeが掲げる「統合型経営プラットフォーム」の実現のためには、多数の社内プロダクト・サービスのボトムアップな協調・連携が重要です。一方で、これによって生じる複雑さをコントロールすることが、継続的な開発のためには必要となります。この課題に対して広い領域での「標準化」に力を入れて取り組んでおり、技術選定はもとより、ログフォーマット、インフラ管理方法、連携用API仕様、さらにはサービス分割の粒度などに対し、ある程度の標準形をガードレールとして定義しています。  
  
フリー株式会社のテックブログは [こちら](https://developers.freee.co.jp/)

  
  

## GO株式会社

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：TypeScript / React / Vue.js</td></tr><tr><td>バックエンド：Go</td></tr><tr><td>iOS：Swift / SwiftUI / needle<br>Android：Kotlin / Jetpack Compose / Dagger Hilt<br>クロスプラットフォームアプリ開発：Dart / Flutter / Riverpod<br></td></tr><tr><th>インフラ</th><td>GCP / AWS / Kubernetes / Docker</td></tr><tr><th>DB</th><td>MySQL / PostgreSQL / Redis</td></tr><tr><th>監視・モニタリング</th><td>Grafana (Mimir / Loki / Tempo)</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

タクシーアプリ『GO』 とは？ タクシー車両とのリアルタイムな位置情報連携と高度な配車ロジックで「早く乗れる」体験を提供するタクシーアプリです。  

- ダウンロード数(23年11月末時点)：1800万
- 年間実車数：6000万回
- 利用可能エリア：45都道府県
- ネットワーク事業者数：1100社以上
  
  

**技術選定で意識していること**

弊社のプロダクトは時間帯や天候によってトラフィックが大きく変動するサービスです。急激なトラフィックを捌くためにも処理パフォーマンスが高い言語で開発する必要があります。  
また、弊社の事業はスピード感がありビジネスロジックも複雑であるため責務が1つのサービスに偏り過ぎないようにマイクロサービス化を積極的に行なってます。月1ペースでマイクロサービスが増えているため、プロダクトエンジニアがサービス開発に専念出来るようにマイクロサービスの標準化・インフラの標準化を積極的に進めています。その結果として、エンジニアがプロダクトの開発に集中できる環境を構築しています。  
  
GO株式会社のテックブログはこちら [こちら](https://lab.mo-t.com/blog)  
  

## GMOペパボ株式会社

<table><tbody><tr><th rowspan="5">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：フロントエンド：HTML / CSS / JavaScript / TypeScript / React / Next.js / Vue.js / Nuxt.js</td></tr><tr><td>バックエンド：Go / TypeScript / PHP / Ruby / Ruby on Rails</td></tr><tr><td>Android：Kotlin / Flutter / Dart / Java / Jetpack Compose / Kotlin Coroutines / Dagger Hilt</td></tr><tr><td>iOS：Swift / SwiftUI / UIKit / Combine</td></tr><tr><th>インフラ</th><td>Nginx / Kubernetes / Sidekiq / Consul /GCP(Google Kubernetes Engine / Google Data Studio / BigQuery / etc) / AWS(Amazon CloudFront / AWS Batch / etc) / Openstack</td></tr><tr><th>DB</th><td>MySQL / PostgreSQL / Redis / Amazon RDS / Amazon Aurora</td></tr><tr><th>監視・モニタリング</th><td>Datadog / Prometheus / Grafana / Mackerel / Sentry / Fluentd</td></tr><tr><th>環境構築</th><td>Terraform / Docker / Ansible / AWS CloudFormation / Chef / Windows Autopilot / Google Cloud Build / AWS Fargate</td></tr><tr><th>CI/CD・テスト</th><td>GitHubAcitons / Argo CD / Bitrise / fastlane / Danger / DeployGate /Renovate</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

GMOペパボは人々の自己表現やアウトプットを支援するインターネットサービスを多数提供しています。1,710万点以上の作品が販売・展示されている国内最大のハンドメイドマーケット「minne」をはじめ、イラストレーターや美術系学生、YouTuber、お笑い芸人から企業まで77万超のクリエイターが登録するオリジナルグッズ作成・販売サービス「SUZURI」など個人から法人まで幅広くご利用いただいています。  
  

**技術選定で意識していること**

GMOペパボは事業のフェーズや規模、性質の異なるサービスを複数有しています。そのため、会社として統一した技術選定を行うのではなく、会社全体で見たときに複数の選択肢を確保しつつ、プロダクト開発を組織として継続させることを意識しています。  
  
[Pepabo Tech Portal](https://tech.pepabo.com/)

  
  

## 株式会社ログラス

<table><tbody><tr><th rowspan="3">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：Next.js (React / TypeScript)</td></tr><tr><td>バックエンド：Spring Boot / Kotlin</td></tr><tr><th>インフラ</th><td>AWS(ECS, Lambda) / Auth0 (IDaaS)</td></tr><tr><th>DB</th><td>PostgreSQL</td></tr><tr><th>監視・モニタリング</th><td>Datadog</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions / AWS CodePipeline / AWS Codebuild / AWS CodeDeploy / AWS Step Functions / AWS Lambda</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

株式会社ログラスでは、社内に散らばる経営データを一元化する経営管理クラウド「Loglass 経営管理」「Loglass IT投資管理」「Loglass 販売計画」等を提供しています。経営企画部門の皆さまが担う、予算策定、予実管理、見込み更新、管理会計のフローを効率的に仕組み化し、柔軟に"次の一手"を打ち出せる機動力を届けます。  
  

**技術選定で意識していること**

ログラスでは「LTV First」というバリューがあり、「長期的に顧客生涯価値を最大化する事を第一にすること」を大切にしています。このバリューを踏まえ、技術選定においても長期的なメリットとコストの両方を考えた上で、最適な意思決定を心がけています。  

また、導入後もきちんと価値とコストを検証し、そのバランスが取れないと判断した技術については、強い意志で撤退を決断します。  

一例ではありますが、長期的な採用のしやすさを考慮してフロントエンドのフレームワークをAngularからReactへリプレイスしています。  

[https://www.wantedly.com/companies/loglass/post\_articles/331280](https://www.wantedly.com/companies/loglass/post_articles/331280)  
このような戦略的かつ持続可能な技術選定を通じて、企業価値の向上を図っています。

  
  

## エムスリー株式会社

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド: Typescript / React / Vue.js / Nuxt.js</td></tr><tr><td>バックエンド: Go / Python / Ruby / Kotlin / Scala / Java</td></tr><tr><td>APP：Kotlin / Flutter / Swift / SwiftUI / UIKit / Dart</td></tr><tr><th>インフラ</th><td>AWS / GCP / オンプレミス / Kubernetes</td></tr><tr><th>DB</th><td>PostgreSQL / MySQL / Oracle / Redis / Elasticsearch / BigQuery、BigTableなど各クラウドサービス</td></tr><tr><th>監視・モニタリング</th><td>DataDog / Grafana / Kibana / Sentry / Fluentd / NewRelic / Prometheus / Zabbix / Qualys / yamory</td></tr><tr><th>環境構築</th><td>Ansible / Terraform / Docker / Packer / AlmaLinux</td></tr><tr><th>CI/CD・テスト</th><td>GitLab CI / GitHub Actions / Renovate / Jenkins / Argo CD / mabl / Selenium</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

エムスリーは医療に関連した100以上のサービス、プロダクトを提供するITベンチャーです。国内医師の9割以上、世界の6割以上の医師が登録するサービス『m3.com』を開発運営しているだけでなく、クリニックDXサービス『デジスマ』や遠隔医療相談サービス『AskDoctors』など患者が利用するサービスも開発しています。東証プライム上場のメガベンチャーでありながら、エンジニアは約100名と少数精鋭です。  
  

**技術選定で意識していること**

エムスリーでは様々な種類、規模の事業が存在するため、上層部が利用言語、プラットフォーム、アーキテクチャを固定するような取り組みは行っていません。  
  
事業のフェーズやプロダクトのゴール、メンバーの技術レベルといった様々な視点から、最適な選択肢を各チームで議論し、各チームが裁量をもって決定しています。  
  
技術選定は、エンジニアリングの視野を事業や人といった観点に広げ、エンジニアとして成長する絶好の機会であるとも考えています。  
  
技術者としてリーダーシップを発揮し、技術的意思決定を繰り返す事が出来る環境を継続することで、将来のCTO/VPoE人材を育成できると考えています。  
  
エムスリー株式会社のテックブログは [こちら](https://www.m3tech.blog/)

  
  

## 株式会社マネーフォワード

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：JavaScript / TypeScript / React / Vue.js / Next.js / Tailwind CSS</td></tr><tr><td>バックエンド：Ruby / Go / Kotlin / Python / Java / Rust / Ruby on Rails / gRPC / Spring Boot</td></tr><tr><td>モバイル<p>Andoroid：Kotlin / Java / Jetpack Compose</p><p>iOS：Swift / Objective-C / SwiftUI</p><p>クロスプラットフォーム：Dart (Flutter)</p></td></tr><tr><th>インフラ</th><td>AWS / GCP / EKS / ECS / Kubernetes (オンプレミス)</td></tr><tr><th>DB</th><td>Aurora MySQL / MongoDB / BigQuery / Redis / DynamoDB</td></tr><tr><th>監視・モニタリング</th><td>Datadog / PagerDuty / Grafana / Prometheus / Kibana</td></tr><tr><th>環境構築</th><td>Terraform / Ansible / Helm</td></tr><tr><th>CI/CD・テスト</th><td>CircleCI / GitHub Actions / Bitrise / Argo CD</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

「お金を前へ。人生をもっと前へ。」をミッションに、すべての人のお金の課題解決を目指し、お金の見える化サービス『マネーフォワードME』やバックオフィスSaaS『マネーフォワードクラウド』などを提供しています。  
  

**技術選定で意識していること**

2023年6月末時点で、マネーフォワードが提供しているサービスの数は 50 を超えています。ユーザーの利便性向上のためにも、各サービスをすばやく改善していくことが重要だと考えています。そのために、私たちはスモールチームを重視し、各チームが自律的に意思決定できる組織体制をつくっています。私たちのサービスには、歴史的な経緯としてRuby on Rails で開発されたサービスが多くあります。近年ではマイクロサービスアーキテクチャに移行しており、Go 言語やサーバーサイドKotlin など環境やサービスの特徴にあった言語選択を行なうようになっています。  
  
株式会社マネーフォワードのテックブログは [こちら](https://moneyforward-dev.jp/)

  
  

## Sansan株式会社

<table><tbody><tr><th rowspan="3">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：Typescript / React / Next.js</td></tr><tr><td>バックエンド：C# (ASP.NET MVC) / Kotlin (Ktor, Spring Boot) / Go / Express / Ruby (Rails) / Python</td></tr><tr><th>インフラ</th><td>AWS / GCP / Azure</td></tr><tr><th>DB</th><td>CloudSQL (PostgreSQL) / Aurora (MySQL) / Elasticsearch / Azure SQL Database, Azure Cosmos DB / BigQuery / Redshift</td></tr><tr><th>運用・監視</th><td>Zabbix / Grafana / New Relic / Amazon Elasticsearch Service / Fluentd / Chef</td></tr><tr><th>CI/CD・テスト</th><td>Jenkins / NUnit / JUnit / GitHub Actions</td></tr><tr><th rowspan="5">その他</th><td>検索エンジン：Elastic Cloud</td></tr><tr><td>API：GraphQL / REST</td></tr><tr><td>認証：Auth0</td></tr><tr><td>開発ツール：GitHub / GitHub Copilot / Figma / Storybook</td></tr><tr><td>コミュニケーション：Teamflow</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**  
「出会いからイノベーションを生み出す」をミッションとして掲げ、働き方を変えるDXサービスを提供しています。主なサービスとして、営業DXサービス「Sansan」や名刺アプリ「Eight」、インボイス管理サービス「Bill One」、契約データベース「Contract One」を国内外で提供しています。「Sansan」の契約件数は9000件を超え、ARRは200億円を突破しました（2023年6月時点）。「Bill One」の有料契約件数は2300件、ARRは59億円となっています（2023年11月時点）。事業のさらなる加速のため、日々プロダクトをアップデートし続けています。  
  

**技術選定で意識していること**  
Sansan株式会社ではプロダクトごとに適切な技術の選定を行うことを基本方針としています。 プロダクトの性質や、成熟度に応じて適切に技術の選択を行い、プロダクトの成長スピードを最大化することを意識しています。 技術に関する意思決定だけでなく、あらゆる開発業務において、その選択をする目的を重視しています。 例えば、特に立ち上げフェーズのプロダクトにおいては、積極的に技術の共有を行い、開発スピードを向上させると同時に、ある程度機能が分化した段階で独立させていく戦略もとっています。

  
  
Sansanのテックブログは [こちら](https://buildersbox.corp-sansan.com/)  
  

## 株式会社SmartHR

| 言語   フレームワーク | Ruby / TypeScript / Ruby on Rails / React / Next.js / SmartHR UI |
| --- | --- |
| インフラ | Google Cloud |
| DB | Cloud SQL / Memorystore |
| 監視・モニタリング | Sentry / New Relic |
| 環境構築 | Terraform |
| CI/CD・テスト | CircleCI |

### 技術選定者のコメント

**事業・プロダクトの規模**

クラウド人事労務ソフト「SmartHR」を提供しています。「すべての人が気持ちよく働ける」社会を目指して、SmartHRは人事・労務業務の効率化からタレントマネジメントまで、シームレスに実現します。  
  

**技術選定で意識していること**

基本機能と各種オプション機能は別リポジトリで管理・開発され、それぞれがREST API や GraphQL で通信する構成となっています。 社内で用意しているSDKや知見を共有して素早く高品質な開発を進めるために、基本となる技術スタックを Ruby on Rails / TypeScript / React で統一しています。 バックエンドは可能な限り Rails Way に則るように意識して開発していますが、フロントエンドは最適な構成を模索中のため、Redux を使っていたり Next.js を使ったりと、各プロダクトで試行錯誤を続けています。 また、各プロダクトで統一されたユーザ体験を提供するために、SmartHR UI を開発し、OSS として公開しています。  
  
株式会社SmartHRのテックブログは [こちら](https://tech.smarthr.jp/)

  
  

## 株式会社タイミー

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：<br>- 開発言語: TypeScript<br>- アーキテクチャ: Next.js CSR(SPA) / React Hooks / SWR<br>- ツール・ライブラリ: Jest / Storybook, reg-suit / Emotion / React Testing Library / SWR / OpenAPI / ESLint / Stylelint / Markuplint / Prettier / Prism</td></tr><tr><td>バックエンド：<br>- 開発言語: Ruby 3.2系<br>- アーキテクチャ: Ruby on Rails 7.0系 / RSpec</td></tr><tr><td>モバイル：<br>- 開発言語：Swift<br>- アーキテクチャ：Flux / Layered Architecture<br>- 外部サービス：Firebase / Adjust / Braze / etc…<br>- ツール・ライブラリ：SwiftUI / Swift Concurrency / RxSwift / Fastlan / Bitrise / OpenAPI(Swagger)<br>- その他：Only Swift Package Manager(No CocoaPods) / Multi-module development（40+）</td></tr><tr><th>インフラ</th><td>AWS(ECS Fargate / Aurora / RDS / S3 / ElastiCache /CloudFront / etc...) 一部のサービスでGCPを利用 / TerraformによるIaC /ログはDatadog LogsとS3に集約</td></tr><tr><th>DB</th><td>MySQL 5.7(Aurora) / Redis</td></tr><tr><th>監視・モニタリング</th><td>Datadog / Sentry</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>CircleCI / GitHub Actions / Dependabot / ecspresso</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

働きたい時間と働いてほしい時間をマッチングするスキマバイトサービス「タイミー」を提供しています。現在ワーカー（働き手）数は600万人、導入事業者数は6万6千社に急成長しています。（2023年10月時点）  
  

**技術選定で意識していること**

タイミーは単一のプロダクトが大きな顧客価値を産むプロダクトであり、またマッチングから給与支払い、労務管理までを一貫して行うことができる、複雑なプロダクトです。このような複雑で大規模なプロダクトを、早く安定して変更できるような技術選定を心がけています。

  
  

## Ubie株式会社

<table><tbody><tr><th rowspan="4">言語<br>フレームワーク</th></tr><tr><td>フロントエンド：TypeScript / Next.js</td></tr><tr><td>バックエンド：TypeScript / Node.js / NestJS / GraphQL / Kotlin / Spring</td></tr><tr><td>共通基盤：Go / gRPC</td></tr><tr><th>インフラ</th><td>Google Cloud / docker / Fastly</td></tr><tr><th>DB</th><td>PostgreSQL / Google Cloud Spanner</td></tr><tr><th>監視・モニタリング</th><td>Sentry</td></tr><tr><th>環境構築</th><td>Terraform</td></tr><tr><th>CI/CD・テスト</th><td>GitHub Actions / mabl</td></tr></tbody></table>

### 技術選定者のコメント

**事業・プロダクトの規模**

テクノロジーで人々を適切な医療に案内する」をミッションに掲げ、症状から適切な医療へと案内する「ユビー」と、診療の質向上を支援する医療機関向けサービスパッケージ「ユビ―メディカルナビ」等を開発・提供しています。現在280名規模で、内60名ほどがエンジニアです。  
  

**技術選定で意識していること**

会社として事業を急速にスケールさせるフェーズに入ったので、社内の多くのエンジニアが開発に関わることを想定した言語・フレームワーク選定を行っています。そのため、キャッチアップのしやすさ、書き方の統一感、エコシステムの発展状況などを重視しています。 一見すると一般的な技術スタックではありますが、事業ドメインや各技術同士の組み合わせによって挑戦することの難しさや面白さがあると思うので、すごく尖った選定にはあえてしていません。  
  
Ubie株式会社のテックブログは [こちら](https://blog.ubie.tech/)