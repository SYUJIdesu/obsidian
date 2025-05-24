---
title: Goはオブジェクト指向言語だろうか？ | POSTD
source: https://postd.cc/is-go-object-oriented/
author:
  - "[[POSTD | ニジボックスが運営するエンジニアに向けたキュレーションメディア]]"
published: 
created: 2025-05-15
description: “オブジェクト指向”の意味を本当に理解するには、この概念の始まりを振り返ることが必要です。最初のオブジェクト指向言語はSimulaという言語で、1960年代に登場しました。オブジェクト、クラス、継承…
tags:
  - web
  - golang
---
2015年5月21日

![](https://postd.cc/wp/wp-content/uploads/2014/11/gopher.png)

[Is Go an Object Oriented Language?](http://spf13.com/post/is-go-object-oriented/)

(2014-06-09)by [Steve Francia](http://spf13.com/)

本記事は、原著者の許諾のもとに翻訳・掲載しております。

“オブジェクト指向”の意味を本当に理解するには、この概念の始まりを振り返ることが必要です。最初のオブジェクト指向言語はSimulaという言語で、1960年代に登場しました。オブジェクト、クラス、継承とサブクラス、仮想メソッド、コルーチンやその他多くの概念を導入した言語です。おそらく最も重要なのは、データとロジックが完全に独立したものであるとする、当時では全く新しい考え方をもたらしたことでしょう。

Simula自体には馴染みがない方も多いかもしれませんが、Simulaからインスピレーションを得たとされるJavaやC++、C#、Smalltalkといった言語は皆さんよくご存知でしょう。さらにそこからインスピレーションを得たものとしてObjective-CやPython、Ruby、JavaScript、Scala、PHP、Perlなど様々な言語があり、Simulaは現在使用されているポピュラーな言語のほぼ全てに影響を与えたとも言える言語なのです。Simulaのもたらした新たな考え方はすっかり主力となり、いまやオブジェクト指向以外でコードを書いたことがないプログラマの方が多いほどです。

基準となる定義が存在しないので、議論にあたり私が1つ定義を提唱したいと思います。

オブジェクト指向システムは、コードとデータとしての構造プログラムではなく、”オブジェクト”という概念を用いてこの2つを統合します。オブジェクトは状態（データ）と振る舞い（コード）を持つ抽象データ型なのです。

初期の実装には継承とポリモーフィズムが備わっており、そこから派生した言語も事実上全てその機能を採用しているため、オブジェクト指向プログラミングを定義する際にはこれらの機能を持つことが必要条件とされるのが一般的です。

それでは、Goにおけるオブジェクトやポリモーフィズム、継承について見ていきましょう。そしてこの言語がオブジェクト指向言語であるか否かを考えてみてください。

## Goにおけるオブジェクト

Goには”オブジェクト”と呼ばれるものはありません。しかし”オブジェクト”とは単に意味を表す言葉です。大事なのは言葉自体ではなく、その意味ですね。

Goには”オブジェクト”と呼ばれる型はありませんが、コードと振る舞いの双方を統合するという同じ定義のデータ構造を持っています。それが”構造体”です。

“構造体”とは、名前をつけたフィールドとメソッドを含むある種の型です。

例で示してみましょう。

```prettyprint
type rect struct {
    width int
    height int
}

func (r *rect) area() int {
    return r.width * r.height
}

func main() {
    r := rect{width: 10, height: 5}
    fmt.Println("area: ", r.area())
}
```

上記について語れることはたくさんありますね。ここではコードを1行1行読み解き、ここで何が起こっているかを説明していくことにします。

最初のブロックは”rect”という新しい型を定義しています。これは構造体の型です。この構造体は2つのフィールドを持ち、どちらも整数型です。

次のブロックではこの構造体に関係付けるメソッドを定義しています。関数を定義しrectにアタッチする（結び付ける）ことにより定義が完結します。技術的に言うと、先ほどの例では本当にrectへのポインタにアタッチされています。メソッドは型に関係づけられていますが、Goでは型の値がゼロ値の場合でも（構造体の場合ゼロ値はnil）、呼び出しを行うには型の値を持つことを要求されます。

最後のブロックはmain関数です。1行目がrect型の値を生成します。これ以外にも使える構文はありますが、これが最も慣用的なやり方です。2行目はrectの”r”の面積関数を呼び出した結果を出力するものです。

私には、これはオブジェクトとほぼ同じものに感じられます。構造化されたデータ型を生成し、特定のデータに作用するメソッドを定義することができるのですから。

まだやっていないことがありますね。大部分のオブジェクト指向言語では、オブジェクトを定義するのに”class”のキーワードを使います。継承を用いる時にそれらのクラスのインターフェースを定義するというのは、いいやり方です。それによって、私たちは継承階層木を定義していることになるのです（単一継承の場合）。

さらにお伝えしておきたいのは、Goでは構造体だけではなく、名前をつけた型はどれもメソッドを持つことができるという点です。例えば、整数型にメソッドを定義する新たな型”Counter”を定義することもできます。こちらで例をご覧ください。  
[http://play.golang.org/p/LGB-2j707c](http://play.golang.org/p/LGB-2j707c)

## 継承とポリモーフィズム

オブジェクト間の関係を定義するにはまた違ったアプローチの仕方もあります。やり方は少しずつ違いますが、コードを再利用する仕組みの目的は全てに共通しています。

- 継承
- 多重継承
- 部分型
- オブジェクトコンポジション

## 単一継承と多重継承

継承とは、あるオブジェクトが別のオブジェクトを基にしたものであり、同じ実装を用いている場合を指します。継承の実装には2種類あります。2つの根本的な違いは、単一のオブジェクトから継承できるのか、複数のオブジェクトから継承できるのかという点です。これは一見小さな違いに思えますが、実は大きな意味があるのです。単一継承の階層は木構造であるのに対し、多重継承の階層は格子状です。単一継承言語には、PHPやC#、Java、Rubyなどがあります。多重継承言語には、PerlやPython、C++などがあります。

## 部分型（とポリモーフィズム）

一部の言語では部分型と継承の結びつきが強く、そうした言語を基盤にした特定の見解を持っている方にとっては、前の項があるのにこの項を設けるのは冗長であるように思えるかもしれません。しかし部分型がis-a関係を構築するのに対し、継承は実装を再利用するだけです。部分型は2つ（以上）のオブジェクトのセマンティックな関係を定義します。継承は単に構文の関係を定義するだけです。

## オブジェクトコンポジション

オブジェクトコンポジションは、あるオブジェクトが別のオブジェクトを内包することにより定義されている場合を指します。別のオブジェクトから引き継ぐのではなく、オブジェクトを取り込むのです。部分型のis-a関係とは異なり、コンポジションはhas-a関係を定義します。

## Goにおける継承

Goはあえて継承の機能を持たない設計になっています。しかしオブジェクト（構造体の値）が関係を持たないということではありません。Goの開発者は、継承ではなく、関係を表現する別の仕組みを用いるという選択をしたのです。初めてGoを使う場合などは、このせいでGoが使いにくくなっていると思う人も多いかもしれません。しかし実際には、これはGoの最も優れた特性の1つであり、十数年も前から続いている継承に関する問題や議論を解決するものなのです。

## 継承は除外されているのがベスト

以下に掲載する文章はこの点について、よい問題提起をしています。JavaWorldの記事、 [なぜextendsは有害なのか](http://www.javaworld.com/article/2073649/core-java/why-extends-is-evil.html) から引用します。

> The Gang of Fourの『Design Patterns』（訳注: 日本語版があります。『オブジェクト指向における再利用のためのデザインパターン』）ではページを割いて、実装継承(extends)をインターフェースによる継承(implements)で置き換えるよう説明しています。
> 
> 以前Javaのユーザグループミーティングに出席した際、James Gosling（Javaの生みの親）がメインの講演者として招かれていました。すばらしいQ&Aセッションの途中に、こんな質問が出ました。「もう一度最初からJavaを作り直すとしたら、どこを変更したいですか？」　答えは「クラスを除外するでしょうね」というものでした。笑いが静まった後、彼が説明したのは、本当の問題はクラス自体ではなく実装継承(extendsの関係)なのだということでした。インターフェースによる継承（implementsの関係）のほうが望ましいのです。できる限り実装継承は避けたほうがよいでしょう。

## Goにおけるポリモーフィズムとコンポジション

Go言語は継承の使用を避け、 [継承ではなくコンポジションを選択するという原則](http://en.wikipedia.org/wiki/Composition_over_inheritance) に厳密に従っていま  
す。Goは、これを達成するのに、構造体とインターフェース間の部分型（is-a）とオブジェクトコンポジション（has-a）の関係を利用しています。

## Goにおけるオブジェクトコンポジション

オブジェクトコンポジションの原則を実装するのにGo言語が用いている仕組みは埋め込み型と呼ばれています。Goではhas-a関係を利用して、構造体の中に構造体を埋め込むことを許可しています。

Person（人）とAddress（住所）の関係がいい例になりそうです。

```prettyprint
type Person struct {
   Name string
   Address Address
}

type Address struct {
   Number string
   Street string
   City   string
   State  string
   Zip    string
}

func (p *Person) Talk() {
    fmt.Println("Hi, my name is", p.Name)
}

func (p *Person) Location() {
    fmt.Println("I’m at", p.Address.Number, p.Address.Street, p.Address.City, p.Address.State, p.Address.Zip)
}

func main() {
    p := Person{
        Name: "Steve",
        Address: Address{
            Number: "13",
            Street: "Main",
            City:   "Gotham",
            State:  "NY",
            Zip:    "01313",
        },
    }

    p.Talk()
    p.Location()
}
```

### アウトプット

```prettyprint
>  Hi, my name is Steve
>  I’m at 13 Main Gotham NY 01313
```

[http://play.golang.org/p/LigPIVT2mf](http://play.golang.org/p/LigPIVT2mf)

ここで重要なのは、Addressが個別のエンティティでありながらPerson内に存在しているという点です。main関数で、住所にp.Addressフィールドを設定、またはドット表記を用いてアクセスすることにより、フィールドを設定できるということが分かりましたね。

## Goにおける疑似的部分型

著者注釈:  
当初この記事を投稿した際、Goは匿名フィールドを使ってis-a関係をサポートすると書きましたが、これは間違いでした。実際には、匿名フィールドは、埋め込まれたメソッドとプロパティを外側の構造体に存在するかのように見せることによって、is-a関係のように見せているのです。これは次に述べる理由により、is-a関係であるというには不十分です。Goでは後述するように、インターフェースを利用してis-a関係をサポートしています。この記事の最新版では匿名フィールドを疑似的なis-a関係としています。ある意味においては部分型のように見え、そのように振る舞いますが、実際には違うからです。

疑似的なis-a 関係も、同じように直感的な方法で機能します。上述の例を拡張し、次のステートメントを使ってみましょう。Person（人）は話すことができる。Citizen（国民）はPersonであるから、CitizenはTalk（話すことが）できる。

上述のコード例に追加してみましょう。

```prettyprint
type Citizen struct {
   Country string
   Person
}

func (c *Citizen) Nationality() {
    fmt.Println(c.Name, "is a citizen of", c.Country)
}

func main() {
    c := Citizen{}
    c.Name = "Steve"
    c.Country = "America"
    c.Talk()
    c.Nationality()
}
```

### アウトプット

```prettyprint
>  Hi, my name is Steve
>  Steve is a citizen of America
```

[http://play.golang.org/p/eCEpLkQPR3](http://play.golang.org/p/eCEpLkQPR3)

Go言語の疑似的なis-a関係を、ここでは匿名フィールドと呼ばれるものを利用して構築しています。この例の中では、Citizenの匿名フィールドがPersonです。型だけが与えられ、フィールド名は与えられていません。全てのプロパティとメソッドはPersonのものと想定され、これを自由に利用したり、自身のメソッドに交換することもできます。

### 匿名フィールドのメソッドを交換する

例を挙げましょう。CitizenもPeople（人々）のようにTalkしますが、その方法は異なります。  
そのため、Talkを *Citizen用に定義し、上記で定義したように同じmain関数を実行します。そうすると、* Person.Talk()がコールされる代わりに\*Citizen.Talk()がコールされます。

```prettyprint
func (c *Citizen) Talk() {
    fmt.Println("Hello, my name is", c.Name, "and I'm from", c.Country)
}
```

### アウトプット

```prettyprint
>  Hello, my name is Steve and I'm from America
>  Steve is a citizen of America
```

[http://play.golang.org/p/jafbVPv5H9](http://play.golang.org/p/jafbVPv5H9)

## なぜ匿名フィールドは正式な部分型ではないのか

これが正式な部分型ではないのには、明確な理由が2つあります。

### 1\. 匿名フィールドは埋め込まれているかのようにアクセスできる。

これは必ずしも悪いことではありません。多重継承の問題の1つは、同一のメソッドが1つ以上の親クラスに存在しているときに、言語がどのメソッドを用いているのかが自明ではなく、あいまいだという点です。

Goでは型と同一の名前を持つプロパティを通じて常に単一のメソッドにアクセスすることができます。

実際には、匿名フィールドを使っている時、Go言語はその型と同じ名前でアクセサを作成します。

前述の例を用いると下記のようなコードになります。

```prettyprint
func main() {
    c := Citizen{}
    c.Name = "Steve"
    c.Country = "America"
    c.Talk()         // <- Notice both are accessible
    c.Person.Talk()  // <- Notice both are accessible
    c.Nationality()
}
```

### アウトプット

```prettyprint
>  Hello, my name is Steve and I'm from America
>  Hi, my name is Steve
>  Steve is a citizen of America
```

### 2\. 本当の部分型は親になる。

これが本当に部分型なら、匿名フィールドにより、それを含む型が部分型になるでしょう。Goではそのようにはなりません。2つの型はそれぞれ個別のものであり続けます。下記の例を見ればお分かりいただけると思います。

```prettyprint
package main

type A struct{
}

type B struct {
    A  //B is-a A
}

func save(A) {
    //do something
}

func main() {
    b := B
    save(&b);  //OOOPS! b IS NOT A
}
```

### アウトプット

```prettyprint
>  prog.go:17: cannot use b (type *B) as type A in function argument
>   [process exited with non-zero status]
```

[http://play.golang.org/p/dt1mTXW-BH](http://play.golang.org/p/dt1mTXW-BH)

この例はHacker Newsの [この記事への回答](https://news.ycombinator.com/item?id=7868870) にあったものをそのまま使わせてもらいました。Optymizerに感謝します。

## Goにおける本当の部分型

Goのインターフェースの働きは非常にユニークです。この項では、まっとうに機能しないはずの部分型を、Goがどのようにして関連づけているのかという点についてフォーカスしたいと思います。この記事の最後尾にある「参考サイト」の項もご参照ください。

上述したように、部分型とはis-a関係のことです。Goではそれぞれの型が区別され他の型として振舞うことはありませんが、複数の型が同じインターフェースを利用することができます。インターフェースは関数（とメソッド）のインプットとしてもアウトプットとしても使うことができ、そうすることにより型の間にis-a関係を構築します。

インターフェースを利用するのに、Goでは、”using”などのキーワードを使ってではなく、型の中で宣言された実際のメソッドを使って定義されます。 [Efficient Go](http://golang.org/doc/effective_go.html#interfaces_and_types) の中で、この関係について「使えるメソッドなら使ってOK」とたとえられています。これは非常に重要なことです。なぜなら外部パッケージで定義された型が利用できるインターフェースを作成することができるからです。

引き続き前述の例を見ていきましょう。今度は新たな関数、SpeakToを追加して、main関数をCitizenとPersonにSpeakToするように変更してみましょう。

```prettyprint
func SpeakTo(p *Person) {
    p.Talk()
}

func main() {
    p := Person{Name: "Dave"}
    c := Citizen{Person: Person{Name: "Steve"}, Country: "America"}

    SpeakTo(&p)
    SpeakTo(&c)
}
```

### アウトプット

```prettyprint
>  Running it will result in
>  prog.go:48: cannot use c (type *Citizen) as type *Person in function argument
>  [process exited with non-zero status]
```

[http://play.golang.org/p/lvEjaMQ25D](http://play.golang.org/p/lvEjaMQ25D)

予想通り、失敗していますね。このコードではCitizenはPersonではなく、共通したプロパティはあっても、それぞれに独自な型だとみなされます。

しかしHumanという名前のインターフェースを追加して、SpeakTo関数のインプットとして用いれば、意図した通りに機能します。

```prettyprint
type Human interface {
    Talk()
}

func SpeakTo(h Human) {
    h.Talk()
}

func main() {
    p := Person{Name: "Dave"}
    c := Citizen{Person: Person{Name: "Steve"}, Country: "America"}

    SpeakTo(&p)
    SpeakTo(&c)
}
```

### アウトプット

```prettyprint
>   Hi, my name is Dave
>   Hi, my name is Steve
```

[http://play.golang.org/p/ifcP2mAOnf](http://play.golang.org/p/ifcP2mAOnf)

Goの部分型について2点指摘しておきたいことがあります。

1. 匿名フィールドを使えば一つに限らず多くのインターフェースに対応させることができます。匿名フィールドをインターフェースと共に使うことにより、本来の部分型に非常に近い形になります。
2. Goは適切な部分型の機能を提供しますが、それは型を用いる時だけです。インターフェースは、様々な型が関数へのインプットとして、または関数からの戻り値として受け入れられることを保証するために用いられます。しかし実際には、彼らはそれぞれの型をずっと保持しています。main関数ではCitizenの上にNameを直接セットすることはできないことから、これが分かります。Nameは実際にはCitizenのプロパティではないからです。これはPersonのプロパティであり、ゆえにCitizenの初期化中にはまだ存在していないのです。

## Goはオブジェクトや継承を使わないオブジェクト指向プログラミング言語である

ここまで見てきたように、多少用語は違っていても、オブジェクト指向の基本理念はGoの中に生きています。しかしほとんどのオブジェクト指向言語においては、用いられているメカニズムは異なるので、用語の違いというのは肝心な点なのです。  
Goは構造体をデータとロジックが結合したものとして用います。コンポジションを用いてhas-a関係が構造体の間に構築され、コードの繰り返しを最小化し、それでいて”継承”という壊れやすいやっかいなものは回避するようにしています。Goは不必要で反作用的な宣言をせず、インターフェースを用いてis-a関係を型の間に築きます。  
Goはオブジェクトを持たない新しいオブジェクト指向プログラミングモデルなのです。

## 参考サイト

- [http://nathany.com/good/](http://nathany.com/good/)
- [http://www.artima.com/lejava/articles/designprinciples.html](http://www.artima.com/lejava/articles/designprinciples.html)
- [http://www.goinggo.net/2014/05/methods-interfaces-and-embedded-types.html](http://www.goinggo.net/2014/05/methods-interfaces-and-embedded-types.html)

## 著者について

Steve Franciaは現在 [Dockerプロジェクト](https://www.docker.com/) のチーフオペレータとして2つの大きな商用のオープンソースプロジェクトに従事しています。前職は [MongoDB](http://mongodb.com/) のチーフデベロッパ·アドヴォケイトでした。他にも [spf13-vim](http://vim.spf13.com/) 、 [Hugo](http://gohugo.io/) 、 [Cobra](http://github.com/spf13/cobra) や [Vipe](https://github.com/spf13/viper) など、人気のあるコミュニティベースのオープンソースプロジェクトを運営しています。

彼はオープンソースを愛しており、通常の勤務時間内に楽しく作業するのは  
もちろん、残業もいといません。Steveは@spf13というアカウントで [twitter](http://twitter.com/spf13) 、 [GitHub](http://twitter.com/spf13) )を利用しています。 [spf13.com](http://spf13.com/) でブログをしていて、 [LinkedIn](http://www.linkedin.com/in/stevefrancia) もやっています。 でたまに記事を書き、O’Reillyから何冊か著書も出版しています。

講演やワークショップを行い、世界中のデベロッパコミュニティと時間を過ごすのが好きです。プログラミングをしていない時は、妻と4人の子どもと共にアウトドアで時間を過ごしています。

監修者 ![監修者_古川陽介](https://postd.cc/is-go-object-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)

**古川陽介**  
株式会社リクルート　プロダクト統括本部　プロダクト開発統括室　グループマネジャー 株式会社ニジボックス　デベロップメント室 室長 Node.js 日本ユーザーグループ代表

複合機メーカー、ゲーム会社を経て、2016年に株式会社リクルートテクノロジーズ(現リクルート)入社。 現在はAPソリューショングループのマネジャーとしてアプリ基盤の改善や運用、各種開発支援ツールの開発、またテックリードとしてエンジニアチームの支援や育成までを担う。 2019年より株式会社ニジボックスを兼務し、室長としてエンジニア育成基盤の設計、技術指南も遂行。 Node.js 日本ユーザーグループの代表を務め、Node学園祭などを主宰。
- Twitter:[@yosuke\_furukawa](https://twitter.com/yosuke_furukawa "Twitter")
- Github:[yosuke-furukawa](https://github.com/yosuke-furukawa "Github")