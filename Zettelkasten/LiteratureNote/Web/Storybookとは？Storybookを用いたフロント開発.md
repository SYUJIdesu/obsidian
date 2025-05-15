---
title: "Storybookとは？Storybookを用いたフロント開発"
source: "https://zenn.dev/fullyou/articles/853b77a3ce9144"
author:
  - "[[Zenn]]"
published: 2022-12-17
created: 2025-05-15
description:
tags:
  - "web"
  - "Storybook"
---
279

9[tech](https://zenn.dev/tech-or-idea)

## はじめに

半年くらい前にプロジェクトにStorybook及びChromaticを導入したのでその知見を今更まとめたいと思います。Storybookに触れたことがない方・Storybookの導入を検討されている方向けの記事になります。

ChromaticはStorybookを用いた非常に強力なサービスです。Chromaticの記事も書いたので以下をご参照ください。

## Storybookとは何か？

[Storybook](https://storybook.js.org/) は「UIカタログ」です。それぞれのUIコンポーネントをブラウザで手軽にチェックすることができます。React以外にもVueやAngular、Svelteなどにも対応しています。オープンソースで無料のサービス（モジュール）です。

以下のように開発中のコンポーネントをブラウジングできます。

![](https://storage.googleapis.com/zenn-user-upload/da089b3db49d-20221215.png)

Storybookの利点として以下の点が挙げられます

- 手軽にUIのテストができる
- サーバー側の準備ができていなくても先にUIを作ることができる

### 手軽なUIのテスト

PM/PdMの方が仕様書を作った際やデザイナーさんがモックを作った際では想定されなかったパターンでもUIがおかしくならないか予め検証することができます。  
例えば、表示するテキストが長くなったときにレイアウトが崩れないか？、値が `null` の場合にどう表示されるのか？、などのパターンを事前にチェックできます。以下の記事でもこれらを事前にチェックできていることが理想だとされています。

加えてデモデータを用意するだけでこのようなケースを簡単にチェックすることができるので、堅牢なフロントエンド開発が実現できます。（ただ、エッジケースを予め開発者が列挙しなくてはならない、という制限があります。エッジケースは事前に想定しにくいというのがその本質だと思うのでなかなか厳しいですが、「テキストが長くなるとき」「値が `null` のとき」などパターンはあると思うので、運用していく中で磨き上げていくしかないかもしれません）

### サーバー側の準備を待たずUI開発ができる

フロントエンド開発では「データが無いとUI見れないのでサーバー側の開発を待たなくてはいけない」みたいなことがあると思います。  
Storybookでは事前に用意した簡素なデモデータを用いてUIチェックを行うことができるので、サーバー側の開発を待たずにフロント開発ができます。開発サイクルを速くすることができるので、結果的にリファクタなどにも時間を割けれるようになりとても嬉しいです。ただし予めスキーマなどは共有しておく必要があるかと思います。

## Storybookを使った開発

※以降は、React + TypeScriptでの開発を前提とします（他でもだいたい同じだと思います）。

## storyという概念

前述の通りStorybookはUIカタログであり、コーディングしたコンポーネントをブラウザで手軽にチェックできるツールです。  
より正しく言うと、「story単位」でコンポーネントをブラウジング出来ます。storyとは特定のデータを与えたコンポーネントの状態であり１つのコンポーネントに対し複数のstoryが存在し得ます。

例：ボタンがclickableな状態とdisableな状態

![](https://storage.googleapis.com/zenn-user-upload/df21e5b5d6a2-20221215.png)

開発者はUIをチェックするために **interestingなstory** を洗いざらい書く必要があります。interestingとはサービスにおいてそのコンポーネントが重要となりうる、検証する価値のあるコンポーネントの状態、ということを意味しています。上の例で言うと、ボタンはclickableなときとdisableなとき両方のUIとも大事であり、ちゃんとStorybookでチェックする意味のあるinterestingなstoryです。  
他にも「テキストが非常に長くなっているときのときのstory」や「データがnullになっているときにstory」などもinterestingなstoryです。

## インストール方法

[公式Docs](https://storybook.js.org/docs/react/get-started/install) に従ってインストールしていきましょう。

```
$ npx storybook init
```

するとルートディレクトリに`.storybook` ディレクトリ、 `src/stories` ディレクトリとその中にサンプルファイルが自動的に生成され、 `package.json` ファイルもnpmコマンドなどが追記されていると思います。

そして以下のコマンドを叩き、Storybookが立ち上がっていることを確認しましょう。デフォルトではlocalhost:6006にサーバーが立ち上がります。

```
$ npm run storybook
```

## 基本的なstoryの書き方

### ファイルの置き場所

`.storybook/.main.js` ファイルがStorybookの設定ファイルになり、デフォルトで以下のような感じになっています。

```js
module.exports = {
  "stories": [
    "../src/**/*.stories.mdx",
    "../src/**/*.stories.@(js|jsx|ts|tsx)"
  ],
  "addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/addon-interactions"
  ],
  "framework": "@storybook/react"
}
```

"stories"フィールドでは、どのファイルをstoryとして扱うかを記載します。上記の例では `src` ディレクトリ以下にある`.stories.tsx` 拡張子ファイルなどに対して、Storybookはそれをstoryのファイルだと認知してくれるように設定されています。storyのファイルはコンポーネントが書かれたファイルと同階層に配置することが一般的です（コンポーネントをすぐに参照できるため）。

"addons"フィールドにはStorybookで使用するアドオンを記載します。

### 最も典型的なstoryの書き方

storyの書き方はComponent Story Format(CSF)とStoriesOf APIの2通りあります。CSFはStorybook5.2から追加された書き方でこちらが推奨されており、StoriesOf APIは将来的に廃止されるようです。特段理由がない限りはCSFで書きましょう。（Storybookに関してググるとたまにStoriesOf APIで書かれたものが出てきますが、適切に読み替えればCSFの方でも参考になるかもしれません。）

CSFではこんな感じでstoryを書きます。

```tsx
import React from 'react';
import { ComponentMeta, ComponentStory } from '@storybook/react';
import { Button } from './Button';

// ①メタデータの記述
export default {
  title: 'components/atom/Button',
  component: Button,
} as ComponentMeta<typeof Button>;

// ②テンプレートの作成
const Template: ComponentStory<typeof Button> = (args) => (
  <Button {...args} />
);

// ③storyの設定
export const Primary = Template.bind({});
Primary.args = {
  label: 'ボタン',
  backgroundColor: 'light-blue',
};

export const Disable = Template.bind({});
Disable.args = {
  label: 'ボタン',
  backgroundColor: 'gray',
};
```

### ①メタデータの記述

このファイルで記述するstoryのメタデータを設定し、default exportします。

`title` は文字通りstoryのタイトルであり、Storybook上のカタログでタイトルになります。スラッシュ（/）で区切ると階層構造を作ることができ、プロジェクトのディレクトリ階層をそのままタイトルにすることが推奨されています。実際に開発してみても、階層構造に基づかないタイトルに設定するとどのstoryがどのコンポーネントに紐づくか分かりにくくなるので、階層構造をそのまま使うのが無難です。

他には `component` でどのコンポーネントに関するstoryを書くのかを明示的に示したり、 `decorators` や `paramaters` などこのファイルで書くstoryに共通の設定などをすることができます（後述）。

### ②テンプレートの作成

このテンプレート自体は別に必須ではありませが、公式Docsでも使われているように、非常に有用なステップです。テンプレートで予め引数をどのようにコンポーネントに渡すかなどを指定しておくことで、このテンプレートを用いて作成したstoryの記述が楽になります。複数のstoryを作成する際に便利です。

### ③storyの設定

名前付きexportされたstoryがStorybook上で閲覧可能です。このときにつけられた名前がstoryの名前になります。ここではそれぞれのstoryに渡すargs（props）などを設定します。

## 2023.06.24 追記 CSF3でのstoryの書き方

Storybookのversionが6から7に上がり、それに伴いstoryの書き方も変わったので追記します。  
前項の書き方はversion6までのものになるので、7以降を使う場合はこちらを参考にしてください。

```tsx
import React from 'react';
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  component: Button,
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Primary: Story = {
  render: () => <Button primary label="ボタン" backgroundColor="light-blue" />,
};

export const Disable: Story = {
  render: () => <Button primary label="ボタン" backgroundColor="gray" />,
};
```

メタデータを記述するなどの基本的なやり方はこれまでと変わっていません。  
Storybookの開発チーム曰く、CSF3は以前と比べて書きやすくそしてメンテナンスしやすくなっているとのことです。

ちなみに、後方互換性をもつため以前の書き方でもversion7で動きます。

## Configure

以上が基本的なstoryの書き方になりますが、もっと細かく設定することができます。というか既存プロジェクトに導入した場合、細かく設定しないとStorybookがまともに動いてくれないかもしれません。

configについて網羅的に書くのは [公式Docs](https://storybook.js.org/docs/react/configure/overview) に譲るとして、本節では自分が実際に設定したconfigについて書いていきます。

### webpackとのIntegration

CSS Moduleなどを使っている場合はwebpackを使うことが多いと思います。ですので、webpackの設定を適切にStorybookにも適用しないとスタイルがちゃんとあたりません。

webpackをintegrateする際は`.storybook/main.js` に設定を以下のように追加します。

.storybook/main.js

```js
const custom = require('../webpack.config.js');

module.exports = {
  "stories": [
    ...
  ],

  webpackFinal: async(config) => {
    return {
      ...config,
      module: custom.module,
    };
  }
};
```

`webpackFinal` フィールドに設定を追加します。上の例では、元々プロジェクトで使っていた `webpack.config.js` をそのまま引っ張ってきて設定しています。

### 外部スタイルシートやフォントなどの読み込み

普段<head>タグの中で読み込むような外部のスタイルシートなどを使っている場合、そのままではStorybookはそれを認識することができません。外部のスタイルシートやフォントなどを読み込む場合は、`.storybook/preview-head.html` に書きます。

.storybook/preview-head.html

```html
<link rel="stylesheet" href="your/index.css">
<link rel=”preload” href=”your/font” />
```

### 画像などの読み込み

コンポーネントで使用している画像はもちろんStorybookで使うことができます。通常通りpathを指定してimportすれば良いのですが、ロゴやアイコンなどよく使う画像は特定のディレクリにまとめておいていることが多いと思います。その場合は`.storybook/main.js` で `staticDirs` フィールドでそのディレクトリを指定してあげると便利です。

.storybook/main.js

```js
module.exports = {
  stories: [],
  addons: [],
  staticDirs: ['../public'],
};
```

こうすることによって `public/` ディレクトリ以下から自動的に画像等を探してくれます。例えばコンポーネントやstoryで<img src="/logo.png">と書くと、 `public/logo.png` を取ってきてくれます。

ちなみに、以下のようにCLIでstatic directoryを指定することもできますがdepreciatedになっています。

```
start-storybook -p 6006 -s ./public
```

## より発展的なstoryの開発

### Decorators

"Decorator"はstoryをラップするものです。典型的にはstory全体に適用するスタイル、Redux Store、Context ProviderなどをDecoratorとして渡します。

コンポーネントの内部で `useContext` を使用している場合、storyにContext Providerを渡して上げないとコンポーネントがもちろん正しくレンダリングされません。そのためDecoratorとしてProviderを渡します。

```tsx
export const Default = Template.bind({});
Default.args = {
  ...
};
Default.decorators = [
  (Story) => (
    <ThemeContext.Provider value={themes.dark}>
      <Story />
    </ThemeContext.Provider>
  ),
];
```

Context ProviderやRedux Storeの場合、すべてのstoryに適用したいかと思います。そんなときはメタデータの方でDecoratorを設定します。

```tsx
export default {
  title: 'components/atom/Button',
  component: Button,
  decorators: [
    (Story) => (
      <ThemeContext.Provider value={themes.dark}>
        <Story />
      </ThemeContext.Provider>
    ),
  ],
} as ComponentMeta<typeof Button>;
```

### storyにstateをもたせる

上位コンポーネントからその `state` と `setState` を受け取るコンポーネントの場合、シンプルにpropsと関数をstory渡してもうまくいきません。というのも、そのpropsと関数の関係は `state` と `useState` の関係ではないからです。storyにstateをもたせることでこれを解決できます。典型的にはテンプレートファイルでstateを持たせます。

```tsx
const Template: ComponentStory<typeof Paginator> = (args) => {
  const [currentPage, setCurrentPage] = React.useState<number>(
    args.currentPage
  );
  return (
    <Paginator
      {...args}
      currentPage={currentPage}
      changePage={setCurrentPage}
    />
  );
};
```

## Storybookを使ってみて起きた変化

Storybookを使う場合、storyを書きやすいようにコンポーネントを設計する必要があると感じました。例えば、内部でAPIからデータを取得するコンポーネントの場合、APIで取得するデータが変わる度にstoryも変わってしまうので、storyでテストしたいものがテスト出来なくなってしまうことになります。

テストしやすいように書くと自然と **Presentational Component** と **Container Component** の区別ができるようになっていました。それぞれのコンポーネントは以下のように説明されることが多いです。

- **Presentational Component**: Viewのみに焦点が当てられたコンポーネント
	- 具体的なDOM表現を持ち、それぞれにあてるスタイルを知っている
	- 典型的には純粋関数、propsが同じであれば同じUIになる
- **Container Component**: Presentational Componentを内包し、データや挙動などをPresentational Componentに提供するコンポーネント
	- 具体的なDOM表現やスタイルに関する情報を持たない
	- 副作用を持つコンポーネント

Presentational Componentはその説明からわかるように、storyの中心的役割を果たします。そもそもStorybookは「UIのカタログ」です。その目に見えるUIを作っているのはPresentational Componentなので、それらのstoryを書くのは必然と言えるでしょう。  
一方で、Container Componentのようにデータの取得などをするコンポーネントは先述の通りstoryを書くのが非常に辛いです。更にその定義上、Container Componentは「そのコンポーネントだけが持っているUI情報」が無いため、UIカタログに掲載する必要は無いかもしれません。

このような理由で、Storybookを用いると自然とPresentational ComponentとContainer Componentの棲み分けができ、結果的に見通しの良くフロント開発ができるようになりました。

実務上では、Presentational⇔Containerをきれいに分けることが難しい場面も多いかもしれません。しかし、「storyの書きやすさ=Presentaional Componentへの近さ」を意識すると、難しくても「なんとかこの責務をこのコンポーネントに渡せないか」みたいなことを考えるきっかけになります。

storyを起点にコンポーネントを書くことは（特に後からStorybookを導入するといった場合）これまでの開発手法を変えることを意味する可能性があり、その意味でStorybookの導入にはハードルがあるかもしれません。Storybookに対応できるようにコードを書き換えるのは確かに骨の折れる作業でしたが、結果的に良いリファクタリングになったんじゃないかと思っています。

[GitHubで編集を提案](https://github.com/Yuya-Furusawa/zenn-content/blob/main/articles/853b77a3ce9144.md)

279

9