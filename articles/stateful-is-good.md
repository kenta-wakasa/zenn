---
title: "Riverpod頑張らなくても大丈夫やで"
emoji: "😌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# Flutter を学習し始めた人がぶつかる壁

Menta や Flutter 大学などで Flutter を教える立場になって 4 ヶ月ほど経ちます。
その中で Riverpod を関連の質問を受けることも増えてきました。

ですがそういう人の中にはそもそもなぜ Riverpod を使うといいのかがわかっていない方もいます。

そのような状態に陥ってしまう理由は一概には言えませんが、Flutter の学習を進めていると Riverpod を称賛する記事が多く見受けられることが一因かなと思います。それらの記事が「このブームに乗れなければ置いていかれそうだ」という過度な焦燥感を煽ってしまっているように思うのです。

必要な道具は必要になったときに学ぶことで一番身につくと思いますし、そもそも「なぜ」それを使うのかという、Why の部分を見失っては、学ぶ意味もありません。

知的好奇心を満たすためだったり、業務で Riverpod を使用していてそのキャッチアップのために学んでいたりするのなら、とてもいいことだと思います。

しかし中には Riverpod が分からないので自分は Flutter に向いてないとさえ思ってしまう人もいるようで、それはとても悲しいことです。

Flutter での状態管理は StatefulWidget を使うことができれば達成したいことの 8 割はできてしまうと思います。

Riverpod を学習し始めて途方に暮れ、Flutter を引退しようかと考えてしまっている方は一度考え直してほしいです。作りたいアプリの完成が先、Riverpod は後です。

# StatefulWidget をまずマスターするといいと思う理由

ではまず何から勉強すればいいのか。わたしは StatefulWidget を使い倒すところからはじめることをお勧めしています。

公式ドキュメントや Flutter に関する記事などに登場するサンプルコードの多くが StatefulWidget で書かれています。

StatefulWidget に慣れ親しめばそれらのコードを難なく読みこなすことができるはずです。

これは Riverpod を習得するよりも価値が高いことだと思います。

# StatefulWidget は怖くないよ

ぼくが Flutter を始めたときは StatefulWidget が怖かったです。
なぜか class を 2 つ作らなければ機能しないし、createState が何をしているかもよくわからないので脳にかかる負荷が大きかったのだと思います。

ですがそんな深いところは考える必要がないのです。
仕組みは慣れてきてから理解するのでいいと思います。

ここで StatefulWidget の初期状態を見てみましょう。
`stfl` とうつと自動的に雛形が生成されるはずです。
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

初学者の方はまずは難しいことは考えずに `stfl` とうって StatefulWidget を作ったら、上記のコメントを頼りに UI と変数と関数を書いていくのがいいと思っています。
慣れてきたらそれぞれがどういう意味合いなのか調べたり考えたりするとよいでしょう。

# StatefulWidget で躓きそうなところ

初学者の方が躓きそうなポイントが 3 つあるので、それらをまとめて解説します。

1. 画面を更新するために `setState((){})` を呼ぶ
2. 最初に一度だけないかを実行したい場合は `initState()` に書く
3. 受け取った変数は widget.<変数名> で使うことができる

## 1. 画面を更新するために `setState((){})` を呼ぶ

# どうしても無理な時はシングルトンという手もある
