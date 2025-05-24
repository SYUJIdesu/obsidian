---
title: ニーズに合わせてツールを選ぶ - CloudFormationからTerraformへ移行する理由｜AY
source: https://note.com/quiet_crocus4147/n/n0b7922090f13
author:
  - "[[note（ノート）]]"
published: 2024-12-05
created: 2025-05-15
description: 本記事は、Japan Digital Design Advent Calendar 2024 の5日目の記事になります。       adventar.org   三菱UFJフィナンシャル・グループ（以下MUFG）の戦略子会社であるJapan Digital Design（以下JDD）でインフラエンジニアをしている 山我です。  この記事では、CloudFormationからTerraformに移行するに至った経緯について紹介いたします。  こんな人向け    CloudFormationを利用しているが、Terraformも気になっている人    IaCの実装
tags:
  - web
  - AWS
  - Ioc
---
![見出し画像](https://assets.st-note.com/production/uploads/images/164093302/rectangle_large_type_2_06e3fc991ba7871f87c107c5a02780e3.png?width=1200)

## ニーズに合わせてツールを選ぶ - CloudFormationからTerraformへ移行する理由

[AY](https://note.com/quiet_crocus4147)

本記事は、 [Japan Digital Design Advent Calendar 2024](https://adventar.org/calendars/10000) の5日目の記事になります。

三菱UFJフィナンシャル・グループ（以下MUFG）の戦略子会社であるJapan Digital Design（以下JDD）でインフラエンジニアをしている 山我です。

この記事では、CloudFormationからTerraformに移行するに至った経緯について紹介いたします。

## こんな人向け

- CloudFormationを利用しているが、Terraformも気になっている人
- IaCの実装を検討している人

## そもそもCloudFormationを採用していたのはなんで？

### AWSの新機能やサービスにいち早く対応している

JDDは新機能を早く取り入れることがあるため。

### AWSサポートに問い合わせできる

AWS純正のサービスであるため、サポートに直接問い合わせが可能である。

### ロールバックに対応している

スタック失敗時に作成できていたリソースも消えて、元の状態に戻って欲しかった。

## CloudFormationから変更しようと考えたのはなんで？

運用をしていく中で、主にリソース変更に関することで以下のことを課題に感じていた。

### ChangeSetの変更差分がわかりづらい

JSON形式のリソース定義は一部の変更でもプロパティ全体が変更箇所となり、実際の変更箇所を判別するのに時間がかかることがある。

![画像](https://assets.st-note.com/img/1732094788-VqRsGJhWoO1uYin83Ud5DMxQ.png?width=1200)

アクションを追加した場合の差分

### ネストされたスタックの場合にリソースが確認しづらい

リソース一覧からは \`AWS::CloudFormation::Stack\` となり、作成したリソースの一覧を確認することはできないため、呼び出しているスタックを見に行く必要がある。

![画像](https://assets.st-note.com/img/1732084103-CFvhiwY82l3RXjNQKg16Wpc5.png?width=1200)

ネストされたスタックのリソース情報

### GitHub Actionを利用したCI/CDの実装がシンプルではない

コードレビューに必要な情報をGitHub上に集約することで、効率化を図りたいが、CloudFormationの処理が複雑化する課題がある。

**ネストされたスタックと通常のスタックで利用するコマンド**

- ネストされたスタックの場合、 \`aws cloudformation deploy\` の前に \`aws cloudformation package\` を実行する必要があり、スタックの種類に応じた分岐を設けなければならない。

**変更セット(変更差分)の結果をGitHubにコメントとして出力する処理**

- \`aws cloudformation deploy\` のオプション \`--no-execute-changeset\` を用いることで変更セットを作成できるが、レスポンスは変更セットを確認するためのコマンドであるため、もう一度コマンドを実行する必要がある。
- 変更セットの結果はJSONであるため、GitHubのコマンド用に加工をする必要がある。

```php
# 変更セット表示サンプル (ここではGitHubへのコメントは除き、変更セットの表示のみ抜粋)

# 変数設定
template_file=xxx.yaml
stack_name=xxx

# 変更セットを作成 tailを使っているのはJSON等で変更セット確認用のコマンドを実行できないため
change_set_command=\`aws cloudformation deploy --template-file ${template_file} --stack-name ${stack_name} --capabilities CAPABILITY_NAMED_IAM  --no-execute-changeset | tail -n 1\`

# 変更セットの内容を取得
change_set=$(eval ${change_set_command} --include-property-values | jq -c '[{StackName: .StackName, Changes: [.Changes[] | {Type: .ResourceChange.ResourceType, LogicalResourceId: .ResourceChange.LogicalResourceId, PhysicalResourceId: .ResourceChange.PhysicalResourceId, Action: .ResourceChange.Action, Replacement: .ResourceChange.Replacement, Before: .ResourceChange.BeforeContext, After: .ResourceChange.AfterContext}]}]')

# 変更セットの表示
stack_details=$(echo $change_set | jq -r --arg stack "$stack_change" '.[] | select(.StackName == $stack)')
for resource_change in $(echo $stack_details | jq -r '.Changes[].LogicalResourceId'); do
   resource_details=$(echo $stack_details | jq -r --arg resource "$resource_change" '.Changes[] | select(.LogicalResourceId == $resource)')
   echo "-----------------"
   echo "Type:" $(echo $resource_details | jq -r '.Type')
   echo "PhysicalResourceId:" $(echo $resource_details | jq -r '.PhysicalResourceId')
   echo "Action:" $(echo $resource_details | jq -r '.Action')
   echo "Replacement:" $(echo $resource_details | jq -r '.Replacement')
   echo "Before": $(echo $resource_details | jq -r '.Before')
   echo "After": $(echo $resource_details | jq -r '.After')
done
```

```php
# 実行サンプル

Type: AWS::S3::Bucket
PhysicalResourceId: null
Action: Add
Replacement: True
Before: null
After: {"Properties":{"BucketName":"BucketName","VersioningConfiguration":{"Status":"Enabled"},"BucketEncryption":{"ServerSideEncryptionConfiguration":[{"ServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}}}
```

## Terraformを選択した理由はなんで？

下記のことからこれからもCloudFormationで運用し続けるよりも、Terraformへの移行するメリットが大きそうだと考えた。

### 社内にTerraformユーザーが増えてきている

- 弊社にJOINするメンバーがCloudFormationよりもTerraformでの運用経験を持っている人が大半であったため、学習効率の面、採用の面でも利点があると考えた。

### Terraformで管理できるサービスの利用が社内に増えてきている

- Google CloudやDatabricksなどの利用が増えてきており、それらもTerraformで管理できるため、社内のツール統一化が図れる。
- 別のサービスでも同様のコマンドを利用できるため、CI/CDのメンテナンス性も向上する。

### 社外の情報やモジュールが豊富

- 変更を考えるきっかけとなっている「GitHub Actionを利用したCI/CD」も [tfcmt](https://github.com/suzuki-shunsuke/tfcmt) を用いることで、簡単に GitHubへの通知ができる。
- そのほかにも様々なモジュール（ [tfsec](https://tfsec.dev/) 、 [tflint](https://github.com/terraform-linters/tflint) 、 [reviewdog](https://github.com/reviewdog/reviewdog) ）が多く存在する。
- 社外の情報も豊富でTerraformを利用したGitHub Actions CI/CD について情報公開していたりする。

### 既存リソースのImportが容易

**Terraformが管理しているリソースの状態を表すファイル\`.tfstate\`の作成が容易**

- [terraform import](https://developer.hashicorp.com/terraform/cli/import)  コマンドや Terraform v1.5.0以上が利用可能な  [importブロック](https://developer.hashicorp.com/terraform/language/import) によって、 \`tfstate\` の作成ができる。

**リソース定義ファイルの作成が容易**

- importブロックを使っている場合、\`terraform plan -generate-config-out\`コマンドを実行すれば、インポート予定のリソース定義ファイルを作成することができる。  
	※ リソースによっては、競合できないパラメータを出力する可能性があるため、Planを実行し、内容の妥当性を確認する必要はある。

### CloudFormationの採用理由となっている部分もカバーできそう

**「AWSの新機能やサービスにいち早く対応している」に対して**

- AWS Provider は活発に活動しており、新機能のサポートもそれなりに早いと考えている。
- 緊急で即時対応していかなければならない場合は、 Terraformの \`aws\_cloudformation\_stack\` を利用し、CloudFormationスタックを管理することで構成の管理は可能である。

**「AWSサポートに問い合わせできる」に対して**

- 多くのユーザーが利用しているため、珍しい使い方をしない限りはIssueの活用で問題ないと考えている。
- Terraformで解決が難しいものは、新機能と同様にCloudFormationスタックを利用して対応可能だと考えている。

**「ロールバックに対応している」に対して**

- Terraformは現在の環境と指定されたコードの差分を判断し、必要な変更だけを適用するので、GitのRevert機能と組み合わせることで過去の任意の時点の設定状態に戻すことが可能になる。
- ただし、CloudFormationと比較すると、ロールバックするのにタイムラグが発生することや、中途半端に適用される可能性は発生してしまう。

## CDKはどう考えた？

以下の理由でCDKの利用よりもTerraformの方がメリットがあると判断した。

- Terraformを採用した理由にも記載した通り、AWS以外のクラウドプロバイダーを利用しているため、AWSだけCDKにするメリットは低かった。
- 選択したプログラミング言語がメンバーのスキルセットによっては学習コストになってしまう。

## まとめ

今回の CloudFormation から Terraform への移行には、以下の理由がありました。

### CloudFormationから変更しようと考えた理由

- 変更差分の可視性が低い
- ネストされたスタックの管理が複雑
- CI/CD の実装が複雑

### Terraform を選んだ理由

- チームのスキルセットとの整合性
- マルチクラウド環境への対応
- 豊富な外部リソースとコミュニティサポート
- CI/CD の簡素化

## 最後に

自分たちのニーズと環境を正確に把握し、定期的に見直すことが大切で、今の状況と将来の目標を見て考えて見るのが大事なのかもしれません。  
そうすることで、より効率的なインフラ管理ができるのではないかと思いました。

今後の展望として、私たちが実際にCloudFormationからTerraformへの移行を行った手順についても、公開できたらと思います。移行のプロセスや、その際のポイントなどを解説していきますので、ぜひご期待ください。

以上、山我でした。  
最後までご覧いただきありがとうございました。

\---

Japan Digital Design株式会社では、一緒に働いてくださる仲間を募集中です。カジュアル面談も実施しておりますので下記リンク先からお気軽にお問合せください。

[https://japan-d2.com/careers](https://japan-d2.com/careers)

この記事に関するお問い合わせはこちら

[https://japan-d2.com/contact](https://japan-d2.com/contact)

Technology & Development Div.  
Akihiro Yamaga

ニーズに合わせてツールを選ぶ - CloudFormationからTerraformへ移行する理由｜AY