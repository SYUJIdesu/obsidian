---
title: "Devinにコードレビューをさせ、コード品質と開発速度を同時に高める話"
source: "https://zenn.dev/globis/articles/28e47f8107c5b5"
author:
  - "[[Zenn]]"
published: 2025-03-11
created: 2025-05-15
description:
tags:
  - "web"
  - "Devin"
---
[GLOBIS Tech](https://zenn.dev/p/globis) [Publicationへの投稿](https://zenn.dev/faq#what-is-publication)

142

55[tech](https://zenn.dev/tech-or-idea)

近年、Devinは「AIコーダー（自動コーディング）」として注目を集めています。しかし実は、「AIレビュアー」として活用することで、コード品質の向上・チームを跨いだ知見の横展開など、さらなるメリットを得られるのをご存じでしょうか。

ジュニアエンジニアレベルの実装タスクを任せられる一方、AIレビュアーとしてのDevinはシニアエンジニア並みの指摘を行えるのが特徴です。その背景には、「知見の蓄積」と「コードレビュー」の相性の良さがあります。私たちのチームではまだ導入を始めて日が浅いものの、すでに「バグの事前防止」「望ましい設計パターンの浸透」といった効果を感じ始めています。

本記事では、こうした初期段階での手応えを踏まえ、DevinをAIレビュアーとしてどのように導入したのか、具体的なチューニングのコツを交えてご紹介します。また、GitHub Actions用に実際に利用しているYAMLも載せているので、是非試してみてください！

## 導入の背景

グロービスでは以下のような課題を抱えていました。

- 組織内でチームが細分化されることによって、「過去に経験済みの失敗」や「典型的な落とし穴」「好ましい設計」が横断的に共有されず、同じ問題が何度も繰り返されてしまう
- 現場で培われた実践的な知恵がチームごとに閉じてしまい、なおかつ一時的・不定期な業務においては忘れられがちになる
- 結果として、コードレビューの品質にバラつきが生じ、本来なら事前に防げるはずのバグがリリース後に発覚したり、望ましい設計パターンから逸脱したコードが少しずつ増加したりしていました。

人間によるコードレビューやドキュメント整備も行いましたが、どうしても属人的な差が生まれ、結果的に知識が散在してしまう課題は解消しきれませんでした。また、ドキュメントの性質上、継続的にメンテナンスされないパターンや、そもそも参照すること自体を忘れてしまうこともあります。

そこで、AIレビュアー（Devin）を導入したことで、問題の未然防止という「マイナスをゼロにする」効果にとどまらず、「理想的なビジネスロジックの配置」や「推奨設計パターンの積極的な提示」といった、より洗練された設計を促進する「プラス方向」の改善を目指しました。Devinを通して蓄積した知識をレビューで活用することで組織全体への浸透を促すことができ、データとしても良い傾向が見られています。

## 導入の定量的成果

Devin導入後に対象とした **45件のPR** のうち、 **25件（約55%）はそもそも指摘の余地がなく十分に高品質でした。  
一方、指摘の余地があった20件** のPRのうち、 **10件** はDevinが不具合や設計上の問題を指摘し、修正につなげることができました。  
これは「明確な問題があるPRのうち、およそ半数をDevinが自動的に検知し、人間によるレビュー前のチェックに成功している & エンジニアの手戻りを減らしている」と言えます。

| 指摘カテゴリ | 内容例 | 指摘件数（件） | 全体に占める割合（%） |
| --- | --- | --- | --- |
| バグ防止 | 潜在的な不具合やエラーの指摘 | 6 | 15.79% |
| パフォーマンス改善 | 複合インデックスの追加や順序変更の提案や、利用頻度が「高そう」なカラムへのインデックス追加など | 3 | 7.89% |
| 設計・リファクタリング | Fat Model化やビジネスロジックの整理提案、不要なサービス層の廃止など | 5 | 13.16% |
| ドメイン特化 | stg環境における個人情報マスキング、db-mask-infraとの連携など | 4 | 10.53% |
| 可読性・メンテナンス性 | 命名の改善、冗長なコードの整理、複雑なロジックの単純化など | 15 | 39.47% |
| その他 | 上記に含まれない指摘（例えば、ケアレスミスなどによる対応漏れも含みます） | 5 | 13.16% |
| **合計** |  | **38** | **100%** |

## チューニング

チューニングにあたっては、大きく以下の3点を重視しました。また、これらを管理するための「プロンプト」と「Knowledge」の使い分けも工夫しています（後述します）。

- レビュー時のお作法を守らせる（AIは細かい作業に分解することが苦手な場合が多いため、ファイル単位でのチェックや処理の追い方といった手順をインプットしています）
- Railsアプリケーションにおける理想的な設計パターンをレビュー時に定着させる
- グロービス特有のドメイン知識・運用ルールを活用させる

これらの観点で情報整理を行うことにより、「一般的に望ましい設計や実装方法」だけでなく、「自社特有の事情を反映した指摘」を自然に提示できるようになります。

また、知識の初期構築に多くの時間を割くことなく、実際に使いながらDevinにフィードバックを行い徐々に精度を高めていけるため、最初は小さく始めることを意識しました。

## Knowledgeとプロンプトの使い分け

前提としてDevin自体が自律性を持っているため、あまり複雑なチューニングはしておらず、以下の2つのみに手を加えています。

1. DevinのKnowledge
2. Devinへのプロンプト (実際のGithub Actions用YAML付き)

管理のしやすさを重視し、変動が大きく頻繁にアップデートする必要のある内容（たとえばRailsの設計パターンや自社特有のドメイン知識など）は、DevinのKnowledgeに集約しています。一方、レビュー時のお作法やチェックリストの作成手順といった、あまり頻繁に変わらない指示事項や基本ルールはプロンプト側に記載することで使い分けています。

基本的にKnowledgeの内容は、Devin自身が自動で提案したものをベースとして、加筆・修正をしています。Devinに直接フィードバックを返すことで、お作法やドメイン知識がKnowledgeとして自然に蓄積されているのが非常に良い体験ですね。

## Knowledge: Railsの理想的な設計パターンを学習させる

グロービスでは、いわゆる“Fat Model”の考え方に基づき、ビジネスロジックはできるだけモデルに集約し、ジョブやコントローラの責務は最小限にすることを目指しています。そのため、DevinのKnowledgeには以下のようなRailsの基本思想やベストプラクティスをまとめました。

- 「DHHスタイル」の設計原則（モデルにビジネスロジックを集約する）
- コールバックの乱用を避ける／コールバックを使う場面では可読性を重視する
- サービスオブジェクトは使わない、など

こうした設計パターンの事例を多数集め、Devinに事前学習させることで、適切なリファクタリングポイント（「この処理はモデルに寄せるべき」「サービス層を使うべきではない」など）を指摘できるようにしました。

## Knowledge: グロービス特有のドメイン知識の蓄積

グロービス特有の運用ルールやドメイン知識をDevinのKnowledgeに組み込むことも重視しました。たとえば以下のような情報は実装者だけでなく人間のレビュアーでも見落としやすく、むしろエンジニアが注意を払うコストは勿体ないので、なるべくAIレビューに寄せました。

- stg環境にリストアする際の個人情報マスキング
- （古くから存在しており）取り扱いに注意が必要なモデル
- 社内のデータ基盤など、他リポジトリが影響を受ける領域

このあたりのノウハウをDevinに覚えさせるために、まずは社内Wikiや過去のPRレビューコメント、トラブルシューティングの履歴などを抽出し、ドキュメント化しました。それをもとにプロンプトや追加スクリプトで読み込ませることで、データベース移行や個人情報の扱いなど、グロービスの運用に即したコメントが生成されるようになります。

## プロンプト: Plannerを意識したプロンプト設計

Plannerは、DevinのようなAIエージェントが与えられたタスク（例：コードレビュー）を段階的・体系的に整理し、抜け漏れなく実行するための仕組みです。例えば「PRの変更内容をチェック → 問題点を洗い出す → 推奨事項をリストアップ → 必要があれば既存知識を参照して深掘り」という形で、タスクをいくつかのステップに分解し、チェックリストや実行計画を自動で作成してくれます。

プロンプトは自由度高く記述できますが、よりレビュー精度を高めるために、Devinが利用するPlanner用のプロンプトにもチューニングを加えています。

下図はその一例で、シンプルな変更の場合でも漏れなくレビューが行えるよう、ファイルごとに明確なチェックリストを自動作成させています。

![](https://storage.googleapis.com/zenn-user-upload/de4d6b502220-20250310.png)

Plannerを上手くチューニングすることでレビュー品質をさらに高められるため、さまざまな応用が可能です。ここでは詳細には踏み込みませんが、「Plannerのチューニングも効果的である」ということをおさえておくと幅が広がるはずです。

## 使い始めるには

GitHub ActionsでDevinを使い始めるためには、事前にAPIキーを取得し、Actions内で参照できるように設定する必要があります。  
以下の手順で簡単にセットアップが可能です。

1. まず、 [DevinのAPIキー設定ページ](https://app.devin.ai/settings/api-keys) にアクセスし、APIキーを生成します。
2. 次に、生成されたAPIキーを利用し、GitHubの設定画面にて以下の通り環境変数を設定します。
- Name: `DEVIN_API_KEY`
- Secret: （先ほど生成したAPIキーを貼り付ける）

これだけですぐにGitHub Actions内でDevinを利用できるようになります。

### 実際のファイル

現在のGitHub Actionsのワークフローは、以下のタイミングで起動します。

- PRが新規作成された時 (`opened`)
- PRに新しいコミットが追加された時 (`synchronize`)
- PR本文が編集され、かつ本文に `devin-review` というキーワードが追加された時 (`edited`)

また、今後はさらに使いやすさを高めるため、PR本文に書かなくても、PRコメント欄で `devin-review` と書くだけでレビューが開始されるような改善も予定しています。こうした工夫を通じて、多くのエンジニアが手軽にDevinを利用できる環境を整えていく予定です。

実際に使用しているYAMLファイルは以下の通りです。

devin-review.yaml

```yaml
name: Devin Code Review

on:
  pull_request:
    types: [opened, synchronize, edited]

jobs:
  code-review:
    runs-on: ubuntu-latest
    env:
      DEVIN_API_KEY: ${{ secrets.DEVIN_API_KEY }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683
        with:
          fetch-depth: 0
          ref: ${{ github.event.pull_request.head.sha }}

      - name: Devin review combined
        if: contains(github.event.pull_request.body, 'devin-review')
        run: |
          session_comment=$(gh pr view "${{ github.event.number }}" --json comments --jq '.comments[] | select(.body | contains("Devin Session ID:")) | .body' | head -n 1)

          create_session() {
            local prompt="$1"

            local response
            response=$(curl -s -w "\\\\\\\\n%{http_code}" -X POST "<https://api.devin.ai/v1/sessions>" \\\\\\\\
                      -H "Authorization: Bearer $DEVIN_API_KEY" \\\\\\\\
                      -H "Content-Type: application/json" \\\\\\\\
                      -d "{\\\\\\\\"prompt\\\\\\\\": \\\\\\\\"$prompt\\\\\\\\"}")

            local http_status
            http_status=$(echo "$response" | tail -n 1)
            echo "Request info: status code = $http_status"

            if [ "$http_status" -ne 200 ]; then
              exit 1
            fi

            local http_body
            http_body=$(echo "$response" | sed '$d')

            local session_id
            session_id=$(echo "$http_body" | jq -r '.session_id')
            if [ -z "$session_id" ] || [ "$session_id" = "null" ]; then
              echo "Failed to get session ID, body = $http_body"
              exit 1
            fi
            gh pr comment "${{ github.event.number }}" --body "Devin Session ID: $session_id"
          }

          update_session() {
            local session_id="$1"
            local message="$2"
            local response
            response=$(curl -s -w "\\\\\\\\n%{http_code}" -X POST "<https://api.devin.ai/v1/session/${session_id}/message>" \\\\\\\\
                      -H "Authorization: Bearer $DEVIN_API_KEY" \\\\\\\\
                      -H "Content-Type: application/json" \\\\\\\\
                      -d "{\\\\\\\\"message\\\\\\\\": \\\\\\\\"$message\\\\\\\\"}")

            local http_status
            http_status=$(echo "$response" | tail -n 1)
            echo "Request info: status code = $http_status"

            if [ "$http_status" -ne 200 ]; then
              exit 1
            fi
          }

          instructions="【レビュー時の指示】\\\\\\\\n1. Knowledgeの活用:\\\\\\\\n  - レビュー時は、最大限Knowledgeを利用すること。\\\\\\\\n2. Plan作成とチェックリスト:\\\\\\\\n  - Planを作るときに、ファイルごとにチェックリストを作り、既存の指摘も列挙する。\\\\\\\\n  - その上で、他の観点で指摘できることがないかチェックする（すでに指摘されているものは指摘しない）。\\\\\\\\n3. 完了時の対応:\\\\\\\\n  - レビューが完了したら、必ずその旨をGitHub上でコメントする。\\\\\\\\n  - 問題がなかった箇所は、手短に列挙する（追加の理由説明は不要）。\\\\\\\\n4. 状況の確認:\\\\\\\\n  - コメントのやり取りを再確認し、最新の状況を把握する。\\\\\\\\n5. 重複防止:\\\\\\\\n  - 既に指摘した内容は再度指摘しない（ghコマンドで既存の指摘を確認する）。\\\\\\\\n6. GitHub連携:\\\\\\\\n  - Devin.ai上での返信だけでなく、必ずghコマンドを利用してGitHub上にコメントを投稿する。ただし、レビューが完了した旨のメッセージは、重複して投稿しないよう注意する。"

          newly_added_devin_review=false
          if [ "${{ github.event.action }}" = "edited" ] && [ -n "${{ github.event.changes.body.from }}" ]; then
            # 変更前の本文に devin-review が含まれていたかどうか
            if ! echo "${{ github.event.changes.body.from }}" | grep -q "devin-review"; then
              newly_added_devin_review=true
            fi
          fi

          if [ "$newly_added_devin_review" = "true" ] || [ "${{ github.run_attempt }}" -gt 1 ]; then
            # 手動での再試行
            base_prompt="${{ github.event.number }}をもう一度、0からレビューしてください。"
            additional="レビュー範囲については、すべての差分を対象にしてください。"
            prompt="$base_prompt $instructions $additional"

            if [ -z "$session_comment" ]; then
              create_session "$prompt"
            else
              session_id=$(echo "$session_comment" | sed -E 's/.*Devin Session ID: (devin-[a-zA-Z0-9]+).*/\\\\\\\\1/')
              update_session "$session_id" "$prompt"
            fi

          elif [ "${{ github.event.action }}" = "synchronize" ] && [ "${{ github.run_attempt }}" -eq 1 ]; then
            # 新規コミットなどが入った場合の自動レビュー
            if [ -z "$session_comment" ]; then
              exit 0
            fi
            session_id=$(echo "$session_comment" | sed -E 's/.*Devin Session ID: (devin-[a-zA-Z0-9]+).*/\\\\\\\\1/')
            base_prompt="${{ github.event.pull_request.html_url }} に変化がありました。再度レビューしてください。"
            additional="レビュー範囲については、基本あなたが前回見たCommitから、最新のCommitまでの範囲で十分です。ただし gh pr diff ${{ github.event.number }} などを使い、PRで変更のあったファイルに限定しレビューしてください。（ただ単にmasterをPRにマージした = ブランチを最新化しただけである場合は、merge commitの差分をレビューしないようにしてください。）"
            prompt="$base_prompt $instructions $additional"
            update_session "$session_id" "$prompt"

          else
            if [ -z "$session_comment" ]; then
              base_prompt="${{ github.event.pull_request.html_url }} をレビューしてください。"
              additional=""
              prompt="$base_prompt $instructions $additional"
              create_session "$prompt"
            else
              exit 0
            fi
          fi
```

## 実例: バグになり得た箇所を事前に指摘

コードの一部（不要な部分は `...` で省略しています）がこちらです。外部サービス連携用のジョブを実行する際、連携先へのリクエストを組み立てる処理が含まれています。

## 元のコード

```ruby
class ExternalService::Contact::BulkUpdateUserNameJob < BaseJob
  sidekiq_options queue: :low_priority

  def perform(contact_ids_cache_key)
    ...
    external_service_contacts.each do |contact|
      updated_name_attrs = user_name_attributes(contact)
      ...
      bulk_update_payload << ExternalService::Schemas::Bulk::Contact.new(
        id: contact.object_id,
        properties: updated_name_attrs
      )

      request_body = JSON.parse(contact.request_body, symbolize_names: true)
      filtered_attrs = updated_name_attrs.except(:firstname, :lastname, :firstname_kana, :lastname_kana)
      request_body[:properties].merge!(filtered_attrs)
      contact.request_body = request_body.to_json
    end
    ...
  end

  private

  def user_name_attributes(contact)
    ...
  end

  def external_service_contact_ids(contact_ids_cache_key)
    ...
  end
end
```

## Devinの指摘

1. プロパティの削除方法について

```ruby
filtered_properties = new_name_properties.except(:firstname, :lastname, :firstname_kana, :lastname_kana)
request_body[:properties].merge!(filtered_properties)
```

現在の実装では `except` を使用して新しいプロパティから古い名前を除外していますが、これだと `request_body` に既に存在する古いプロパティは削除されません。以下のように `delete_if` を使用して明示的に削除することを推奨します：

```ruby
request_body[:properties].merge!(new_name_properties)
request_body[:properties].delete_if { |key, _| [:firstname, :lastname, :firstname_kana, :lastname_kana].include?(key.to_sym) }
```

1. テストケースの網羅性について  
	既存のプロパティが正しく削除されることを確認するテストを追加することを推奨します。現在のテストは新しいプロパティの追加は確認していますが、古いプロパティの削除については十分にカバーできていない可能性があります。

## Devinの指摘の評価

Devinの指摘により、リクエストボディに古いプロパティが残りバグを引き起こすことを指摘できており素晴らしいですね。プロダクトチームにも確認したところ「リリースされていたらバグになっていたので、事前検知ができありがたい」とコメントをいただきました。

## 実例: ビジネスロジックをモデルへ移動するよう指摘

もう一つの例として、期限切れの `GroupProposalUser` をマスクするジョブのコードをDevinがレビューしたケースを紹介します。

## 元のコード

```ruby
module GroupProposals
  class MaskGroupProposalUsersJob < BaseJob
    def perform
      ActsAsTenant.with_tenant(ContentPlan.find_by!(slug: ContentPlan::Slugs::HODAI_SLUG)) do
        expired_group_proposals = GroupProposal.where(expires_at: (1.day.ago).all_day, corporate_application_id: nil)

        expired_group_proposals.find_each do |group_proposal|
          group_proposal.group_proposal_users.each do |group_proposal_user|
            masked_email = format(GloplaUser::EMAIL_MASKING_FORMAT, group_proposal_user.id, Date.current.strftime("%Y%m%d"))
            group_proposal_user.assign_attributes(
              email: masked_email,
              first_name: User::MASKED_NAME,
              last_name: User::MASKED_NAME,
              first_name_kana: User::MASKED_KANA,
              last_name_kana: User::MASKED_KANA
            )
            group_proposal_user.save(validate: false)
          end
        end
      end
    end
  end
end
```

## Devinの指摘

- 全体的なフィードバック

このPRでは、期限切れのGroupProposalUserの個人情報をマスクするジョブが実装されています。実装自体は機能的に問題ないと思われますが、DHHスタイルのRailsパターンの観点からいくつか改善点があります。

- ビジネスロジックのモデルへの移動

現在の実装では、ビジネスロジック（期限切れGroupProposalの検索とGroupProposalUserのマスキング処理）がジョブクラスに配置されています。DHHスタイルのRailsパターンでは、ビジネスロジックは必ずモデルに配置することが推奨されています。

以下のようにリファクタリングすることを提案します：

- GroupProposalUserモデルにマスキングメソッドを追加

```ruby
# app/models/group_proposal_user.rb
class GroupProposalUser < ApplicationRecord
  # 既存のコード...

  # 個人情報をマスクするメソッド
  def mask_personal_information!
    masked_email = format(GloplaUser::EMAIL_MASKING_FORMAT, id, Date.current.strftime("%Y%m%d"))

    assign_attributes(
      email: masked_email,
      first_name: User::MASKED_NAME,
      last_name: User::MASKED_NAME,
      first_name_kana: User::MASKED_KANA,
      last_name_kana: User::MASKED_KANA
    )

    save(validate: false)
  end
end
```

- GroupProposalモデルに期限切れユーザーを検索するクラスメソッドを追加

```ruby
# app/models/group_proposal.rb
class GroupProposal < ApplicationRecord
  # 既存のコード...

  # 期限切れかつ申込されていないGroupProposalを検索するクラスメソッド
  def self.expired_without_application
    where(expires_at: (1.day.ago).all_day, corporate_application_id: nil)
  end

  # 関連するユーザーの個人情報をマスクするメソッド
  def mask_users_personal_information!
    group_proposal_users.each(&:mask_personal_information!)
  end
end
```

- ジョブをシンプルにリファクタリング

```ruby
module GroupProposals
  class MaskGroupProposalUsersJob < BaseJob
    def perform
      ActsAsTenant.with_tenant(ContentPlan.find_by!(slug: ContentPlan::Slugs::HODAI_SLUG)) do
        GroupProposal.expired_without_application.find_each(&:mask_users_personal_information!)
      end
    end
  end
end
```

- メリット
1. **責務の明確化**: 各モデルが自身のデータに関する処理を担当
2. **コードの再利用性**: マスキング処理が他の場所からも呼び出し可能に
3. **テストの容易さ**: モデルのメソッドを単体でテスト可能
4. **可読性の向上**: ジョブがシンプルになり、全体の流れが把握しやすい
- 参考資料

DHHスタイルのRailsパターンについては、以下の記事が参考になります：

[Vanilla Rails is Plenty](https://dev.37signals.com/vanilla-rails-is-plenty/)  
[Put chubby models on a diet with concerns](https://signalvnoise.com/posts/3372-put-chubby-models-on-a-diet-with-concerns)

- テストについて

テストケースは網羅的で、以下のシナリオがカバーされています：

- 期限切れかつcorporate\_application\_idがnilのGroupProposalのケース
- 期限内のGroupProposalのケース
- 申込済みのGroupProposalのケース

また、日付に依存するテストでは `travel_to` を使用して安定性を確保しており、Flaky Testの可能性も低いと思われます。

ビジネスロジックをモデルに移動した場合は、それに合わせてテストも調整する必要があります。

## Devinの指摘の評価

（試験的に）自分も並行でレビューをしており、「ビジネスロジックがジョブにまとまりすぎているので、モデルへ移動すべき」と指摘を入れたい と思っていたのですが、Devinもこの指摘をしており良かったです。ジョブの責務はあくまでモデルのメソッドを呼び出すだけで、モデル内にビジネスロジックを集約するのが望ましい為、狙い通りの性能を出せていることになります。

コードの提案に加えて、設計思想 + Flaky Testといったレベルまで親切に案内できている点も嬉しいです。また、事前にインプットした「推奨記事」を場面に応じて提示している点も素晴らしいです。 **なお、これらの指摘はすべて修正され、人間による最終レビュー時には指摘が無くスムーズにApproveされました。**

## おわりに

この記事では、AIレビュアーとしてのDevin導入の経緯から具体的なチューニング方法、そして得られた効果までを幅広くご紹介しました。

実際に導入してみて感じるのは、AIを活用したコードレビューを取り入れることで、フィードバックを迅速に得られるようになり、人間のレビュー負荷も大幅に軽減されたということです。具体的には、過去に何度も繰り返し指摘してきたようなパターン的な問題をAIが肩代わりしてくれるため、人間がより本質的で難易度の高い課題に集中できるようになりました。また、組織内の他チームがすでに経験した問題や失敗を未然に回避できるようになる点も、大きなメリットだと実感しています。

レビュアー自身も、「どこまでをDevinが見てくれているのか」が明確になるため安心してレビューに臨めますし、AIが事前に不具合や設計の問題をある程度指摘・修正してくれるので、実際のレビュー時には指摘事項が少なくなり、心理的にも楽になりました。

今後運用をする中で、具体的なレビュー時間の推移などの定量的なデータがさらに蓄積されていく予定です。そうしたデータを丁寧に分析し、引き続き具体的な知見や成果について発信できればと思っています。

この記事が、皆さまのチームでのAIレビュー導入の参考になれば幸いです。  
最後までお読みいただき、ありがとうございました！

142

55