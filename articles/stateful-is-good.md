---
title: "Riverpod頑張らなくても大丈夫やで"
emoji: "😌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: true
---

# Flutter を学習し始めた人がぶつかる壁

Menta や Flutter 大学などで Flutter を教える立場になって 4 ヶ月ほど経ちます。
その中で Riverpod を関連の質問を受けることも増えてきました。

ですがそういう人の中にはそもそもなぜ Riverpod を使うといいのかがわかっていない方もいます。

そのような状態に陥ってしまう理由のひとつに、Flutter の学習を進めていると Riverpod を称賛する記事が多く見受けられることがあると思っています。それらの記事が「このブームに乗れなければ置いていかれそうだ」という過度な焦燥感を煽ってしまっているように思うのです。

必要な道具は必要になったときに学ぶことで一番身につくと思いますし、そもそも「なぜ」それを使うのかという部分を見失っては、学ぶ意味もありません。

知的好奇心を満たすためだったり、業務で Riverpod を使用していてそのキャッチアップのために学んでいたりするのなら、とてもいいことだと思います。

中には Riverpod が分からないので、自分は Flutter に向いてないとさえ思ってしまう人もいるようで、それはとても悲しいことです。

Flutter での状態管理は StatefulWidget を使うことができれば達成したいことの 8 割はできてしまうと思います。

Riverpod を学習し始めて途方に暮れ、Flutter を引退しようかと考えてしまっている方は一度考え直してほしいです。作りたいアプリの完成が先、Riverpod は後です。

# StatefulWidget をまずマスターするといいと思う理由

ではまず何から勉強すればいいのでしょうか。わたしは StatefulWidget を使い倒すところからはじめることをお勧めしています。

公式ドキュメントや Flutter に関する記事などに登場するサンプルコードの多くが StatefulWidget で書かれています。

StatefulWidget に慣れ親しめば、それらのコードを難なく読みこなすことができるはずです。

これは Riverpod を習得するよりも価値が高いことだと思います。

# StatefulWidget は怖くないよ

ぼくが Flutter を始めたときは StatefulWidget が怖かったです。
なぜか class を 2 つ作らなければ機能しないし、createState が何をしているかもよくわからないので脳にかかる負荷が大きかったのだと思います。

ですがそんな深いことを考える必要ありません。
内部的な仕組みは慣れてきてから理解するので大丈夫です。

ここで StatefulWidget の初期状態を見てみましょう。
`stf` くらいまで打つと自動的に雛形が生成されるはずです。
あとはそこに Widget 名を付けてあげましょう。
今回は `Sample` と名付けました。

```dart
class Sample extends StatefulWidget {
  const Sample({Key? key}) : super(key: key);


  @override
  _SampleState createState() => _SampleState();
}

class _SampleState extends State<Sample> {
  /// ここに変数を書く

  /// ここに関数を書く

  @override
  Widget build(BuildContext context) {
    /// ここにUIを書く
    return Container();
  }
}
```

初学者の方はまずは難しいことは考えずに `stf` と打って StatefulWidget を作ったら、上記のコメントを頼りに変数と関数 UI を書いていくのがいいと思っています。

慣れてきたらそれぞれがどういう意味合いなのか調べたり考えたりするとよいでしょう。

# StatefulWidget で躓きそうなところ

初学者の方が躓きそうなポイントが 3 つあるので、それらをまとめて解説します。

1. 画面を更新するために `setState((){})` を呼ぶ
2. 最初に一度だけなにかを実行したい場合は `initState()` に書く
3. 受け取った変数は `widget.<変数名>` で使うことができる

## 1. 画面を更新するために `setState((){})` を呼ぶ

何も知らずに Flutter をはじめると、ロジックをちゃんと書いているのになぜか UI が一切変化しないという壁にぶち当たります。
これは初めに生成されるカウンターアプリのコメントに説明が載っているのですが多くの人がスルーしてしまうものでもあると思います。

この問題は `setState((){})` を呼ぶことで解決できます。

