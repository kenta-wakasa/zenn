---
title: "テトリスを画面に表示する"
---

# さらに課題を分割する

この章ではテトリスを画面に表示することを目標にします。しかしこのままでは解決するべき課題がまだ大きく、何をすればいいか曖昧な部分も多いのでさらに分割して考えてみましょう。

:::message
**WORK : テトリスを画面に表示するにはどうすればいいか掘り下げてみよう**
:::

このとき、一気に実装までの流れが思い浮かべばいいのですが、そんなに簡単なことではないですよね。そういうときは自分にできそうな最も簡単なところからスタートして、とにかく手を動かしてみるというのも一つの方法です。

テトリスには 7 種類のパターンがあります。これらのテトリスは総称してテトリミノと呼ばれていますので、今後はこの文中でもテトリミノと呼ぶことにします。

さらにテトリミノはその形からそれぞれ個別の名前がつけらています。
:::details テトリミノの名称

- T ミノ
- I ミノ
- O ミノ
- Z ミノ
- S ミノ
- J ミノ
- L ミノ
  :::

:::message alert
ここに参考画像を載せる
:::
これらのミノをよく観察してみましょう。
するとこれらはすべて、4 つの正方形の組み合わせで表現されていることに気がつくと思います。

ということはまず、正方形を画面に表示させないことには始まらなさそうです。
これならなんとかできそうですよね。
ひとつ目の課題を **正方形を画面に表示する** としましょう。

# 正方形を画面に表示する

まずは雛形となるコードを作ります。
今回は [カウンターアプリ](https://dartpad.dartlang.org/b6409e10de32b280b8938aa75364fa7b) を雛形として書いていきます。

[カウンターアプリ](https://dartpad.dartlang.org/b6409e10de32b280b8938aa75364fa7b) からできるだけ無駄な部分を削ぎ落とすとこのようになります。

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container() // ここに正方形を表示させるための Widget を書いていく
    );
  }
}
```

それではこのコードを編集して画面に正方形を表示してみましょう。

:::message
**WORK : 画面に正方形を表示させる**
:::

やり方はいろいろあると思いますので、思いついた方法でやってみてください。

# 表示させてみる
@[codepen](https://codepen.io/kenta-wakasa/pen/MWJbgEY)

左上の Flutter ボタンを押すことでソースコードをみることができます。