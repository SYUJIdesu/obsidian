---
title: OAuth2.0について
source: https://qiita.com/hka9/items/09ba27108a68760f0842
author:
  - "[[hka9]]"
published: 2024-03-19
created: 2025-05-22
description: "OAuth2.0とはOAuth 2.0（Open Authorization 2.0）: Web サービスやアプリケーションの認可や認証を行うためのプロトコル。OAuth を用いることで、ユーザ…"
tags:
  - web
  - 認証
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

この記事は最終更新日から1年以上が経過しています。

[@hka9](https://qiita.com/hka9)

投稿日

## OAuth2.0とは

OAuth 2.0（Open Authorization 2.0）: Web サービスやアプリケーションの認可や認証を行うためのプロトコル。

OAuth を用いることで、ユーザーは自身のユーザーID/パスワードを使う代わりに、アクセストークンを使用してサードパーティーアプリケーションのデータにアクセスすることができます。こうすることで、ユーザーは自身のクレデンシャルをサードパーティーと共有することによるセキュリティリスクを抱えずに済むようになります。

### OAuth 2.0 の主要な概念と役割

- **リソースオーナー** （Resource Owner）: 通常、ユーザーのことを指します。
- **クライアント** （Client）: リソースオーナーの代理としてアクセストークンを取得しリソースサーバーへのアクセスを行います。ウェブアプリケーション、モバイルアプリケーション、デスクトップアプリケーション、API サービスなどを指します。
- **認可サーバー** （Authorization Server）:  
	認可サーバーは、クライアントがアクセストークンを取得するための認可プロセスを処理します。リソースオーナーの承認を受け、アクセストークンを発行します。IdP(Identity Provider)の役割を果たします。
- **リソースサーバー** （Resource Server）:  
	リソースサーバーは、保護されたリソースをユーザーに提供し、アクセストークンを使用してリクエストを検証します。SP（Service Provider）の役割を果たします。

### OAuth2.0のプロトコルフロー

## 認可フローの種類について

- OAuth（Open Authorization）にはいくつかの異なる認可フローがあります。以下に、主要な OAuth 認可フローの種類を説明します

### Authorization Code フロー

- このフローは、Web アプリケーションやモバイルアプリケーションなどのクライアントがリソースオーナー（ユーザー）の承認を受け、アクセストークンを取得するための最も一般的な方法です。
- クライアントは、認可サーバーにリダイレクトされ、ユーザーに認証を要求します。ユーザーが認可を許可すると、認可コードがクライアントに返されます。その後、クライアントはこの認可コードを使用して認可サーバのトークンエンドポイントにアクセスし、アクセストークンを取得します。

### Client Credentials フロー

- このフローは、信頼されたクライアント（たとえば、サーバー上で実行されるバッチジョブや API サービス）が自身の名義で API にアクセスする場合に使用されます。
- クライアントがクレデンシャル（クライアント ID とシークレット）を使用して認可サーバのトークンエンドポイントに直接リクエストを送信し、アクセストークンを取得します。

### Implicit フロー(非推奨)

- このフローは、JavaScript を使用してクライアント側で実行されるアプリケーション（たとえば、SPA）や、モバイルアプリケーションなどの場合に使用されます。
- クライアントがユーザーの認可を取得し、リダイレクト URI にアクセストークンを含むフラグメントを返すように要求します。このフローでは、認可コードは使用されません。
- トークン漏洩リスクが高いため、現在非推奨とされています。(参照： [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-15#section-appendix.a-7.1.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-15#section-appendix.a-7.1.1))

### Resource Owner Password Credentials フロー(非推奨)

- このフローは、信頼されたクライアントがユーザー名とパスワードを使用してアクセストークンを取得する場合に使用されます。
- クライアントがユーザーの資格情報を使用して直接トークンエンドポイントにリクエストを送信し、アクセストークンを取得します。このフローに関してもImplicit フロー同様トークン漏洩リスクが高く、現在非推奨とされています。(参照： [https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-15#section-appendix.a-7.1.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-15#section-appendix.a-7.1.1))

### Refresh Token フロー

- アクセストークンの有効期限切れの場合は、事前に発行を受けていたリフレッシュトークンを認可サーバのトークンエンドポイントに提示することにより、アクセストークンの再発行を受けます。

## Client Typesについて

- クライアントアプリケーションにはクライアントタイプというものがあり、publicもしくは confidentialのどちらかです。
- [https://openid-foundation-japan.github.io/rfc6749.ja.html#client-types](https://openid-foundation-japan.github.io/rfc6749.ja.html#client-types) によると、次のように定義されています。
	- **コンフィデンシャル** (confidential)
		- クレデンシャルの機密性を維持することができるクライアント (例えば, クライアントクレデンシャルへのアクセスが制限されたセキュアサーバー上に実装されたクライアント), または他の手段を使用したセキュアなクライアント認証ができるクライアント.
	- **パブリック** (public)
		- (例えば, インストールされたネイティブアプリケーションやブラウザベースのWebアプリケーションなど, リソースオーナーのデバイス上で実行されるクライアントのように) クライアントクレデンシャルの機密性を維持することができず, かつ他の手段を使用したセキュアなクライアント認証もできないクライアント.

#### Public Clientを用いる時の注意点

- Public Client の場合は認可リクエスト/レスポンスだけでなくクライアントと認可サーバー/リソースサーバー間の通信内容(下図の①,②の部分)も第3者に知られる可能性があるので、Access Tokenの有効期限を短くしたりRefresh Tokenを発行しないなどの考慮をする必要があります。
- もしくは、クライアントにBackend Serverがある場合は、認可サーバー/リソースサーバーとのやりとりを Backend Server に任せることにより、Backend Serverがconfidentialなクライアントとなるので、全体で見てセキュアな実装とすることができます。
- 上記の内容は、以下の記事に詳しく説明してありました。
	- [https://zenn.dev/ritou/articles/d26c7861047a2d](https://zenn.dev/ritou/articles/d26c7861047a2d)

## まとめ

今回は以下の内容についてまとめてみました。

- OAuth2.0の概要
- 認可フローのパターンについて
- Client Typeについて、またそれに関するセキュリティリスクについて

概念の理解はできたので、いつか認証の実装もしてみたいです。

[0](https://qiita.com/hka9/items/#comments)

コメント一覧へ移動

[5](https://qiita.com/hka9/items/09ba27108a68760f0842/likers)

いいねしたユーザー一覧へ移動

3

ストックを更新しました