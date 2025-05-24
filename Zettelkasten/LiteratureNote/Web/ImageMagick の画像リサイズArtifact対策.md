---
title: ImageMagick の画像リサイズArtifact対策
source: https://qiita.com/yoya/items/964cdea0b4d8a86b622b
author:
  - "[[yoya]]"
published: 2023-09-10
created: 2025-05-22
description: はじめに一般に画像変換を行うと、元の画像にないノイズのようなものが大なり小なり乗り、その変化を Artifact(アーティファクト、アーチファクト) と呼びます。リサイズ処理では Blur、Bl…
tags:
  - web
  - ライブラリ
  - PHP
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## はじめに

一般に画像変換を行うと、元の画像にないノイズのようなものが大なり小なり乗り、その変化を Artifact(アーティファクト、アーチファクト) と呼びます。  
リサイズ処理では Blur、Blocking、 Ringing、Aliasing、Halo といった形で Artifact が現れ、これらは、どれかを抑えると別のものが目立つトレードオフの関係にあります。

- [https://imagemagick.org/Usage/filter/#artifacts](https://imagemagick.org/Usage/filter/#artifacts)

そこで画像に応じて Artifact を目立たなくする為の調整が必要で、補間フィルタや前処理/後処理等、綺麗な画像を作るのにコツがあります。

あまり知られていませんが、ImageMagick は以下のページにその tips をまとめています。

- [https://imagemagick.org/Usage/filter/nicolas/](https://imagemagick.org/Usage/filter/nicolas/)

このエントリはほぼ、Nicolas Robidoux 特設ページの紹介です。  
だいぶ意訳して省略もしているので、正確な詳細を知りたければ上記のページを直接読んで下さい。

あと、リサイズ補完フィルタを知っている前提で記事を書いています。知らない場合は以下のエントリを参考にどうぞ。

- ImageMagick リサイズ補間アルゴリズム
	- [https://qiita.com/yoya/items/b1590de289b623f18639](https://qiita.com/yoya/items/b1590de289b623f18639)

## 簡易コマンド例

なるべく簡潔なオプションで実行したい場合の例です。

## \-resize

バランスの良い以下のコマンドが使えます。-filter Mitchell は省略可です。

```shell
magick {input} -filter Mitchell -resize 200% {output}
```

Mitchell は BiCubic の一種で、主観テストによりバランスの良いパラメータを探った結果のフィルタです。

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/4cfbe918-c300-e73f-95a3-a23e3b4b4e9d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F4cfbe918-c300-e73f-95a3-a23e3b4b4e9d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a28c3086ab20f3a3734965dd959956e5)

- 転載元) [http://www.imagemagick.org/Usage/filter/#mitchell](http://www.imagemagick.org/Usage/filter/#mitchell)
- 論文) [https://www.cs.utexas.edu/~fussell/courses/cs384g-fall2013/lectures/mitchell/Mitchell.pdf](https://www.cs.utexas.edu/~fussell/courses/cs384g-fall2013/lectures/mitchell/Mitchell.pdf)

ぼやけてしまう場合、Lanczos に変更する事で画像をよりシャープに出来ます。ただしハロー効果(Halo effect)が出やすくなります。

```shell
magick {input} -filter Lanczos -resize 200% {output}
```

これらは HDRI 版 ImageMagick では問題ないですが、Q8, Q16 版だと各々、0〜255, 0〜65535 に clamp されて、画質が微妙に劣化します。また、sinc フィルタの限界もあります。

それでも処理がそこそこ速いので、多くのサービスでのリサイズで -resize が使われています。

## \-distort Resize

\-resize より処理が重たいですが、画質を求めるならば -distort Resize がお勧めです。

\-resize の sinc が -distort Resize では jinc 相当に置き換わり、EWA(Elliptical Weighted Average) として動作します。いわゆる Cylindrical Filter です。  
要するに縦横と同等に斜めも綺麗にリサイズします。

\-distort Resize はデフォルトで -filter Robidoux が適用されます。

```shell
magick {input} -distort Resize 200% {output}
```

これでだいたいバランスの良い結果になります。

ボケる場合、以下のコマンドでよりシャープな画像にする手もあります。

```shell
magick {input} -filter LanczosRadius -distort Resize 200% {output}
```

シャープにすると副作用でハロー効果（halo effect)を生じます。これを抑えるのに良いのが、　EWA Quadratic-windowed Jinc 3-lobe です。

```shell
magick {input} -define filter:window=Quadratic -distort Resize 200% {output}
```

また、Quadratic B-spline でもハローを消せます。ただし画像がぼやけがちです。

```shell
magick {input} -filter Quadratic -distort Resize 200% {output}
```

このように、さまざまな Artifacts (今回は主にブラーとハローのせめぎ合い)のトレードオフに応じてオプションを使い分けると良い結果を得られます。

## 推奨コマンド

ノウハウを詰め込んだ推奨コマンドです。オプションがそこそこ複雑です。

## 拡大

- [https://imagemagick.org/Usage/filter/nicolas/#upsampling](https://imagemagick.org/Usage/filter/nicolas/#upsampling)

```shell
magick {input} -colorspace RGB +sigmoidal-contrast 7.5 \
        -filter Lanczos -define filter:blur=.9891028367558475 \
        -distort Resize 500% \
        -sigmoidal-contrast 7.5 -colorspace sRGB {output}
```

まず、素の RGB は sRGB 規格に従いガンマ補正されているので、それを元に計算すると値が歪みます。傾向として暗くなります。それで、 -colorspace RGB でリニアRGB に変換して、リサイズ処理した後、-colorspace sRGB で元に戻します。

- Resizing with Colorspace Correction
	- [http://www.imagemagick.org/Usage/resize/#resize\_colorspace](http://www.imagemagick.org/Usage/resize/#resize_colorspace)

| sRGB のままリサイズ | gamma:1 にしてリサイズ |
| --- | --- |
| [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/e8ae81ed-b05c-bbc2-3204-b3113cf5daf5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fe8ae81ed-b05c-bbc2-3204-b3113cf5daf5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=beaf1ec2ae0d58326c41c3d22a9ee769) | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/1dfd3ef2-d947-f7c9-1ed0-8b224d1fbf8c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F1dfd3ef2-d947-f7c9-1ed0-8b224d1fbf8c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a57ba361fb9defd24e191e71dc637a95) |

あと、リサイズをすると画像がボケがちなので、見た目の印象を変えない為には、Lanczos かつ filter:blur に 1未満の値を渡す事で画像をシャープ化します。

更に、シャープ化するとハロが出やすいので、シグモイド変換でハローを抑えます。

## 縮小

- [https://imagemagick.org/Usage/filter/nicolas/#downsample](https://imagemagick.org/Usage/filter/nicolas/#downsample)

```shell
magick {input} -colorspace RGB \
        -filter Lanczos -define filter:blur=.9891028367558475 \
        -distort Resize 20%    -colorspace sRGB {output}
```

縮小はハロー対策のシグモイド変換を外した事以外は、同じです。

## 参考

- [https://imagemagick.org/Usage/filter/](https://imagemagick.org/Usage/filter/)
- [https://imagemagick.org/Usage/filter/#jinc](https://imagemagick.org/Usage/filter/#jinc)
- [https://imagemagick.org/Usage/filter/nicolas/](https://imagemagick.org/Usage/filter/nicolas/)
- ImageMagick リサイズ補間アルゴリズム
	- [https://qiita.com/yoya/items/b1590de289b623f18639](https://qiita.com/yoya/items/b1590de289b623f18639)
- 画像リサイズ処理のうんちく
	- [https://qiita.com/yoya/items/95c37e02762431b1abf0](https://qiita.com/yoya/items/95c37e02762431b1abf0)

[0](https://qiita.com/yoya/items/#comments)

[0](https://qiita.com/yoya/items/964cdea0b4d8a86b622b/likers)

1

ストックを更新しました