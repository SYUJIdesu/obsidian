---
title: （Java）DTOクラスとは？どう実装すれば良いのか？
source: https://zenn.dev/fujishiro/scraps/d58591dc9a2bc1
author:
  - "[[Zenn]]"
published: 2023/11/07にコメント追加
created: 2025-05-15
description: ふじしろさんのスクラップ
tags:
  - web
  - オブジェクト指向
---
## DTOクラスとは？

DTO（Data Transfer Object）は、  
エンティティの中から必要な値だけを取得したクラス。

DBに重要な情報が保存されている場合、取得したエンティティをそのままユーザーに全て渡すと、それらの情報まで見られてしまう。  
かといって、状況に応じてエンティティからいちいち削除するような処理は非常に手間だし、実装ミスの原因になる。さらに、渡すデータに変更が加わった際に、全て手作業で確認することになる。  
そこで出てくるのがDTO。  
DTOというクラスを作成し、あらかじめ必要なデータだけをエンティティから取得するようにしておけば、そのDTOをユーザーに渡すだけで済む。  
そして保守も簡単になる。

参考になった記事

- プロジェクトにおけるDTOの位置付け
	- Spring Frameworkで作るWeb三層アプリケーション - ぺんぎんらぼ [https://penguinlabo.hatenablog.com/entry/spring/web-layered](https://penguinlabo.hatenablog.com/entry/spring/web-layered)
- DTOの存在意義について
	- ☆ [SpringBoot/DTO - KobeSpiral2021](https://cs27.org/wiki/kobespiral2021/?SpringBoot/DTO)
	- [【Java】頼むからDTOをしっかりと実装して欲しいお話 | mokabuu.com](https://mokabuu.com/it/java/%E3%80%90java%E3%80%91%E9%A0%BC%E3%82%80%E3%81%8B%E3%82%89dto%E3%82%92%E3%81%97%E3%81%A3%E3%81%8B%E3%82%8A%E3%81%A8%E5%AE%9F%E8%A3%85%E3%81%97%E3%81%A6%E6%AC%B2%E3%81%97%E3%81%84%E3%81%8A%E8%A9%B1)
	- [Spring Boot API で DTO をエンティティへ自動的にマッピングする](https://auth0.com/blog/jp-automatically-mapping-dto-to-entity-on-spring-boot-apis/)

ログインするとコメントできます