---
title: LambdaでS3に上がったMP4をHLSに変換する (動画サイト構築までの道②)
source: https://zenn.dev/jinwatanabe/articles/44f0906c6f317b#s3-%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B
author:
  - "[[Zenn]]"
published: 2022-11-20
created: 2025-05-15
description: 
tags:
  - web
  - AWS
---
5[tech](https://zenn.dev/tech-or-idea)

## はじめに

このシリーズでは React を利用して、ストリーミング動画サイトを構築するために必要な動画周りの配信について解説していくシリーズになります

今回は Lambda を利用して S3 に保存された `MP4` を自動で `HLS` に変換する仕組みを作っていきます。

色々記事は世の中にありどれも同じような仕組みでやっていますが、Lambda 上で実行するコードの設定がわかりづらいと感じたので丁寧に説明できればと思います。

## 過去のシリーズ

- [CloudFront と S3 で HLS の動画配信を行う (動画サイト構築までの道 ①)](https://zenn.dev/jinwatanabe/articles/3d84675c06ee43)

## 環境

VSCode  
Ubuntu 20.04 (WSL2)  
Docker 20.10.12  
docker-compose version v2.2.3  
git version 2.25.1

## S3 を作成する

AWS コンソールを開いて「S3」と検索して開きます

![](https://storage.googleapis.com/zenn-user-upload/a0d094630887-20221120.png)

「バケットを作成」をクリック

![](https://storage.googleapis.com/zenn-user-upload/1abde77b1555-20221120.png)

バケット名に「hls-handson」と入力して「バケットを作成」をクリック  
※S3 のバケット名は各自でユニークなものをつけて以降読み替えてください

![](https://storage.googleapis.com/zenn-user-upload/2c1794fb2cd8-20221120.png)

作成した「hls-handson」をクリック

![](https://storage.googleapis.com/zenn-user-upload/bee3aa84adf6-20221120.png)

「フォルダを作成」をクリック

![](https://storage.googleapis.com/zenn-user-upload/9c560fb577fe-20221120.png)

フォルダ名に「input」と入力して「フォルダの作成」をクリック

![](https://storage.googleapis.com/zenn-user-upload/4268c6316cfa-20221120.png)

同じ要領で「output」というフォルダを作成します  
hls-handson に input と output のフォルダができていれば大丈夫です

![](https://storage.googleapis.com/zenn-user-upload/2984fb26810f-20221120.png)

## AWS Elemental MediaConvert の設定

AWS コンソールから「AWS Elemental MediaConvert」を検索して開きます

![](https://storage.googleapis.com/zenn-user-upload/b8c640a2377e-20221120.png)

左メニューから「ジョブテンプレート」をクリック

![](https://storage.googleapis.com/zenn-user-upload/648ba231c457-20221120.png)

「テンプレートの作成」をクリック

![](https://storage.googleapis.com/zenn-user-upload/6f5b26340a69-20221120.png)

「名前」に「mp4ToHLS」を入力

![](https://storage.googleapis.com/zenn-user-upload/05561ee2a8eb-20221120.png)

左メニューから「入力」の横にある「追加」をクリック  
ここで何か設定する必要はありません

左メニューから「出力」の横にある「追加」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/b4544e240024-20221120.png)

「Apple HLS」を選択して「選択」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/4a78fb5cfd1c-20221120.png)

「送信先」の「参照」から「作成したバケット名/input/」を選択します

![](https://storage.googleapis.com/zenn-user-upload/c5e67f0f25c9-20221120.png)

左メニューから「H264, AAC」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/d305cf5b615f-20221120.png)

「名前修飾子」に「\_hls」と入力します

![](https://storage.googleapis.com/zenn-user-upload/9b19d5f2b87c-20221120.png)

「最大ビットレート(ビット/秒)」に「5000000」と入力します  
![](https://storage.googleapis.com/zenn-user-upload/08e0cf841aff-20221120.png)

最後に「作成」をクリックします

作成できました  
![](https://storage.googleapis.com/zenn-user-upload/f3d8477aea2d-20221120.png)

## Lambda の設定

今回は Python を利用して Lambda 関数を作ります  
関数の中では `/input` に `MP4` がアップロードされたら起動して、 `HLS` に変換します

AWS コンソールから「lambda」と検索してひらきます

![](https://storage.googleapis.com/zenn-user-upload/244161fd3d70-20221120.png)

「関数の作成」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/8fb03b5da217-20221120.png)

| 項目 | 値 |
| --- | --- |
| 関数名 | hlsConvert |
| ランタイム | Python3.9 |
| 実行ロール | AWS ポリシーテンプレートから新しいロールを作成 |
| ロール名 | hls-handson-lambda-role |
| ポリシーテンプレート - オプション | Amazon S3 オブジェクトの読み込み専用アクセス権 |

![](https://storage.googleapis.com/zenn-user-upload/4403c704f202-20221120.png)

![](https://storage.googleapis.com/zenn-user-upload/ecf0dc255c0c-20221120.png)

「関数の作成」をクリック

「トリガーを追加」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/9d9eb02daa13-20221120.png)

| 項目 | 値 |
| --- | --- |
| トリガー | S3 を選択 |
| Bucket | 作成したバケット |
| Prefix | input/ |
| Suffix | .mp4 |
| Recursive invocation | ☑ |

![](https://storage.googleapis.com/zenn-user-upload/1f1d4c2cb47f-20221120.png)

「追加」をクリック

次に関数を登録します。タブから「コード」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/78fb5cc71908-20221120.png)

以下のコードをエディタに貼り付けます

```
import os
import urllib.parse
import boto3

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')

    settings = make_settings(bucket, key)
    user_metadata = {
        'JobCreatedBy': 'videoConvertSample',
    }

    client = boto3.client('mediaconvert', endpoint_url = "[MediaConvertのAPIエンドポイント]")
    result = client.create_job(
        Role = "arn:aws:iam::[アカウントID]:role/service-role/MediaConvert_Default_Role",
        JobTemplate = "[MediaConvertのテンプレート名: 今回はmp4ToHLS]",
        Settings=settings,
        UserMetadata=user_metadata,
    )

def make_settings(bucket, key):
    basename = os.path.basename(key).split('.')[0]

    return \
    {
        "Inputs": [
            {
                "FileInput": f"s3://{bucket}/{key}",
            }
        ],
        "OutputGroups": [
            {
                "Name": "Apple HLS",
                "OutputGroupSettings": {
                    "Type": "HLS_GROUP_SETTINGS",
                    "HlsGroupSettings": {
                        "Destination": f"s3://{bucket}/output/",
                    },
                },
                "Outputs": [
                    {
                        "VideoDescription": {
                            "Width": 640,
                            "Height": 360,
                        },
                    },
                ],
            },
        ],
    }
```

コードの 3 箇所を修正します。

1. MediaConvert の API エンドポイントは MediaConvert の左メニューから「アカウント」をクリックすると表示されます

![](https://storage.googleapis.com/zenn-user-upload/59db01a11d78-20221120.png)

1. アカウント ID にはそれぞれのアカウント ID をいれます。注意としてはここで用いるサービスロールは前回のハンズオンで作成したものになります。作成していない場合は MediaConvert のジョブの作成を 1 度行って、「AWS の統合」を設定することで作成できます

![](https://storage.googleapis.com/zenn-user-upload/dc945aa31b8e-20221120.png)

1. MediaConvert で作成したテンプレート名をいれます

私の場合以下のようなコードになりました

```
import os
import urllib.parse
import boto3

def lambda_handler(event, context):
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = urllib.parse.unquote_plus(event['Records'][0]['s3']['object']['key'], encoding='utf-8')

    settings = make_settings(bucket, key)
    user_metadata = {
        'JobCreatedBy': 'videoConvertSample',
    }

    client = boto3.client('mediaconvert', endpoint_url = "https://xxxxxx.mediaconvert.ap-northeast-1.amazonaws.com")
    result = client.create_job(
        Role = "arn:aws:iam::xxxxxxxxxx:role/service-role/MediaConvert_Default_Role",
        JobTemplate = "mp4ToHLS",
        Settings=settings,
        UserMetadata=user_metadata,
    )

def make_settings(bucket, key):
    basename = os.path.basename(key).split('.')[0]

    return \
    {
        "Inputs": [
            {
                "FileInput": f"s3://{bucket}/{key}",
            }
        ],
        "OutputGroups": [
            {
                "Name": "Apple HLS",
                "OutputGroupSettings": {
                    "Type": "HLS_GROUP_SETTINGS",
                    "HlsGroupSettings": {
                        "Destination": f"s3://{bucket}/output/",
                    },
                },
                "Outputs": [
                    {
                        "VideoDescription": {
                            "Width": 640,
                            "Height": 360,
                        },
                    },
                ],
            },
        ],
    }
```

次に変更を Lambda に反映させるため「Deploy」をクリック

次にタブの「設定」をクリックして左メニューから「アクセス権限」をクリックして「実行ロール」の「ロール名」をクリックします

![](https://storage.googleapis.com/zenn-user-upload/15aa003a0a81-20221120.png)

「許可を追加」→「ポリシーをアタッチ」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/618d3dc06fab-20221120.png)

「IAMFullAccess」, 「AWSElementalMediaConvertFullAccess」にチェックをいれて「ポリシーをアタッチ」をクリック

## 試してみる

「S3」を開いて作成したバケットの「input」ディレクトリを開きます  
「アップロード」をクリック  
![](https://storage.googleapis.com/zenn-user-upload/2d08d12f0406-20221120.png)

「ファイル追加」から適当な`.mp4` のファイルを選択します  
![](https://storage.googleapis.com/zenn-user-upload/14e2741cd854-20221120.png)

「アップロード」をクリック

そのあと 30 秒ほど待ってから「output」ディレクトリを開きます

![](https://storage.googleapis.com/zenn-user-upload/a133efffdd49-20221120.png)

S3 にアップロードされた `mp4` が自動で `hls` に変換されました

## おわりに

これである程度動画サイトを作るための用意が終わりました  
次回は最後に React を使って簡単な動画アップロードのフォームと仕組みを作成してこのシリーズも終了したいと思います。

## 参考

[GitHubで編集を提案](https://github.com/jinwatanabe/zenn_article/blob/master/articles/44f0906c6f317b.md)

5