特に何も考えずに画面を更新したかったら `setState((){})` と書いておく、それくらいの理解で問題ないです。

// TODO(kenta-wakasa): `setState((){})` の内部構造、リファレンスとなるような記事を見つけておく

## 2. はじめに一度だけ実行したい場合は `initState()` に書く

はじめに一度だけ実行したいことってよくありますよね。例えば何かしらのリストデータ全件をはじめに一度だけ取得したいとかです。

そういった処理は `initState()` に書きます。

さきほどの Sample コードを拡張してみましょう。

```dart
class Sample extends StatefulWidget {
  const Sample({Key? key}) : super(key: key);


  @override
  _SampleState createState() => _SampleState();
}

class _SampleState extends State<Sample> {
  /// ここに変数を書く

  /// ここに関数を書く

  @override
  void initState() {
    super.initState();
    /// ここにはじめに一度だけ実行したいものを書く
  }

  @override
  Widget build(BuildContext context) {
    /// ここにUIを書く
    return Container();
  }
}
```

これをコピペして使えば簡単ですね。

## 3. 受け取った変数は `widget.<変数名>` で使うことができる

外側から変数を受け取って使う場面があると思います。

この場合 state でその変数を使うには `widget.<変数名>` と書く必要があります。

これも Sample コードで見てみましょう。

```dart
class Sample extends StatefulWidget {
  const Sample({Key? key, required this.title}) : super(key: key);

  /// 例えば外側から文字列でtitleを与えるように書く
  final String title

  @override
  _SampleState createState() => _SampleState();
}

class _SampleState extends State<Sample> {
  /// ここに変数を書く

  /// ここに関数を書く

  @override
  void initState() {
    super.initState();
    /// ここにはじめに一度だけ実行したいものを書く
  }

  @override
  Widget build(BuildContext context) {
    /// ここにUIを書く
    return Scaffold(
      appBar: AppBar(
        // Text(title) とかくとだめ
        // Text(widget.title) といける
        title: Text(widget.title),
      )
    );
  }
}
```

# どうしても無理な時はシングルトンという手もある

Stateful を使っていると苦しくなるのは、複数の画面間で状態を共有したいときです。
このとき Riverpod を使っていればグローバル領域に Provider を宣言できるので楽に管理することができます。

ですが Riverpod にはそれなりの学習コストを要すると思うので、まずはシングルトンパターンでその問題を解決するのがいいかなと思っています。

シングルトンというのは、そのプロジェクトの中でインスタンスを一つしか持たないように設計する手法です。

Dart では次のようにシングルトンを簡単に作ることができます。

```dart
class Manager {
  /// ._() のように書くことで、外部ファイルが Manager インスタンスを作成できないようにすることができる。
  Manager._();

  /// 内部の変数に ひとつだけ Manager のインスタンスを生成し代入する。
  static final instance = Manager._();

  /// ここに複数の画面間で使いたい変数群を書いていく
  String userName = 'こんぶ';
  String userId = 'presssedkonbu';
}

/// 上記シングルトンをWidgetの中で使う場合
class Sample extends StatefulWidget {
  const Sample({Key? key}) : super(key: key);

  @override
  _SampleState createState() => _SampleState();
}

class _SampleState extends State<Sample> {
  @override
  Widget build(BuildContext context) {
    /// ここにUIを書く
    return Scaffold(
      appBar: AppBar(
        title: Text(Manager.instance.userName),
      ),
      floatingActionButton: FloatingActionButton(onPressed: () {
        // 値を更新したければ再代入すればいい。
        Manager.instance.userName = '昆布の名前を変更したよ';
        // 画面を更新したいので setState を呼ぶ
        setState((){});
      }),
    );
  }
}
```

このように書いておけば、userName をどの画面からでも参照可能になります。

# おしまいに

Stateful とシングルトンを使えば 9 割くらいの問題には対処できると思っています。
まずはざっくりと理解して、Flutter の楽しさを存分に味わってもらえれば幸いです。
