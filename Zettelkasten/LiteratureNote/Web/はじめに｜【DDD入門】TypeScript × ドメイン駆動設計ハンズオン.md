---
title: "はじめに｜【DDD入門】TypeScript × ドメイン駆動設計ハンズオン"
source: "https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/chapter1_intro"
author:
  - "[[Zenn]]"
published:
created: 2025-05-15
description:
tags:
  - "web"
  - "DDD"
---
このチャプターの目次

1. [この本の目的](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E3%81%93%E3%81%AE%E6%9C%AC%E3%81%AE%E7%9B%AE%E7%9A%84)
2. [この本を読んで欲しい人](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E3%81%93%E3%81%AE%E6%9C%AC%E3%82%92%E8%AA%AD%E3%82%93%E3%81%A7%E6%AC%B2%E3%81%97%E3%81%84%E4%BA%BA)
3. [この本で学ぶこと](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E3%81%93%E3%81%AE%E6%9C%AC%E3%81%A7%E5%AD%A6%E3%81%B6%E3%81%93%E3%81%A8)
4. [この本で学ばないこと](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E3%81%93%E3%81%AE%E6%9C%AC%E3%81%A7%E5%AD%A6%E3%81%B0%E3%81%AA%E3%81%84%E3%81%93%E3%81%A8)
5. [読み方](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E8%AA%AD%E3%81%BF%E6%96%B9)
6. [利用する技術](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E5%88%A9%E7%94%A8%E3%81%99%E3%82%8B%E6%8A%80%E8%A1%93)
7. [参考文献](https://zenn.dev/yamachan0625/books/ddd-hands-on/viewer/#%E5%8F%82%E8%80%83%E6%96%87%E7%8C%AE)

## この本の目的

この本は、TypeScript を使用してドメイン駆動設計（DDD）の原則に基づいた API の構築を学ぶためのガイドです。  
私自身、ドメイン駆動設計 を利用して API を構築する際に、十分な情報やガイドを見つけることに苦労しました。特に、TypeScript で ドメイン駆動設計 を網羅的に説明している資料は少なかったため、さまざまな情報をかき集め、試行錯誤を繰り返してきました。この経験が、本書を執筆する動機の一つとなりました。

私の学びをもとに、複雑なビジネス要求を効果的にソフトウェアに反映する手法を探している開発者の方々へ、実践的な知識とノウハウを共有することを目的としています。

この本ではオンライン書店サービスをドメインとして扱い、その中でも在庫管理に関するサービスを取り上げます。そのドメインを実装するための API を構築する流れを通してドメイン駆動設計の基本的な概念や原則、実践的な実装方法を学びます。ハンズオン形式で進んでは行きますが、辞書のように使っていただくことも可能となっています。

## この本を読んで欲しい人

- ドメイン駆動設計 の初心者や中級者で、基本的な概念や実践的な実装方法を学びたい方。
- ドメイン駆動設計 を学習したが実装への落とし込み方がわからない方。
- TypeScript を用いた開発経験はあるが、ドメイン駆動設計 を組み合わせる方法を知りたい方。
- ビジネス要求を正確にソフトウェアに反映させる方法を探求しているエンジニアやアーキテクト。

## この本で学ぶこと

- ドメイン駆動設計 の基本的な概念と原則の理解
- ビジネスの要求を捉え、それをドメインモデルとして設計する方法
- TypeScript を活用して、ドメインモデルを適切にコードに落とし込む技術
- 集約、エンティティ、値オブジェクト、リポジトリなどの ドメイン駆動設計 のキーコンセプトの実践的な実装

## この本で学ばないこと

- TypeScript の基本的な文法や機能の詳細な解説
- 特定のライブラリやプラグインの使い方・詳細な説明
- ドメイン駆動設計に関連しない一般的なソフトウェア開発のベストプラクティスやテクニック

## 読み方

## 利用する技術

- TypeScript (言語) v5.2.2
- Node.js (実行環境) v18.15.0
- Docker (DB 用コンテナ)
- PostgreSQL (DB)
- Prisma (ORM)
- Express (Web フレームワーク)
- Jest (テストフレームワーク)