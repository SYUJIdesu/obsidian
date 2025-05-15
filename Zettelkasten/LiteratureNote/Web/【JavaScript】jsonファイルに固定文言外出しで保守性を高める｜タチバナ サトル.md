---
title: "【JavaScript】jsonファイルに固定文言外出しで保守性を高める｜タチバナ サトル"
source: "https://note.com/taro1212/n/n89fff6381f6b"
author:
  - "[[note（ノート）]]"
published: 2021-01-06
created: 2025-05-15
description: "生のJavaScript + jQuery でalertで出す文言やログに出す文言など、固定文言をそのままハードコーディングするのではなく、別ファイルに外出しして保守性を高めようという話です。  もちろんVueやReactなどのフレームワークを使っていれば綺麗に、簡単に実現できますが、生のJavaScriptを使っている場合は綺麗とは言えない形になります。  簡単な話ですが調べてみると意外とたどり着かなかったので、こちらで説明することにしました。  手順は以下の3つ。  1. jsonファイルに外出ししたい文言を記載  2. JavaScriptでJsonファイルを読み込む  3. J"
tags:
  - "web"
  - "Json"
---
## 【JavaScript】jsonファイルに固定文言外出しで保守性を高める

[タチバナ サトル](https://note.com/taro1212)

生のJavaScript + jQuery でalertで出す文言やログに出す文言など、固定文言をそのままハードコーディングするのではなく、別ファイルに外出しして保守性を高めようという話です。

もちろんVueやReactなどのフレームワークを使っていれば綺麗に、簡単に実現できますが、生のJavaScriptを使っている場合は綺麗とは言えない形になります。

簡単な話ですが調べてみると意外とたどり着かなかったので、こちらで説明することにしました。

手順は以下の3つ。

1\. jsonファイルに外出ししたい文言を記載

2\. JavaScriptでJsonファイルを読み込む

3\. JavaScriptでオブジェクト型にして利用する

## 1\. jsonファイルに外出ししたい文言を記載

ファイル名：lang.json

```javascript
{
 "Hello":{
   "ja": "こんにちは",
   "en": "Hello"
 },
 "GoodBye":{
   "ja": "さようなら",
   "en": "GoodBye"
 }
}
```

今回は英語文言と日本語文言があり、切り替えるようなイメージで書いてみました。

## 2\. JavaScriptでJsonファイルを読み込む

ファイル名：index.js

```typescript
let message = {}
$.getJSON("message.json", (data) =>{
  message = data;
})
```

$.getJSON("jsonファイルのパス")

data→jsonファイルのオブジェクトが入ります。

## 3\. JavaScriptでオブジェクト型にして利用する

ファイル名：index.js

```typescript
let message = {}
$.getJSON("message.json", (data) =>{
  message = data;
})

function hello(){
  if(lang=="ja"){
    alert(message.Hello.ja)
  }else if(lang=="en"){
    alert(message.Hello.en)
  }
}
```

特に説明することもないですが、オブジェクト型の要素へのアクセスの仕方は オブジェクト.メンバ名.メンバ名 といった風にアクセスします。

## まとめ：【JavaScriptでjsonファイルを読み込む】

調べてみると上手いこと出てこなかったので、まとめました。

同じことで悩んでいる方の助けになれば幸いです。

それでは。

## いいなと思ったら応援しよう！

- [
	#JavaScript
	](https://note.com/hashtag/JavaScript)
- [
	#jQuery
	](https://note.com/hashtag/jQuery)