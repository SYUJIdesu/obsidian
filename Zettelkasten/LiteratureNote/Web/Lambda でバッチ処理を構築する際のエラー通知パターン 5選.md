---
title: Lambda でバッチ処理を構築する際のエラー通知パターン 5選
source: https://blog.serverworks.co.jp/lambda-notification-pattern
author:
  - "[[サーバーワークスエンジニアブログ]]"
published: 2021-08-05
created: 2025-05-15
description: はじめに パターン1. 直接Publish パターン メリット デメリット パターン2. DLQパターン メリット デメリット パターン3. 失敗時送信先パターン メリット デメリット パターン4. メトリクスフィルターパターン メリット デメリット パターン5. サブスクリプションフィルターパターン メリット デメリット 各パターン比較表 まとめ はじめに アプリケーションサービス部の宮本です。 お仕事でLambda を使ったバッチ処理を構築することが多いのですが、バッチ処理でエラーが発生した場合、通知が必要なケースが大半です。そこで通知の方法について、幾つかパターンがあるので纏めてみること…
tags:
  - web
  - AWS
---
この記事は約5分で読めます。

この記事は1年以上前に書かれたものです。  
内容が古い可能性がありますのでご注意ください。

## はじめに

アプリケーションサービス部の宮本です。

お仕事でLambda を使ったバッチ処理を構築することが多いのですが、バッチ処理でエラーが発生した場合、通知が必要なケースが大半です。そこで通知の方法について、幾つかパターンがあるので纏めてみることにしました。

イメージとしては以下の様なバッチ処理です。条件に当てはまらない場合は別のアプローチもご検討ください。

**ここでいうバッチ処理のイメージ**

- 実行頻度は1日数回程度以下。
- 失敗すると何かしらの業務がストップするので、確実に通知・リカバリしたい。
- Lambda の呼び出し方法は [非同期呼び出し](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/invocation-async.html)

例えば以下の様な、S3 への Put をトリガーに起動する Lambda をイメージいただければと思います。

![f:id:swx-miyamoto:20210728201319p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728201319.png)

f:id:swx-miyamoto:20210728201319p:plain

また、説明の都合上、通知は SNS へ Publish しメール送信する前提とします。

## パターン1. 直接Publish パターン

最もシンプルなパターンです。当該 Lambdda にて、エラー発生時に SNS の Publish API を呼び出します。

![f:id:swx-miyamoto:20210728202542p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728202542.png)

f:id:swx-miyamoto:20210728202542p:plain

コードのイメージは以下の通りです。 `try ... except` で例外を補足し、例外時は SNS へ Publish します。Lambda 設定の再試行（リトライ）は0回としておき、通知が複数回行われることを防止します。また、最後に raise で例外を Lambda サービスに送出することで、CloudWatch の Errors メトリクスがカウントされます。

```python
import os
 
def handler(event, context):
 
    try:
        # 失敗するかもしれない処理
        do_something()
    except:
        import boto3
        sns = boto3.client('sns')
        import traceback
        
        sns.publish(
            TopicArn=os.environ['TOPIC_ARN'],
            Subject='Something went wrong!',
            Message=traceback.format_exc()
        )
        raise
 
def do_something():
    a = 1 / 0
```

実行すると以下の内容がメール通知されます。

```
[件名] Something went wrong!
[本文]
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 7, in handler
    do_something()
  File "/var/task/lambda_function.py", line 20, in do_something
    a = 1 / 0
ZeroDivisionError: division by zero
```

### メリット

- 実装が分かりやすく、サーバレス入門者でも理解しやすい。
- エラー内容に応じて通知メッセージを柔軟に編集できる。

### デメリット

- Lambda が複数ある場合、エラー通知処理を個別に実装する必要がある。
- 通知が複数回行われることを防止するため、Lambda のリトライ機能を利用出来ない。必要に応じてコード内でリトライする実装が必要。

## パターン2. DLQパターン

