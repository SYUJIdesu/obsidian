---
title: "[TS][JS] スプレッド構文とディープコピー"
source: "https://zenn.dev/yusuke_docha/articles/d51c2ca86887e0#%E3%83%87%E3%82%A3%E3%83%BC%E3%83%97%E3%82%B3%E3%83%94%E3%83%BC"
author:
  - "[[Zenn]]"
published: 2022-12-21
created: 2025-05-15
description:
tags:
  - "web"
  - "TS"
  - "スプレッド構文"
---
3[tech](https://zenn.dev/tech-or-idea)

## スプレッド構文のおさらい

スプレッド構文を使うとオブジェクトをコピーすることができます。

```tsx
const object = { name: "tanaka" }

const copyObject = {
  ...object,
}

console.log(copyObject);
// {"name": "tanaka"}
```

## オブジェクトのコピー

オブジェクトをコピーする方法としては下記のような代入が思いつくと思います。

```tsx
const object = { name: "tanaka" };

const copyObject = object;
```

しかしこのコピーではcopyObjectの値を変えてみると、コピー先のobjectの値も変わってしまいます。

```tsx
copyObject.name = "suzuki";

console.log(object);
// {name: suzuki}
```

なぜなら、値そのものはコピー元もコピー先も同じ値を参照しているからです。

## スプレッド構文は別々のコピーを作成する

その点スプレッド構文では代入とは違い、別々のオブジェクトとしてコピーすることができます。

```tsx
const object = { name: "tanaka" };

const copyObject = {
  ...object,
};

copyObject.name = "suzuki";

console.log(copyObject);
// {name: suzuki}
console.log(object);
// {name: tanaka}
```

## スプレッド構文はシャローコピー

ただ、スプレッド構文はネストしたオブジェクトの値の参照先は同じになってしまいます。

スプレッド構文でオブジェクトをコピーすると下記のようになります。

```tsx
const object = {
  id: 1,
  items: [
    {userId: 1, name: "tanaka", age: 20},
    {userId: 2, name: "suzuki", age: 20},
    {userId: 3, name: "satou", age: 20},
  ]
}

const copyObject = {
  ...object,
}
```

このコピーしたcopyObjectのプロパティの値を変更してみると、、

```tsx
copyObject.items[0].userId = 10

console.log(object.items[0])
// {"userId": 10, "name": "tanaka", "age": 20} 
console.log(copyObject.items[0]);
// {"userId": 10, "name": "tanaka", "age": 20}
```

このように、コピー先だけでなくコピー元まで変更されてしまいました。

## シャローコピー

上記のようなスプレッド構文でのコピーをシャローコピーといいます。一見オブジェクトのすべてをコピーしているのように見えますが、 **スプレッド構文はあくまでもプロパティのコピーです** 。

プロパティまではコピー元とは別個のオブジェクトとして存在していますが、その値はコピー元と同じメモリ上の値を参照しています。  
そのため、コピー先を変更するとコピー元まで変更してしまうのです。

## ディープコピー

一方ディープコピーとはその名の通り、ネストした値まで、独立したオブジェクトとしてコピーを作ることができます。

## ディープコピーの方法

ネストしたオブジェクトの値までコピーする、ディープコピーを実現する方法としては、JSON.parse(JSON.stringfiy())を使用する方法。Array.prototype.map()やforEach()などとスプレッド構文を使ってオブジェクトを新たに作る方法などがあります。

今回はmap()とスプレッド構文で新たなオブジェクトを作成してみます。

```tsx
const object = {
  id: 1,
  items: [
    {userId: 1, name: "tanaka", age: 20},
    {userId: 2, name: "suzuki", age: 20},
    {userId: 3, name: "satou", age: 20},
  ]
};

const copyObject = object.items.map((v) => ({...v}));
```

その後、コピーしたcopyObjectの値を変更してもコピー元には影響していないことが分かります。

別のオブジェクトとしてコピーすることができました。

```tsx
copyObject[0].age = 30;

console.log(object);
// {
//   "id": 1,
//   "items": [
//     {
//       "userId": 1,
//       "name": "tanaka",
//       "age": 20
//     },
//     {
//       "userId": 2,
//       "name": "suzuki",
//       "age": 20
//     },
//     {
//       "userId": 3,
//       "name": "satou",
//       "age": 20
//     }
//   ]
// }
console.log(copyObject);
// [{
//   "userId": 1,
//   "name": "tanaka",
//   "age": 30
// }, {
//   "userId": 2,
//   "name": "suzuki",
//   "age": 20
// }, {
//   "userId": 3,
//   "name": "satou",
//   "age": 20
// }]
```

3