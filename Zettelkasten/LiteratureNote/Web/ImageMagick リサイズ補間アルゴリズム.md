---
title: ImageMagick リサイズ補間アルゴリズム
source: https://qiita.com/yoya/items/b1590de289b623f18639
author:
  - "[[yoya]]"
published: 2017-11-28
created: 2025-05-22
description: はじめにImageMagick は画像の拡大や縮小、いわゆるリサイズ処理を制御するオプションが幾つもあります。-resize, -thumbnail, -scale, -sample, -res…
tags:
  - web
  - ライブラリ
---
![](https://relay-dsp.ad-m.asia/dmp/sync/bizmatrix?pid=c3ed207b574cf11376&d=x18o8hduaj&uid=3516551)

## はじめに

ImageMagick は画像の拡大や縮小、いわゆるリサイズ処理を制御するオプションが幾つもあります。

\-resize, -thumbnail, -scale, -sample, -resample, -geometry。  
これらのオプションと、リサイズのちょっとだけ細かい話をします。

## 本エントリで説明しない事

ImageMagick 初心者がはじめに引っかかる、100x100 を指定したのに 100x100 の画像が出来ない問題は別記事にしました。そちらをご参照下さい。

- ImageMagick の Geometry 仕様
	- [https://qiita.com/yoya/items/704fdfbd881f166125d8](https://qiita.com/yoya/items/704fdfbd881f166125d8)

あと、少し高度なリサイズとして -distort Resize があります。時間がかかっても質の良いリサイズが欲しい場合にお勧めです。更に、特殊なリサイズとして、-liquid, -magnify(pixel art scaling), -adaptive-resize(edge detected super sampling) 等もあります。これらはまた機会があれば解説を書くつもりです。

## 画像のリサイズとは？

ここではビットマップ画像の拡大や縮小処理の事とします。  
リサイズ処理の一般的な話は、こちらを参照下さい。

- 画像リサイズ処理のうんちく
	- [https://qiita.com/yoya/items/95c37e02762431b1abf0](https://qiita.com/yoya/items/95c37e02762431b1abf0)

画像処理的にはビットマップのグリッドの仕切り直しと捉える事もできます。 (主流は点光源を波でサンプリングした信号ですが、ここでは置いておく)

- 拡大

| src | resample |  | dst |
| --- | --- | --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3-3x3-dot16.png](https://qiita-image-store.s3.amazonaws.com/0/1798/72943b4a-b21c-b67a-d613-1c012986977b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F72943b4a-b21c-b67a-d613-1c012986977b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7110d5e6cdf991b44a99f251e521ab2a) | [![3x3-3x3-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/a5a927a2-6257-e34b-7357-7ba2d4a7d147.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fa5a927a2-6257-e34b-7357-7ba2d4a7d147.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1a00e50d23911ad8b9507776e5933ada) | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/ae79a269-dbf0-b94a-e341-7fd8dbc97e93.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fae79a269-dbf0-b94a-e341-7fd8dbc97e93.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0d8555406ca48c2c1f9d09f4143d8f14) |

画像の拡大を行うと、元のピクセルの位置から広げて配置し直します。  
この図の空白の部分をどう埋めるのかが、補間(Interpolation)アルゴリズムです。

## 4隅の扱い

上記の拡大イメージはよく見ると、右はじ１列と、下はじ1行が余ってますよね。  
実は、ImageMagick は画像の4隅をあわせるので、これと少し異なります。  
具体的にはこうです。

| src | resample | dst |
| --- | --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1798/2062092c-978a-2dcd-4dcf-03f2195239d5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1798%2F2062092c-978a-2dcd-4dcf-03f2195239d5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=ad586519deaa944dd0b6cc4c3300738c) | [![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1798/580326cc-d007-f4d6-4dfd-2b6128b8c737.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F1798%2F580326cc-d007-f4d6-4dfd-2b6128b8c737.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3b29e05e7d640caf25b88c42e4923e48) |

オフセットを 0.5 ずらす方式です。 以下のように確認できます。

```shell
% echo "P2 2 2 4 1 2 3 4" | convert - -filter triangle -resize 4x4 -compress none -
P2
4 4
255
64 80 112 128
96 112 143 159
159 175 207 223
191 207 239 255
```

なお、6.4.6-5 (2008年11月) でも同じ結果になりました。昔からこれのようです。

## 左上寄せ、右下寄せ

あえて左上に寄せるリサイズは ImageMagick だと distort で再現できます。  
左上(←↑)を基準にするので右下(→↓)に隙間が出来ます。ついでに折角なので逆の右下基準バージョンも作ってみます。

```shell
$ convert 3x3primary.png -filter triangle -set option:distort:viewport 6x6 \
                         -distort SRT "0,0 1.66 0 0,0" left_top-aligned.png
$ convert 3x3primary.png -filter triangle -set option:distort:viewport 6x6 \
                         -distort SRT "0,0 1.66 0 1,1" right_bottom-aligned.png
```

| left\_top-aligned.png | right\_bottom-aligned.png |
| --- | --- |
| [![left_top-aligned-16x16.png](https://qiita-image-store.s3.amazonaws.com/0/1798/efc7704f-010b-1b09-4c21-076700970f2b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fefc7704f-010b-1b09-4c21-076700970f2b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=6294684928722c663dc9170d2e608ba2) | [![right_bottom-aligned-16x16.png](https://qiita-image-store.s3.amazonaws.com/0/1798/3a7a7a91-a3f6-7ab1-9f15-db53d85c77ce.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F3a7a7a91-a3f6-7ab1-9f15-db53d85c77ce.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=db86d343ceeb48699e4f0598841c2281) |
| [![3x3-3x3-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/a5a927a2-6257-e34b-7357-7ba2d4a7d147.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fa5a927a2-6257-e34b-7357-7ba2d4a7d147.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=1a00e50d23911ad8b9507776e5933ada) | [![3x3-3x3-2-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/047dc798-441f-6f14-bfc0-947031845076.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F047dc798-441f-6f14-bfc0-947031845076.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=135fb3c16efb0dc37266d6c384308a84) |

## 補間アルゴリズム

ピクセルを広げた隙間を埋める補間アルゴリズムを ImageMagick はオプションで細かく指定出来ます。  
縮小を行う際にも、どのようにピクセルを削除するか、又は混ぜるかで同様の補間処理があります。

## サンプル画像

比較用サンプルとして、ドット単位のテスト画像と、実際のイラスト画を用います。

```shell
% echo  P3  3 3  9 \
 9 2 3  9 7 1  9 9 0 \
 7 4 7  7 7 4  7 9 0 \
 0 6 9  0 8 6  0 9 0 | convert - 3x3colors.png
```

| 3x3colors.png (+ ドット拡大x16) |
| --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) |

また、水緑様のイラストから髪飾りの画像を使わせて頂きます。

- [https://www.pixiv.net/member\_illust.php?mode=medium&illust\_id=24242462](https://www.pixiv.net/member_illust.php?mode=medium&illust_id=24242462)

[![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d)

## \-sample (最速)

Nearest Neighbor と呼ばれる、単純で速いアルゴリズムで処理します。  
引き延ばす時には隣のピクセルをコピーして増やし、縮小する時には単にピクセルを削除して詰めるだけです。新しい色を作らなくて済むのも大きな特徴です。  
あえてドット絵のような画像を作りたい場合や、GIF や PNG8 のように色数が増えると面倒な時に便利です。

### 拡大

```text
% convert 3x3colors.png -sample 200% 3x3colors-sample200.png
% convert ornament.png  -sample 200% ornament-sample200.png
```

| original | \-sample 200% |
| --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3colors-sample200-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/8255f1b8-ea02-5c6c-0b07-cdebaea0e68b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F8255f1b8-ea02-5c6c-0b07-cdebaea0e68b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=dbc4a2395799dc805d87869497dcd4e1) |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-sample200.png](https://qiita-image-store.s3.amazonaws.com/0/1798/d87db02c-e914-3433-f467-1a67bf9769e6.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fd87db02c-e914-3433-f467-1a67bf9769e6.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=18c4ae9ebdd4f58915d71d3dfea6ed9d) |

斜めの線にドット絵のようなカクカクが付き易いです。よく、ジャギーといった表現を使います。  
また、元の画像にない人工的な歪みが出るのを、アーチファクトと呼びます。医療画像だと誤診を誘発する致命的なやつです。

### 縮小

```shell
% convert 3x3colors.png -sample 50% 3x3colors-sample50.png
% convert ornament.png  -sample 50% ornament-sample50.png
```

| original | \-sample 50% |
| --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3colors-sample50-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/f7c55799-cb72-d9f4-1937-dddd4906883c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Ff7c55799-cb72-d9f4-1937-dddd4906883c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0a27d40c460e775e19043d75573fac50) |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-sample50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/fba6d98e-1625-1b0f-493c-692a90208c48.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Ffba6d98e-1625-1b0f-493c-692a90208c48.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=acf261f8fc0de97814e87c056a55defa) |

こちらも斜めの線がジャギーになり易いです。

## \-scale (次に速い)

### 拡大

```shell
% convert 3x3colors.png -scale 200% 3x3colors-scale200.png
% convert ornament.png  -scale 200% ornament-scale200.png
```

拡大は -sample と同じです。

### 縮小

```shell
% convert 3x3colors.png -scale 50% 3x3colors-scale50.png
% convert ornament.png  -scale 50% ornament-scale50.png
```

Pixel Mixing アルゴリズムを使います。Pixel Averaging, Area map とも呼ばれます。

- Pixel mixing
	- [http://entropymine.com/imageworsener/pixelmixing/](http://entropymine.com/imageworsener/pixelmixing/)
- 画像リサイズのうんちく (補間フィルタ) Pixel Mixing
	- [https://qiita.com/yoya/items/f167b2598fec98679422#pixel-mixing](https://qiita.com/yoya/items/f167b2598fec98679422#pixel-mixing)

縮小前のピクセルは数が多いので、それらの色の平均を使います。中途半端に重なるピクセルはカバー率で重み付けします。

| original | \-sample 50% |
| --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3colors-scale50-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/340187c6-97d3-4a2c-d66f-2fd03acf780d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F340187c6-97d3-4a2c-d66f-2fd03acf780d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=378b2b8a784ab102680fcfaeb8403e2e) |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-scale50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/13d9fc25-417f-20b8-8fac-9eb6a2a0a757.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F13d9fc25-417f-20b8-8fac-9eb6a2a0a757.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3e9f50943727baf4b1d99c8668be60d8) |

曲線が -sample の縮小よりも少し滑らかですが、少しボヤけます。  
ジャギーさも少し残ります。

## \-resize (お勧め)

深く考えずに画像のサイズを変えたいときは、-resize を使うと良いでしょう。

### 拡大

```shell
% convert 3x3colors.png -resize 200% 3x3colors-resize200.png
% convert ornament.png  -resize 200% ornament-resize200.png
```

Bi-Cubic Family の中でバランスの良いと言われる Mitchell フィルタを用います。

| original | \-resize 200% |
| --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3colors-resize200-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/c9f773cb-6245-5cf8-07eb-aab993235205.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fc9f773cb-6245-5cf8-07eb-aab993235205.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9fb23db9dab8fb9493375e20efc16d9a) |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-resize200.png](https://qiita-image-store.s3.amazonaws.com/0/1798/912b51c1-344c-7ec4-aa13-8b4b70fac287.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F912b51c1-344c-7ec4-aa13-8b4b70fac287.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d913f2969b298b23dcae6c4beb75092c) |

滑らかに拡大出来ます。ただし結構ボヤけます。

### 縮小

```shell
% convert 3x3colors.png -resize 50% 3x3colors-resize50.png
% convert ornament.png  -resize 50% ornament-resize50.png
```

Lanzcos フィルタを用います。(ただし、パレット画像 or 透明度がついてる場合には縮小でも Mitchell を適用)  
エイリアシング対策のされたフィルタです。アーチファクトノイズが乗りにくいのが特徴です。

| original | \-sample 50% |
| --- | --- |
| [![3x3colors-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/86962af4-8a9c-83a8-169a-1c854d1be697.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F86962af4-8a9c-83a8-169a-1c854d1be697.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=003c9b0c9076c367b37bfb82d6d4e803) | [![3x3colors-resize50-dot32.png](https://qiita-image-store.s3.amazonaws.com/0/1798/44dac5e0-288d-d2c2-b1bd-56c6eaec1548.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F44dac5e0-288d-d2c2-b1bd-56c6eaec1548.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=f05d235a458a8a41e01642b2864fdfd3) |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-resize50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/1b0c4ebd-2200-a767-d1fd-0106c14da0b5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F1b0c4ebd-2200-a767-d1fd-0106c14da0b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0325d73ff7ac5c85803e9e1c3cbf6bf3) |

滑らかに縮小しますが、こちらも画像がボヤけます。

## \-resize & -unsharp

\-resize のボケ対策には unsharp を併せて使うと良いです。

```shell
% convert ornament.png  -resize 200% -unsharp 10x5+0.7+0  ornament-resize200-unsharp.png
% convert ornament.png  -resize 50% -unsharp 10x5+0.7+0  ornament-resize50-unsharp.png
```

unsharp のパラメータは自分の好みもありますが、ここでは分かりやすいよう少し強めにしてます。2015年当時の GIMP は 12x6+0.5+0 だそうです > [http://freeparticle.hatenablog.com/entry/2015/01/16/230956](http://freeparticle.hatenablog.com/entry/2015/01/16/230956)

### 拡大

| original | \-resize 200% | \-unsharp 10x5+0.5+0 |
| --- | --- | --- |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-resize200.png](https://qiita-image-store.s3.amazonaws.com/0/1798/912b51c1-344c-7ec4-aa13-8b4b70fac287.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F912b51c1-344c-7ec4-aa13-8b4b70fac287.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d913f2969b298b23dcae6c4beb75092c) | [![ornament-resize200-unsharp.png](https://qiita-image-store.s3.amazonaws.com/0/1798/7a6c0d51-5251-22d8-99c6-bf48294ebfcb.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F7a6c0d51-5251-22d8-99c6-bf48294ebfcb.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7f40a7df8151fd51bd6029d93f11f006) |

### 縮小

| original | \-resize 50% | \-unsharp 10x5+0.5+0 |
| --- | --- | --- |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-resize50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/1b0c4ebd-2200-a767-d1fd-0106c14da0b5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F1b0c4ebd-2200-a767-d1fd-0106c14da0b5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0325d73ff7ac5c85803e9e1c3cbf6bf3) | [![ornament-resize50-unsharp.png](https://qiita-image-store.s3.amazonaws.com/0/1798/520f5167-f457-b2a1-dde8-1674099dde1a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F520f5167-f457-b2a1-dde8-1674099dde1a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=58cbca4116a54dcd85895932111b230f) |

### unsharp について

なお、シャープ(sharpness)にするのにオプション名が unsharp なのは、アナログ写真の時代からある技術 unsharp-masking (USM) に由来します。ぼかした上で反転した画像のマスクを作り、それを合成する事でボケを相殺します。unsharp はこれと同じ処理をします。

| [https://en.wikipedia.org/wiki/Unsharp\_masking](https://en.wikipedia.org/wiki/Unsharp_masking) |
| --- |
| [![220px-Unsharp_mask_principle.svg.png](https://qiita-image-store.s3.amazonaws.com/0/1798/8f011869-8e9b-6b5a-11e8-438c5efef35b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F8f011869-8e9b-6b5a-11e8-438c5efef35b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bf688f09427bf4cba13487fc8a603e0a) |

## \-thumbnail (メタデータ削除)

~~画像の変換アルゴリズムは -resize と同じで、更に~~ Exif 等のメタデータを削除します。  
Exif には撮影位置や(画像上書き前の)サムネール画像が入る事もあるので、不特定多数に公開する、又は漏れる可能性のある写真では削った方が身の為です。自宅の位置とか他人に知られたく無いですよね。

### 追記 (2023/02/13)

画像の変換アルゴリズムは以下の重ねがけです。

- 4倍以上縮小する場合は、一旦、目的サイズの4倍の大きさに -sample 相当のアルゴリズム適用
- 2倍以上縮小する場合は、一旦、目的サイズの2倍の大きさに、Box(Nearest-Neighbor) アルゴリズム適用
- 最後に Lanczos 適用。(-resize と違って拡大でも適用される)

拡大に Lanczos は向かないので、縮小に絞って使いましょう。

また、tEXt チャンクに Thumb::〜 シリーズのサムネール補足情報を追加するので、むしろファイルサイズが膨らむ事さえある事に注意が必要です。freedesktop.org の thumbnail specification に合わせた情報なので、関係ない場合は -strip で削除するのが良いでしょう。

- Thumb::URI file:////
	- [https://legacy.imagemagick.org/discourse-server/viewtopic.php?t=15049](https://legacy.imagemagick.org/discourse-server/viewtopic.php?t=15049)
- Thumbnail Managing Standard
	- [https://specifications.freedesktop.org/thumbnail-spec/thumbnail-spec-latest.html](https://specifications.freedesktop.org/thumbnail-spec/thumbnail-spec-latest.html)

指摘ありがとうございます。 > [@shitsyndrome](https://qiita.com/shitsyndrome "shitsyndrome") さん

## \-resample (dpi合わせ)

メタデータの dpi を指定した値で上書きすると同時に、印刷上の画像サイズが dpi 上書き前と変わらないように画像データをリサイズします。

### dpi って？

- [https://ja.wikipedia.org/wiki/Dpi](https://ja.wikipedia.org/wiki/Dpi)

dpi は物理的な解像度で、1 inch に何pixel 詰めるかの単位です。  
例えば、画像をプリンタで印刷するときは dpi に応じて大きさが決まりますし、  
WYSIWYG（ウィジウィグ） 環境ではモニタ表示にも影響します。  
なお、ディスプレイ表示に限定して ppi (pixel per inch) の単位もあります。

### 具体例

既存のディスプレイの多くは 72 dpi 相当ですが、例えば、Retina ディスプレイでは2倍、3倍の解像度を持ちます。  
それを反映して MacBook Pro の Retina ディスプレイでスクリーンショットを撮ると、144 dpi として PNG のメタ情報に記録されます。

```shell
% identify ss.png
ss.png PNG 1064x560 1064x560+0+0 8-bit sRGB 120732B 0.000u 0:00.000
% identify -verbose  ss.png | grep -A 2 Resol
  Resolution: 56.69x56.69
  Print size: 18.7687x9.87829
  Units: PixelsPerCentimeter
```

Resolution が cm 単位なので inch に変換すると、56.69\[px/cm\] \* 2.54 \[cm/inch\] = 144 \[px/inch\]

\-resample オプションを使うと、これを既存モニタ向けの 72dpi 相当に簡単に変換できます。inch 指定、デフォルトの cm指定、 どちらでも良いです。

```shell
% convert ss.png -resample 28.35  ss2.png
% convert ss.png -units PixelsPerInch -resample 72  ss3.png
% identify ss.png ss2.png ss3.png
ss.png PNG 1064x560 1064x560+0+0 8-bit sRGB 120732B 0.000u 0:00.000
ss2.png PNG 532x280 532x280+0+0 8-bit sRGB 43726B 0.000u 0:00.000
ss3.png PNG 532x280 532x280+0+0 8-bit sRGB 43726B 0.000u 0:00.000
```

## \-geometry

\-geometry オプションでも -resize と同じアルゴリズムでリサイズ出来ますが、利用はおすすめ出来ません。

ImageMagick は convert 以外にも色んなコマンドがあって、各々で -geometry が違う意味を持たされている多義的なオプションなのと、歴史的な都合で残してるだけで出来る限り使うのは避けてね。と公式にも注意文言があります。

- [http://www.imagemagick.org/Usage/resize/#geometry](http://www.imagemagick.org/Usage/resize/#geometry)

```text
Geometry is a very special option.
The operator behaves slightly differently in every IM command, and often in special and magical ways.
The reasons for this is mostly due to legacy use and should be avoided if at all possible.
```

## \-filter 付きで -resize

\-filter で任意の補間フィルタを適用できます。

### \-point, -scale, -resize と -filter 〜 の関係

| option | \-filter (enlarge/reduce) |
| --- | --- |
| \-sample | point |
| \-scale | point / pixel mixing(triangleと似てる) |
| \-resize | mitchell / lanczoc |

### 補間フィルタ一覧

\-list filter でフィルタの一覧が出るので、全種類を簡単に試せます。

```shell
for f in \`convert -list filter\` ;
  do convert koishi.png -filter $f -resize 10%  filter/$f.png ;
  done
```

[![スクリーンショット 2018-02-06 0.39.15.png](https://qiita-image-store.s3.amazonaws.com/0/1798/7cce764b-657f-c418-0667-c56631bc3f2a.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F7cce764b-657f-c418-0667-c56631bc3f2a.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=74493bb81d35fd05eb23b17c04ae060a)

これを見るだけでも、縮小には Lanczos が良さそうな感じがします。

## filter:verbose=1

\-define filter:verbose=1 をつけると、実際に適用されるアルゴリズム名や詳細なパラメータ、実際に補間する時の重み付け数列を確認出来ます。

```shell
% convert in.png -define filter:verbose=1 -filter box -resize 100x100 out.png
# Resampling Filter (for graphing)
#
# filter = Box
# window = Box
# support = 0.5
# window-support = 0.5
# scale-blur = 1
# practical-support = 0.5

 0.00    1
 0.01    1
 0.02    1
＜略＞
 0.49    1
 0.50    1
 0.50    0
```

### グラフ

出力された数列は補間する際の重み付けで使う値です。  
アルゴリズム別にグラフにしてみます。

```shell
% # Nearest Neighbor
% convert logo: -define filter:verbose=1 -filter box      -resize 100% null:
% # Bi-Linear
% convert logo: -define filter:verbose=1 -filter triangle -resize 100% null:
% # Bi-Cubic (B:1/3,C:1/3 - Mitchell)
% convert logo: -define filter:verbose=1 -filter mitchell -resize 100% null:
% # Lanczos (Lobe:3)
% convert logo: -define filter:verbose=1 -filter lanczos -resize 100% null:
```

[![Interpolation-Functions.png](https://qiita-image-store.s3.amazonaws.com/0/1798/f7566d68-f479-d3a5-c683-fbf56911c1c4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Ff7566d68-f479-d3a5-c683-fbf56911c1c4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e94d76d2b34ff89d7930272f413dfb18)

- ■ box は 0 か 1 か。ピクセルを混ぜない。
- ■ triangle は線形でピクセルを混ぜる。0 と 1 で傾きが急に変化するのが欠点。
- ■ mitchell は更に一個外のピクセルを参考にして滑らかにつなげる。triangle の欠点を補ってる。
- ■ lanczos は mitchell の更に外のピクセルを考慮できるが、そこは本質でなく sinc で LPF をかける(つまり縮小に強い)。

詳しくはこちらの解説が図付きで分かりやすいです。

- 画素の補間（Nearest neighbor,Bilinear,Bicubic）
	- [http://imagingsolution.blog107.fc2.com/blog-entry-142.html](http://imagingsolution.blog107.fc2.com/blog-entry-142.html)

## \-filter cubic

cubic フィルタを指定すると Bi-Cubic として知られるアルゴリズムで補間します。  
box(Nearest-Neighbor) や triangle(Bi-Linear) は２点の輝度値から計算しますが、Bi-Cubic は更にもう一つ外を含め４点を使う事で滑らかに補間できます。

| [![Bi-Cubic.png](https://qiita-image-store.s3.amazonaws.com/0/1798/f29a47fe-4c61-bfad-ce46-6e7b0ed7d544.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Ff29a47fe-4c61-bfad-ce46-6e7b0ed7d544.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=7635614198fe108c7e2027b4785fc51c) |
| --- |
| 転載元) [https://en.wikipedia.org/wiki/Bicubic\_interpolation](https://en.wikipedia.org/wiki/Bicubic_interpolation) |

Bi-Cubic は B(b-spline) と C(cardinal) のパラメータを 0〜1 の間で調整して色々なフィルタを作り出せます。  
Cubic ファミリーといった呼び方もされ、B,C に応じて以下のようなフィルタが存在します。

| name | B | C |
| --- | --- | --- |
| Hermite | 0 | 0 |
| General | 1 | 0 |
| Catmull-Rom | 0 | 1/2 |
| Mitchell | 1/3 | 1/3 |

対応するソースコードは、MagickCore/resize.c にあります。

[![スクリーンショット 2018-03-20 17.17.20.png](https://qiita-image-store.s3.amazonaws.com/0/1798/6a1cde02-fb9a-d067-e75d-2f13fb960b03.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F6a1cde02-fb9a-d067-e75d-2f13fb960b03.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=0efe25a2efa3511465e06ac9fe8c4476)

B,C の 0,1 を組み合わせた、重み付け数列のグラフです。  
[![図1.png](https://qiita-image-store.s3.amazonaws.com/0/1798/06bfe1fc-6e2f-36d8-2e0c-97e2c8ba9b37.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F06bfe1fc-6e2f-36d8-2e0c-97e2c8ba9b37.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bc7a94ae1fde854b2287c1f68ae592d6)

Bi-Cubic は B:0, C:0 だと Nearest-Neighbor と Bi-Liner の合いの子位の性能で、あまりジャギーを隠せません。そこで B や C の値を上げて調整します。

### B (b-spline)

B:1 のグラフです。  
[![図2.png](https://qiita-image-store.s3.amazonaws.com/0/1798/79fd706e-3ea3-c675-ec2c-f599fd9e6ea0.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F79fd706e-3ea3-c675-ec2c-f599fd9e6ea0.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4c945b2c488f2a01daf64092cdec8bc8)  
青い線が B:1, C:0 に対応します。

```shell
convert ornament.png  -filter cubic \
        -define filter:b=1 \
        -define filter:c=0 \
        -resize 200% ornament-cubic-1-0-200.png
```

[![ornament-cubic-1-0-200.png](https://qiita-image-store.s3.amazonaws.com/0/1798/eb9866ca-31a4-9fcd-1896-40040721d522.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Feb9866ca-31a4-9fcd-1896-40040721d522.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=bd1b911404f4503b97cb509c4cbc9e78)

単純にぼやけてますね。

### C (cardinal)

C:1 のグラフです。  
[![図3.png](https://qiita-image-store.s3.amazonaws.com/0/1798/4c4ee412-2db5-09b5-9d7a-6e2ff1b96659.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F4c4ee412-2db5-09b5-9d7a-6e2ff1b96659.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=09f43b0bd4451072750bb025a997cecd)  
緑の線が B:0, C:1 に対応します。

```shell
convert ornament.png  -filter cubic \
        -define filter:b=0 \
        -define filter:c=1 \
        -resize 200% ornament-cubic-0-1-200.png
```

[![ornament-cubic-0-1-200.png](https://qiita-image-store.s3.amazonaws.com/0/1798/bd1882a3-b204-86a3-06b4-64744dc9c646.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fbd1882a3-b204-86a3-06b4-64744dc9c646.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e6783dff05a8f0a808a1d0045191730b)

こちらは絵がくっきりしますが、本物の輪郭と平行してニセの輪郭が現れています。

[![ornament-cubic-0-1-200のコピー.png](https://qiita-image-store.s3.amazonaws.com/0/1798/d780b081-6729-1d57-0878-82da5472b049.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fd780b081-6729-1d57-0878-82da5472b049.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d6fe2af886a1f27e96a102e16cee5611)

太くて黒い線の隣に元には無かった白い線が追加されてますね。

このように、B と C は単に大きくすれば良い訳ではありません。

### Mitchell

拡大する時に主にフィルタ窓の影響で発生するブロックノイズ(Blocking)を消したいのですが、B を増やすとブラー(Blur)でボヤけ、C を増やすとリンギング(Ringing)が発生し、B,C 両方でエイリアシング(Aliasing)が目立ってくるので、それらのバランスをとって B:1/3, C:1/3 の値を採用したのが Mitchell-Netravali フィルタです。

当てずっぽうに決めたのでなく、実際に主観的テストであらわれた傾向を元に決めています。  
[![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/4cfbe918-c300-e73f-95a3-a23e3b4b4e9d.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F4cfbe918-c300-e73f-95a3-a23e3b4b4e9d.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a28c3086ab20f3a3734965dd959956e5)

- 転載元) [http://www.imagemagick.org/Usage/filter/#mitchell](http://www.imagemagick.org/Usage/filter/#mitchell)
- 論文) [https://www.cs.utexas.edu/~fussell/courses/cs384g-fall2013/lectures/mitchell/Mitchell.pdf](https://www.cs.utexas.edu/~fussell/courses/cs384g-fall2013/lectures/mitchell/Mitchell.pdf)

いい感じに変換出来たパラメータを点線で示していて、そのうち Mitchell と矢印で示されたポイントが B:1/3, C:1/3 です。

### 計算方法

MagickCore/resize.c のコメントで説明されています。

```text
Coefficents are determined from B,C values:
       P0 = (  6 - 2*B       )/6 = coeff[0]
       P1 =         0
       P2 = (-18 +12*B + 6*C )/6 = coeff[1]
       P3 = ( 12 - 9*B - 6*C )/6 = coeff[2]
       Q0 = (      8*B +24*C )/6 = coeff[3]
       Q1 = (    -12*B -48*C )/6 = coeff[4]
       Q2 = (      6*B +30*C )/6 = coeff[5]
       Q3 = (    - 1*B - 6*C )/6 = coeff[6]

    which are used to define the filter:

       P0 + P1*x + P2*x^2 + P3*x^3      0 <= x < 1
       Q0 + Q1*x + Q2*x^2 + Q3*x^3      1 <= x < 2
```

先に、JavaScript で書き直したサンプルを示します。

まず、B,C から8つの係数を算出します。(尚、ImageMagick を真似して上記の式をはじめから 1/6 します)

```js
function cubicBCcoefficient(b, c) {
    var p = 2 - 1.5*b - c;
    var q = -3 + 2*b + c;
    var r = 0;
    var s = 1 - (1/3)*b;
    var t = -(1/6)*b - c;
    var u = b + 5*c;
    var v = -2*b - 8*c;
    var w = (4/3)*b + 4*c;
    return [p, q, r, s, t, u, v, w];
}
```

その8つの係数を3次方程式に当てはめて、Cubic を計算します。

```js
function cubicBC(x, coeff) {
    var [p, q, r, s, t, u, v, w] = coeff;
    var y = 0;
    var ax = Math.abs(x);
    if (ax < 1) {
        //y = p*(ax*ax*ax) + q*(ax*ax) + r*(ax) + s;
        y = ((p*ax + q)*ax + r)*ax + s;
    } else if (ax < 2) {
        //y = t*(ax*ax*ax) + u*(ax*ax) + v*(ax) + w;
        y = ((t*ax + u)*ax + v)*ax + w;
    }
    return y;
}
```

それでは、ImageMagick での実際の処理です。

B,C から8つの係数を算出するのがこちら。

```c
const double
          twoB = B+B;

        /*
          Convert B,C values into Cubic Coefficents. See CubicBC().
        */
        resize_filter->coefficient[0]=1.0-(1.0/3.0)*B;
        resize_filter->coefficient[1]=-3.0+twoB+C;
        resize_filter->coefficient[2]=2.0-1.5*B-C;
        resize_filter->coefficient[3]=(4.0/3.0)*B+4.0*C;
        resize_filter->coefficient[4]=-8.0*C-twoB;
        resize_filter->coefficient[5]=B+5.0*C;
        resize_filter->coefficient[6]=(-1.0/6.0)*B-C;
```

3次方程式の計算はこちら。

```c
if (x < 1.0)
    return(resize_filter->coefficient[0]+x*(x*
      (resize_filter->coefficient[1]+x*resize_filter->coefficient[2])));
  if (x < 2.0)
    return(resize_filter->coefficient[3]+x*(resize_filter->coefficient[4]+x*
      (resize_filter->coefficient[5]+x*resize_filter->coefficient[6])));
  return(0.0);
```

重たいといっても四則演算なので、次に紹介する lanczos ほどでは無いです。

## \-filter lanczos

lanczos フィルタを指定すると、Lanczos(ランチョス、ランツォシュ)窓を使って補間します。  
ImageMagick でフィルタを指定せずに縮小リサイズを動かすと、多くの場合 Lanczos で動作します。(パレット形式だったり透明度が含まれる場合は拡大と同じく mitchell を使います)

ちなみに、Lanczos アルゴリズムと言うと行列変換の方がメジャーっぽいので、区別する為にも、この記事では窓フィルタである事を強調してます。

### Lanczos と Sinc窓

Sinc窓という LPF(ローパスフィルタ)の窓関数があります。  
Lanczos はこの Sinc の LPF 特性を受け継ぐので、(少ないサンプル数でサンプリングし直す)縮小処理に都合がよいです。

!\[image.png\](https://qiita-image-store.s3.amazonaws.com/0/1798/a0d01b24-648c-5779-b03c-b7a4eb418fb3.png) (c) https://en.wikipedia.org/wiki/Sinc\_filter

見ての通り、波が左右方向に永遠に続きます。無限の大きさを持つフィルタなので、畳み込みの計算量が膨大になり、そのままだと扱い辛いです。  
Lancos 窓はこの Sinc窓を更に計算して 2,3,4の範囲で抑えます。

[![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/c40f811f-13f5-43a0-5ba1-ef2c3ab48da4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fc40f811f-13f5-43a0-5ba1-ef2c3ab48da4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=b00be7df1d70f7015e2e42e6e26b3f5d)

グラフを見た方が早いですね。

|  |  |
| --- | --- |
| n=2 | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/6a035f51-85d8-788c-cec4-25e968f7ef02.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F6a035f51-85d8-788c-cec4-25e968f7ef02.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=81617f4c6ea4ed85dfe233e205faff30) |
| n=3 | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/c1bb5c57-8b3c-a4ee-4154-c8eb745d5086.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fc1bb5c57-8b3c-a4ee-4154-c8eb745d5086.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e312ec79d715295fe7837144a05b38b3) |
| n=4 | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/ba88ae9c-df61-b7ed-1d69-384b477f64ed.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fba88ae9c-df61-b7ed-1d69-384b477f64ed.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=d968066601e6bfa47b35d95e7e68ac06) |

このように、n の値で区間を自由に変えられます。  
この窓フィルタを一定区間に収める事。別の言い方だとカーネルを有限の大きさにする事を、コンパクトサポートと表現します。

### \-define filter:lobes

区間 n は -define filter:lobes で指定できます。  
フィルタの上下に出っ張る形を lobe(耳たぶ、または葉)と呼ぶので、それに合わせた命名です。

```shell
convert ornament.png  -filter lanczos \
        -define filter:lobes=2 \
        -resize 50% ornament-lanczos2-50.png
convert ornament.png  -filter lanczos \
        -define filter:lobes=3 \
        -resize 50% ornament-lanczos3-50.png
convert ornament.png  -filter lanczos \
        -define filter:lobes=4 \
        -resize 50% ornament-lanczos4-50.png
```

| original | lobe:2 | lobe:3 | lobe:4 |
| --- | --- | --- | --- |
| [![ornament.png](https://qiita-image-store.s3.amazonaws.com/0/1798/599aae1a-4b4c-9323-28ed-f6c31365ff5b.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F599aae1a-4b4c-9323-28ed-f6c31365ff5b.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=08367f5c0ebcb5b81c759b7035694e5d) | [![ornament-lanczos2-50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/f7337afd-4c11-6c68-b88e-b96556ddf766.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Ff7337afd-4c11-6c68-b88e-b96556ddf766.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=cd576b9b44f7b8e5449ed3a1bc259f1b) | [![ornament-lanczos3-50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/6146f2c1-cad4-a5f5-a300-4e4daf8722a5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F6146f2c1-cad4-a5f5-a300-4e4daf8722a5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=80897a0eb5e531e5fe11b08e86127a00) | [![ornament-lanczos4-50.png](https://qiita-image-store.s3.amazonaws.com/0/1798/8062944f-41e7-e6ed-ee86-924f0cdc3754.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F8062944f-41e7-e6ed-ee86-924f0cdc3754.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=9bea7d0855aa11dbae38d9b91970363f) |

Lanczos2, Lanczos3, Lanczos4 と呼ぶ場合、この n=2,3,4 の Lanczos を指します。  
lobe の n が大きいほど Sinc に近付き、アーチファクトが出にくくなります。が、この例だと違いが分かりませんね。。(後でサンプル探します)

ちなみに、ImageMagick の Lanczos lobe デフォルトは 3 です。(2017年12月の時点で)

### 計算方法

ImageMagick の Lanczos の処理はそれ以外のフィルタも共通化して使えるよう抽象化されたルーチンで動作していて、一見さんには分かりにくいです。先に JavaScript で書き直したサンプルを示します。こんな感じで計算できます。

```js
function sinc(x) {
    var pi_x = Math.PI * x;
    return Math.sin(pi_x) / pi_x;
}
function lanczos(x, lobe) {
    if (x === 0) {
        return 0;
    }
    if (Math.abs(x) < lobe) {
        return sinc(x) * sinc(x/lobe);
    }
    return 0;
}
```

さて、ImageMagick の処理も紹介しておきます。  
まず、Sinc はこちらです。単純に sin(a)/a の計算です。

```c
static double Sinc(const double x,
  const ResizeFilter *magick_unused(resize_filter))
{
  magick_unreferenced(resize_filter);

  /*
    Scaled sinc(x) function using a trig call:
      sinc(x) == sin(pi x)/(pi x).
  */
  if (x != 0.0)
    {
      const double alpha=(double) (MagickPI*x);
      return(sin((double) alpha)/alpha);
    }
  return((double) 1.0);
}
```

Lanczos の lobe 指定はフィルタの support として扱います。

```c
artifact=GetImageArtifact(image,"filter:lobes");
  if (artifact != (const char *) NULL)
    {
      ssize_t
        lobes;

      lobes=(ssize_t) StringToLong(artifact);
      if (lobes < 1)
        lobes=1;
      resize_filter->support=(double) lobes;
    }
```

この support を元に重み付けをします。

```c
contribution[n].weight=GetResizeFilterWeight(resize_filter,scale*
        ((double) (start+n)-bisect+0.5));
```

### SincFast

Sinc 窓は sin 関数を使ったり、割り算をしたりと結構重たいので、この近似として SincFast 関数が用意されています。こちらだと掛け算と足し算だけで済みます。

[![スクリーンショット 2018-07-10 3.17.25.png](https://qiita-image-store.s3.amazonaws.com/0/1798/7ad644d4-19ca-e1d8-47c1-4b4b4af94b36.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F7ad644d4-19ca-e1d8-47c1-4b4b4af94b36.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=e4695d4db14ece5932b36a59b4989256)

ImageMagick の画像縮小で Lanczos フィルタが適用される際、この SincFast の lobes:3 (結果として support:3) 扱いで動作するようです。

## \-colorspace RGB (ガンマ補正の考慮)

一般的なファイル画像の RGB 値は物理的な輝度(明るさ)とリニアな関係にはなく、ガンマ補正のかかった関係にあります。

- ガンマ補正のうんちく
	- [https://qiita.com/yoya/items/122b93970c190068c752](https://qiita.com/yoya/items/122b93970c190068c752)

極端な例だと輝度100%(R,G,B:255,255,255)と0%(R,G,B:0,0,0)が隣り合う時に画像をリサイズしてピクセルを補間した時に算出するピクセルの輝度は以下の暗さになります。

|  |
| --- |
| [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/b4d01075-353c-7bfd-4673-72d9afe9d2d4.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fb4d01075-353c-7bfd-4673-72d9afe9d2d4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=4fc6e09aea2c9c75ac6e604094e35010) |

半分以下ですね。これは極端で稀なケースと思いきや。星空撮影を含め夜間の撮影題材に当てはまるケースが結構あります。

## Resizing with Colorspace Correction

- Resizing with Colorspace Correction
	- [http://www.imagemagick.org/Usage/resize/#resize\_colorspace](http://www.imagemagick.org/Usage/resize/#resize_colorspace)

| sRGB のままリサイズ | gamma:1 にしてリサイズ |
| --- | --- |
| [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/e8ae81ed-b05c-bbc2-3204-b3113cf5daf5.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2Fe8ae81ed-b05c-bbc2-3204-b3113cf5daf5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=beaf1ec2ae0d58326c41c3d22a9ee769) | [![image.png](https://qiita-image-store.s3.amazonaws.com/0/1798/1dfd3ef2-d947-f7c9-1ed0-8b224d1fbf8c.png)](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F1798%2F1dfd3ef2-d947-f7c9-1ed0-8b224d1fbf8c.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a57ba361fb9defd24e191e71dc637a95) |

一度 -colorspace RGB でガンマ補正を解除して、リサイズを行い、改めてまた -colorspace sRGB でガンマ補正をかけ直すと、良い結果が得られます。

```shell
% convert earth_lights_4800.tif -colorspace RGB     -resize 500    \
          -colorspace sRGB  earth_lights_colorspace.png
```

なお、ImageMagick では RGB を輝度リニアな RGB、sRGB をガンマ補正(sRGB相当でなくても gamma:1 以外は全て含む)がかかった RGB として区別するので、それを利用しています。

## Resampling by Nicolas Robidoux

ImageMagick に貢献のある画像処理の研究者 Robidoux さんによる、更に高度なリサイズの話が以下のページにまとまっています。

- Resampling by Nicolas Robidoux
	- [https://www.imagemagick.org/Usage/filter/nicolas/](https://www.imagemagick.org/Usage/filter/nicolas/)
- 拡大

```shell
convert {input} -colorspace RGB +sigmoidal-contrast 7.5 \
        -filter Lanczos -define filter:blur=.9891028367558475 \
        -distort Resize 500% \
        -sigmoidal-contrast 7.5 -colorspace sRGB {output}
```

- 縮小

```shell
convert {input} -colorspace RGB \
        -filter Lanczos -define filter:blur=.9891028367558475 \
        -distort Resize 20%    -colorspace sRGB {output}
```

もう少しだけ細かい解説を書いたので、興味があればこちらもどうぞ。

- [ImageMagick の画像リサイズArtifact対策](https://qiita.com/yoya/items/964cdea0b4d8a86b622b)

## 最後に

フィルタについて語りたい事が山ほどありますが、エントリが更に倍以上に膨らむのでもし需要があれば別エントリで書きます。window や lobe と、それらに絡む計算式はもう少し詳しく解説したいですし、 sample:offset に言及してないのも中途半端な気がしてます。

## 参考

- [http://www.imagemagick.org/Usage/resize/](http://www.imagemagick.org/Usage/resize/)
- [http://www.imagemagick.org/Usage/filter/#filter](http://www.imagemagick.org/Usage/filter/#filter)
- [https://twitter.com/yoya/status/826708839302991873](https://twitter.com/yoya/status/826708839302991873)  
	\-define filter:verbose=1

[1](https://qiita.com/yoya/items/#comments)

[154](https://qiita.com/yoya/items/b1590de289b623f18639/likers)

112

ストックを更新しました