2つ目のパターンは [Lambda の DLQ](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/invocation-async.html#dlq) を使ったエラー通知方法です。

![f:id:swx-miyamoto:20210728230153p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728230153.png)

f:id:swx-miyamoto:20210728230153p:plain

DLQ の設定は以下の通りです。デッドレターキューサービスは SNS または SQS が指定できます。

![f:id:swx-miyamoto:20210728205317p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728205317.png)

f:id:swx-miyamoto:20210728205317p:plain

コードのイメージは以下の通りです。パターン1とは違い、 `try ... except` で例外を補足して SNS に Publish せず、 Lambda サービスに送出します。

```python
def handler(event, context):
 
    # 失敗するかもしれない処理
    do_something()
 
def do_something():
    a = 1 / 0 # ZeroDivisionError
```

この場合、Lambda サービスにより先程の DLQ サービスの設定項目である `再試行` の設定値分、リトライが行われます。 リトライ時もエラーが発生した場合、DLQサービスにエラーが送信され、通知が行われます。

```
[件名] AWS Notification Message
[本文]
{}
```

おや、本文に情報がありませんね... DLQサービスへ送信されるのは event 変数の内容です。呼び出し時に以下の様に event データを指定していない為、 `{}` が出力されました。

```sh
aws lambda invoke --function-name notify-pattern-2 \
--invocation-type Event pattern2_response.json
```

人間が読む様なメッセージとするには、SNS の Subscriber として Lambda を指定し、メッセージ内容を整形、再度 Publish する必要があります。

以下の様なイメージです。

![f:id:swx-miyamoto:20210728230153p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728230153.png)

f:id:swx-miyamoto:20210728230153p:plain

### メリット

- バッチ処理 Lambda で SNS への Publish が不要、処理がシンプルになる。

### デメリット

- メッセージを整形したい場合は、別途 Lambda を用意する必要がある。
- 例外の内容は DLQサービスに送信されない為、別途 CloudWatch Logs を参照する必要がある。

## パターン3. 失敗時送信先パターン

3つ目のパターンは [Lambda の 送信先](https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/gettingstarted-features.html#gettingstarted-features-destinations) を使ったエラー通知方法です。

![f:id:swx-miyamoto:20210728235207p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210728/20210728235207.png)

f:id:swx-miyamoto:20210728235207p:plain

送信先の設定は以下の通りです。失敗時の送信先として SNS を指定します。

![f:id:swx-miyamoto:20210729000554p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210729/20210729000554.png)

f:id:swx-miyamoto:20210729000554p:plain

コードのイメージは以下の通りです。パターン2と同様、 `try ... except` で例外を補足して SNS に Publish せず、 Lambda サービスに送出します。

```python
def handler(event, context):
 
    # 失敗するかもしれない処理
    do_something()
 
def do_something():
    a = 1 / 0 # ZeroDivisionError
```

パターン2と同様、設定値分リトライを行った後、以下の内容がメール通知されます。

```
[件名] AWS Notification Message
[本文]
{"version":"1.0","timestamp":"2021-07-28T14:40:37.927Z","requestContext":{"requestId":"baee8a49-2f15-453f-bacc-fe769b392b43","functionArn":"arn:aws:lambda:ap-northeast-1:XXXXXXXXXXXX:function:notify-pattern-3:$LATEST","condition":"RetriesExhausted","approximateInvokeCount":3},"requestPayload":{},"responseContext":{"statusCode":200,"executedVersion":"$LATEST","functionError":"Unhandled"},"responsePayload":{"errorMessage": "division by zero", "errorType": "ZeroDivisionError", "stackTrace": ["  File \"/var/task/lambda_function.py\", line 5, in handler\n    do_something()\n", "  File \"/var/task/lambda_function.py\", line 8, in do_something\n    a = 1 / 0 # ZeroDivisionError\n"]}}
```

メッセージの整形をする場合は、パターン2同様に 整形用の Lambda が必要です。パターン2との違いとしては、例外の内容がメッセージに含まれているところでしょうか。

### メリット

- バッチ処理 Lambda で SNS への Publish が不要、処理がシンプルになる。

### デメリット

- メッセージを整形したい場合は、別途 Lambda を用意する必要がある。

## パターン4. メトリクスフィルターパターン

4つ目のパターンは Lambda のログから [メトリクスフィルター](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/MonitoringPolicyExamples.html) でエラーログを検出し、CloudWatch Alarm 経由で通知を行う方法です。

![f:id:swx-miyamoto:20210729002603p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210729/20210729002603.png)

f:id:swx-miyamoto:20210729002603p:plain

コードのイメージは以下の通りです。 `try ... except` で例外を補足し、ログ出力します。Lambda 設定の再試行（リトライ）は0回としておき、通知が複数回行われることを防止します。

```python
import logging
logger = logging.getLogger(__name__)
 
def handler(event, context):
 
    try:
        # 失敗するかもしれない処理
        do_something()
 
    except:
        logger.exception('[エラー通知] Something went wrong!')
        raise
  
def do_something():
    a = 1 / 0 # ZeroDivisionError
```

ログ出力のイメージは以下の通りです。

```
START RequestId: 3ba3f04d-75af-4997-aafe-8100e8ed08bd Version: $LATEST
[ERROR] 2021-08-04T02:34:48.343Z    3ba3f04d-75af-4997-aafe-8100e8ed08bd    [エラー通知] Something went wrong!
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in handler
    do_something()
  File "/var/task/lambda_function.py", line 16, in do_something
    a = 1 / 0 # ZeroDivisionError
ZeroDivisionError: division by zero
[ERROR] ZeroDivisionError: division by zero
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in handler
    do_something()
  File "/var/task/lambda_function.py", line 16, in do_something
    a = 1 / 0 # ZeroDivisionError
END RequestId: 3ba3f04d-75af-4997-aafe-8100e8ed08bd
REPORT RequestId: 3ba3f04d-75af-4997-aafe-8100e8ed08bd  Duration: 1.72 ms   Billed Duration: 2 ms   Memory Size: 128 MB Max Memory Used: 52 MB
```

このログに対し、メトリクスフィルターを設定します。 フィルターパターンとして、 `"[エラー通知]"` を設定することで、エラーログの出現数をメトリクスとして扱うことが出来ます。

![f:id:swx-miyamoto:20210804113034p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210804/20210804113034.png)

f:id:swx-miyamoto:20210804113034p:plain

![f:id:swx-miyamoto:20210804121239p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210804/20210804121239.png)

f:id:swx-miyamoto:20210804121239p:plain

次にアラームの設定です。先程作成したメトリクスに対するアラーム設定を行います。

![f:id:swx-miyamoto:20210803194908p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210803/20210803194908.png)

f:id:swx-miyamoto:20210803194908p:plain

通知先として SNS を設定します。

![f:id:swx-miyamoto:20210729013533p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210729/20210729013533.png)

f:id:swx-miyamoto:20210729013533p:plain

実行すると以下の内容がメール通知されます。

```
[件名] ALARM: "notify-pattern-4-alarm" in Asia Pacific (Tokyo)
[本文]
You are receiving this email because your Amazon CloudWatch Alarm "notify-pattern-4-alarm" in the Asia Pacific (Tokyo) region has entered the ALARM state, because "Threshold Crossed: 1 out of the last 1 datapoints [1.0 (04/08/21 02:26:00)] was greater than the threshold (0.0) (minimum 1 datapoint for OK -> ALARM transition)." at "Wednesday 04 August, 2021 02:31:45 UTC".

View this alarm in the AWS Management Console:
https://ap-northeast-1.console.aws.amazon.com/cloudwatch/deeplink.js?region=ap-northeast-1#alarmsV2:alarm/notify-pattern-4-alarm

Alarm Details:
- Name:                       notify-pattern-4-alarm
- Description:               
- State Change:               INSUFFICIENT_DATA -> ALARM
- Reason for State Change:    Threshold Crossed: 1 out of the last 1 datapoints [1.0 (04/08/21 02:26:00)] was greater than the threshold (0.0) (minimum 1 datapoint for OK -> ALARM transition).
- Timestamp:                  Wednesday 04 August, 2021 02:31:45 UTC

- AWS Account:                XXXXXXXXXXXX
- Alarm Arn:                  arn:aws:cloudwatch:ap-northeast-1:XXXXXXXXXXXX:alarm:notify-pattern-4-alarm

Threshold:
- The alarm is in the ALARM state when the metric is GreaterThanThreshold 0.0 for 300 seconds.

Monitored Metric:
- MetricNamespace:                     notify-pattern-4-error
- MetricName:                          notify-pattern-4-error
- Dimensions:                         
- Period:                              300 seconds
- Statistic:                           Maximum
- Unit:                                not specified
- TreatMissingData:                    missing

State Change Actions:
- OK:
- ALARM: [arn:aws:sns:ap-northeast-1:XXXXXXXXXXXX:error-notifier]
- INSUFFICIENT_DATA:
```

パターン2、3と同様、例外の内容をメッセージに含めたい場合は加工用 Lambda が必要です。

### メリット

- バッチ処理 Lambda で SNS への Publish が不要、処理がシンプルになる。

### デメリット

- メッセージを整形したい場合は、別途 Lambda を用意する必要がある。
- アラームに設定する期間によるが、短時間に複数回のエラーが発生した場合、通知が一つに纏められてしまうことがある。

## パターン5. サブスクリプションフィルターパターン

最後はLambda のログから [サブスクリプションフィルター](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/SubscriptionFilters.html) でエラーログを検出し、通知用の Lambda を呼び出すパターンです。

![f:id:swx-miyamoto:20210729095913p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210729/20210729095913.png)

f:id:swx-miyamoto:20210729095913p:plain

バッチ処理コードのイメージはパターン4と同じです。 `try ... except` で例外を補足し、ログ出力します。Lambda 設定の再試行（リトライ）は0回としておき、通知が複数回行われることを防止します。

```python
import logging
logger = logging.getLogger(__name__)
 
def handler(event, context):
 
    try:
        # 失敗するかもしれない処理
        do_something()
 
    except:
        logger.exception('[エラー通知] Something went wrong!')
        raise
 
 
def do_something():
    a = 1 / 0 # ZeroDivisionError
```

ログ出力のイメージは以下の通りです。

```
START RequestId: 784f7fdb-0bfa-43f6-a455-100b01ba0399 Version: $LATEST
[ERROR] 2021-08-04T02:51:42.550Z    784f7fdb-0bfa-43f6-a455-100b01ba0399    [エラー通知] Something went wrong!
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in handler
    do_something()
  File "/var/task/lambda_function.py", line 16, in do_something
    a = 1 / 0 # ZeroDivisionError
ZeroDivisionError: division by zero
[ERROR] ZeroDivisionError: division by zero
Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in handler
    do_something()
  File "/var/task/lambda_function.py", line 16, in do_something
    a = 1 / 0 # ZeroDivisionError
END RequestId: 784f7fdb-0bfa-43f6-a455-100b01ba0399
REPORT RequestId: 784f7fdb-0bfa-43f6-a455-100b01ba0399  Duration: 2.04 ms   Billed Duration: 3 ms   Memory Size: 128 MB Max Memory Used: 52 MB  Init Duration: 122.15 ms
```

このログに対し、サブスクリプションフィルターを設定します。 フィルターパターンとして、 `"[エラー通知]"` を設定することで、エラーログを後続の Lambda に渡すことが出来ます。

今回はログを通常のテキスト形式としましたが、json 形式で出力することで、フィルターパターンとして json のキーが指定できます。また、サブスクライバー Lambda の処理でも同様に扱いやすくなる為おすすめです。Python の場合、 [Lambda Powertools Python](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/logger/) を使用すると簡単に json 形式のログが出力できます。

![f:id:swx-miyamoto:20210804114932p:plain](https://cdn-ak.f.st-hatena.com/images/fotolife/s/swx-miyamoto/20210804/20210804114932.png)

f:id:swx-miyamoto:20210804114932p:plain

メッセージ整形・通知 Lambda のコードは以下の様なイメージです。event 変数に エラーログが入っています。ログは圧縮・base64エンコードされている為、デコード・解凍する必要があります。

```python
import base64
import gzip
import json
import boto3
import os
 
sns = boto3.client('sns')
 
def handler(event, context):
    data = event['awslogs']['data']
    # base64デコード
    decoded_data = base64.b64decode(data)
    # 解凍
    decompressed_data = gzip.decompress(decoded_data)
 
    payload = json.loads(decompressed_data)
    for log_event in payload['logEvents']:
        # SNS Publish
        sns.publish(
            TopicArn=os.environ['TOPIC_ARN'],
            Subject='Something went wrong!',
            Message=log_event['message']
        )
```

参考) event 変数のイメージ

```json
{'awslogs': {'data': 'H4sIAAAAAAAAAJVRwWrjMBD9lVltDy3EtSTLkmXoIVBvL7ssJD41DkGWldTUloOsJGRL/71y2kLZPe1NM2/mvXlPL6g346h2pjzvDcrR/bycb34Vy+X8oUAzNJyscaHNUsyFwCnFCQ/tbtg9uOGwD0isTmPcqb5uVGwH327P0V55b5yN0vfJpXdG9WGUYkpiLGIq49XVz3lZLMs1k6LhTJtUyjqIGMWlMXWamIzjVPMmUIyHetSu3ft2sD/aLlCPKF+hv8Wir3PR9jKI1pcLiqOxflp6QW0TDkk4lUwyklAqMpplSXhzTjlnlAQglFlAMGGc8URKwRgmPMkmP74NeXnVB+uEU5FSwlJBaTb7zDHQr4rF4vdiXfnJcIRFRGWJSU5EnojbQPxY+VTrNIjhSGRSR0yabVTXhEZCqoTQTGdb1lR+OfTGP7V2B6dgAE5usLtvlS2d0qZW+hmu+2H04IyeYK26Djo1+pu8sgAhKgMVio/KxV6Nzx+ftNkerJ4iut2fKzSDrrUGshm0Fp6UbTrjpmWAZtiMn/LXN/9LSNIL41eSd1oFd0AgBgzf4dG44b49tmNYLpwbgvI/rRyajxLqM/wJMHpdv74BbvKPKLYCAAA='}}
```

参考) デコード・解凍後の data イメージ

```json
{
    "messageType": "DATA_MESSAGE",
    "owner": "XXXXXXXXXXXX",
    "logGroup": "/aws/lambda/notify-pattern-5",
    "logStream": "2021/08/04/[$LATEST]497d64ce599b450ea69eeb53e8605c6d",
    "subscriptionFilters": ["notify-pattern-5-filter"],
    "logEvents": [
        {
            "id": "36294962588639700789273709401966296954148190780642099201",
            "timestamp": 1627522410822,
            "message": "[ERROR]\t2021-08-04T02:45:27.094Z\t8a75db5e-f9af-46e2-9138-05cd4c19c7b8\t[エラー通知] Something went wrong!\nTraceback (most recent call last):\n  File "/var/task/lambda_function.py", line 8, in handler\n    do_something()\n  File "/var/task/lambda_function.py", line 15, in do_something\n    a = 1 / 0 # ZeroDivisionError\nZeroDivisionError: division by zero"
        }
    ]
}
```

実行すると以下の内容がメール通知されます。

```
[件名] Something went wrong!
[本文]
[ERROR] 2021-08-04T02:45:27.094Z        347197bc-2a8d-4681-bc46-070c15fa69fa    [エラー通知] Something went wrong!

Traceback (most recent call last):
  File "/var/task/lambda_function.py", line 8, in handler
    do_something()
  File "/var/task/lambda_function.py", line 16, in do_something
    a = 1 / 0 # ZeroDivisionError
ZeroDivisionError: division by zero
```

### メリット

- バッチ処理 Lambda で SNS への Publish が不要、処理がシンプルになる。
- メッセージ整形・通知 Lambda でメッセージの整形が可能。

### デメリット

- メッセージ整形・通知 Lambda の実装コストがかかる
- 通知が複数回行われることを防止するため、Lambda のリトライ機能を利用出来ない。必要に応じてコード内でリトライする実装が必要。

## 各パターン比較表

|  | パターン1 | パターン2 | パターン3 | パターン4 | パターン5 |
| --- | --- | --- | --- | --- | --- |
| お手軽度 | ★★★ | ★★☆ | ★★☆ | ★☆☆ | ★☆☆ |
| メッセージ内容の整形 | 可能 | 整形用のLambda を追加すれば可能 | 整形用のLambda を追加すれば可能 | 整形用のLambda を追加すれば可能 | 可能 |
| Lambda のリトライ | 利用不可 | 利用可能 | 利用可能 | 利用不可 | 利用不可 |

## まとめ

5つのパターンをご紹介しました。各パターンのメリット・デメリットを踏まえて要件に合ったエラー通知を実装しましょう。皆様の通知パターンの引き出しに加えていただければ幸いです。

[« AWSのホワイトペーパーから学ぶ AWS Organ…](https://blog.serverworks.co.jp/recommended-ous) [DynamoDB のストレージ容量を自力で取得す… »](https://blog.serverworks.co.jp/get-dynamodb-storage)