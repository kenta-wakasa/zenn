---
title: "StatefulWidgetをつかいたい"
---

# A. スニペット と initState と setState を覚えましょう

Flutter で状態管理と聞くと何を思い浮かべますか？
Riverpod でしょうか？ Provider でしょうか？　それとも BLoC?

StatefulWidget のことを忘れていないでしょうか。

一足飛びで学習を進めてしまって、StatefulWidget のことを素通りしてしまったという人のために、StatefulWidget の基本をおさらいします。

# スニペットを使って生成する

何も見ずに StatefulWidget を書ける人はそうそういません。
みんなスニペットという補完機能を使っています。

VS Code でスニペット機能を使ってみましょう。
`st`まで入力すると Flutter Stateful Widget と出てくるのでそれを選択してください。
そのあと Widget の名前を入力します。

![](/images/q3/1.gif)

```dart
  State<SamplePage> createState() => _SamplePageState();
```

この辺りの文法がわからなくて不安という方、スニペットを使えば自動的に生成できるので積極的に使いましょう。

スニペットで Widget を生成した後は状態を作ってみましょう。

状態とは内部的に変化し得る変数です。
今回は count という状態を用意します。

```dart
int count = 0;
```

# initState を使って一度だけ実行する

initState 関数はその中に書かれた処理を一度だけ実行します。
実行のタイミングは Widget が生成された直後です。

先ほどつくった count を一度だけ 10 増加させます。

さらに、10 増加されたか確認するために画面の真ん中に count を表示させます。

```dart
class _SamplePageState extends State<SamplePage> {
  /// 状態 はここに書きます
  int count = 0;

  @override
  void initState() {
    super.initState(); // これが何か知りたい人はクラスや継承について学びましょう
    // ここから下に一度だけ実行したい処理を書きます
    count += 10;
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text('$count'),
      ),
    );
  }
}
```

ではこれを実行してみてください。
いかがでしょうか？
10 と真ん中に表示されたかと思います。

# setState で画面を更新する

今度は count を 1 秒に +1 ずつ増加させたいと思います。

while 文で無限ループを作ってみましょう。

```dart
  Future<void> loop() async {
    while (true) // 条件式がずっとtrueなので、無限ループする
    {
      // 1 秒間待つ
      await Future<void>.delayed(const Duration(seconds: 1));
      // count を 1 増加させる。（インクリメントと言います）
      count++;
      print(count); // 本当に +1 されているかコンソールで確認する。
    }
  }
```

この loop()関数を先ほどの initState の中で実行します。

```dart
  @override
  void initState() {
    super.initState(); // これが何か知りたい人はクラスや継承について学びましょう
    // ここから下に一度だけ実行したい処理を書きます
    count += 10;
    loop(); // +10 したあとに実行する
  }
```

それでは実行してみましょう。
いかがでしょうか？

DEBUG CONSOLE には次のように表示されるはずです。

```sh
I/flutter (17602): 11
I/flutter (17602): 12
I/flutter (17602): 13
I/flutter (17602): 14
I/flutter (17602): 15
```

しかしながら画面の数字は依然 10 のまま。
これはどういうことでしょうか。

実は StatefulWidget は何もしなければ画面を再描画してくれないのです。

状態は変化していますが、それが画面に反映されていません。
画面も正しく変化するように修正しましょう。

画面を再描画するには setState という関数を呼び出します。

```dart
  Future<void> loop() async {
    while (true) // 条件式がずっとtrueなので、無限ループします。
    {
      // 1 秒間待つ
      await Future<void>.delayed(const Duration(seconds: 1));
      // count を 1 増加させる。（インクリメントと言います）
      count++;
      print(count);
      setState(() {}); // これを実行すると画面が再描画されます。
    }
  }
```

実行してみましょう。

![](/images/q3/2.gif)

正しく画面が更新されました！

# 深入りは禁物

StatefulWidget は奥が深く、まだまだたくさんの機能があります。
ですがまず覚えておきたいのは、ここまでに紹介してきた スニペット, initState, setState くらいです。

実用的な部分から覚えて、徐々に知見を深めていくように心がけましょう。
最初から深いところまで知る必要はありませんので、安心してくださいね。

# コピペで動くサンプルコード

```dart
import 'package:flutter/material.dart';

void main() async {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return const MaterialApp(home: SamplePage());
  }
}

class SamplePage extends StatefulWidget {
  const SamplePage({Key? key}) : super(key: key);

  @override
  State<SamplePage> createState() => _SamplePageState();
}

class _SamplePageState extends State<SamplePage> {
  /// 状態 はここに書きます
  int count = 0;

  @override
  void initState() {
    super.initState(); // これが何か知りたい人はクラスや継承について学びましょう
    // ここから下に一度だけ実行したい処理を書きます
    count += 10;
    loop(); // 10 プラスしたあとに実行する
  }

  Future<void> loop() async {
    while (true) // 条件式がずっとtrueなので、無限ループする
    {
      // 1 秒間待つ
      await Future<void>.delayed(const Duration(seconds: 1));
      // count を 1 増加させる。（インクリメントと言います）
      count++;
      print(count);
      setState(() {}); // これを実行すると画面が再描画される。
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Text('$count'),
      ),
    );
  }
}
```

# 質問募集中

質問は随時受け付けています。
GitHub の Issue か読者コミュニティにお寄せください！
