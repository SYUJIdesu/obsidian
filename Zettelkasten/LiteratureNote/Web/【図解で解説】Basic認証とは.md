---
title: 【図解で解説】Basic認証とは
source: https://zenn.dev/mukkun69n/articles/e697d5b9589345
author:
  - "[[Zenn]]"
published: 2025-01-02
created: 2025-05-15
description: 
tags:
  - web
  - 認証
---
2[tech](https://zenn.dev/tech-or-idea)

## Basic認証とは

**ユーザー名とパスワードを平文で送信して本人である事を確認する認証方式** です。  
下記画像の通り、localhost:3000というurlにアクセスした際に、ブラウザ上でUsernameとPasswordを要求されます。  
この２つを入力してSign inボタンを押下することで、本人であるか確認を行います。  
![](https://storage.googleapis.com/zenn-user-upload/17408a29352f-20250102.png)

## 認証の流れ

![](https://storage.googleapis.com/zenn-user-upload/0321bd0ee17c-20250102.png)

① ブラウザでBasic認証が設定されたURLにアクセスします。

② クライアントからサーバーに対してAuthorizationヘッダー（認証情報）を送信することができます。  
今回の場合は、まだBasic認証ができていないためにAuthorization Headerは空であるため、サーバーは認証情報を確認できません。

③ サーバーはクライアントに認証ができていないことを伝えるために、レスポンスのステータスコードを401に設定します。これは、まだ本人であることを確認できていないことをクライアントに伝えます。  
そして、サーバーがクライアントにどの認証方式を使用するか伝えるために、WWW-Authenticateヘッダーを設定します。今回はBasic認証を行うので、「WWW-Authenticate: Basic realm="some\_realm"」と設定します。  
realmとは、認証が必要な領域を示す文字列です。realm="user\_pages"の場合は、認証の対象がuser\_pagesという名前の領域であることを伝えます。

![](https://storage.googleapis.com/zenn-user-upload/481498a20002-20250102.png)

④ ログイン認証画面が表示されるので、ユーザー名とパスワードを入力します。  
そして、Log inボタンを押します。

⑤ サーバーにユーザー名とパスワードを含めたリクエストを送信します。  
AuthorizationヘッダーにBasicと設定し、Basic認証での認証を行ってくださいと伝えます。  
また、username:passwordの文字列をbase64でエンコードしたものをAuthorizationヘッダーに設定します。

base64とはデータの変換方式の一つです。  
データを64文字の文字（a~z、A～Z、0～9、+、/）と「=」で表現します。  
参考記事： [「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word11338.html)

![](https://storage.googleapis.com/zenn-user-upload/dad1564abddd-20250102.png)

⑥ リクエストのAuthorizationヘッダーをデコードして、元のデータに戻します。  
今回の場合だと、username:passwordの文字列に戻します。

そのusernameとpasswordを元に、認証情報が正しいか確認します。

⑦ 認証情報が正しい場合、ステータスコードが200の成功レスポンスを返します。

引用： [Basic Authentication | Authentication Series](https://www.youtube.com/watch?v=mwccHwUn7Gc)

## Basic認証を体験してみよう

下記はBasic認証のサイトを実際に動かしてみるハンズオンの動画になります。  
時間がある人は、実際にBasic認証を体験してみてください。<iframe src="https://www.youtube-nocookie.com/embed/mwccHwUn7Gc" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

## おまけ （エンコードとデコードについて）

**エンコード** とは、 **データを他の形式へ変換すること** です。  
**デコード** とは、 **エンコードされたデータを元の形式に戻すこと** です。

今回は、認証情報（username:password）をbase64形式でエンコードしてAuthorizationヘッダに設定し、そのリクエストをサーバーに送信していました。  
そもそもなぜエンコードする必要があったのか解説します。

HTTPヘッダーには文字列を含める必要がありますが、特定の特殊文字（コロン：、空白など）はHTTPプロトコルで特別な意味を持ちます。  
認証情報（username:password）をそのまま送信すると、特殊文字によるパースエラーや不具合が発生する可能性があります。  
base64エンコードを使うことで、認証情報を安全に扱える形式にしているそうです。

HTTP Basic認証は、RFC 7617によって定義されています。  
この仕様で、username:passwordをbase64でエンコードすることが規定されています。  
サーバーとクライアントは、この仕様に従ってデータを送受信する必要があります。

参考記事： [「エンコード」と「デコード」の違い](https://wa3.i-3-i.info/diff397data.html)

## 参考にしたもの

<iframe src="https://www.youtube-nocookie.com/embed/mwccHwUn7Gc" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe><iframe src="https://www.youtube-nocookie.com/embed/rhi1eIjSbvk" allow="accelerometer; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen=""></iframe>

2

### Discussion

![](https://static.zenn.studio/images/drawing/discussion.png)

ログインするとコメントできます