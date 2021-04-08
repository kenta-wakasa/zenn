---
title: "Flutterでタップしたら波紋が出るやつをつくる"
emoji: "💧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, dart]
published: true
---

# はじめに

タップした場所から何かエフェクトが出る UI はスマホゲームの中などでよく見かけますよね。
今回はそれをつくってみたので紹介します。

Animation の使い方にまだ慣れていないため、冗長な書き方になっているかもしれません。
もっと優れたやり方があれば教えてください！

::: message
細かな解説はコードのコメントにまわして本文は少なめな構成です
:::

# つくったもの

@[codepen](https://codepen.io/kenta-wakasa/pen/GRrvxLO)

# 方針

およそ次のような手順で実装していきました。
それぞれの手順についてサンプルコードなど交えつつ説明していきます。

1. タップした位置を検知する

1. タップした位置に輪っかを出す

1. 輪っかをアニメーションさせる

1. 輪っかをたくさん出せるようにする

## 1. タップした位置を検知する

### GestureDetector を使う

```dart
Widget build(BuildContext context) {
  return GestureDetector(
      // onTap では position はとれないけれど、onTapDown ならとれる。
      // 他にもいろいろな動きをとれて便利です。
      // https://api.flutter.dev/flutter/widgets/GestureDetector-class.html
      onTapDown: (details) => print(details.globalPosition),
      child: const Scaffold(),
  );
}
```

## 2. タップした位置に輪っかを出す

### CustomPaint を使う

@[codepen](https://codepen.io/kenta-wakasa/pen/YzNrEew)

::: details コード

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) =>
      MaterialApp(theme: ThemeData(), home: MyHomePage());
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: ripplePointer, // 引数が同じだとこうやって短く書ける
      child: Scaffold(
        body: offset != null
            ? CustomPaint(painter: RipplePainter(offset: offset))
            : const SizedBox(),
      ),
    );
  }

  Offset offset;
  void ripplePointer(TapDownDetails details) {
    // 単純に offset を更新しているだけ
    setState(() {
      offset = details.globalPosition;
      print(details.globalPosition);
    });
  }
}

class RipplePainter extends CustomPainter {
  RipplePainter({@required this.offset});
  final Offset offset;
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..style = PaintingStyle.stroke // 図形をぬりつずつかどうか
      ..color = Colors.blue // 色の指定
      ..strokeWidth = 2; // 線の太さの指定
    canvas.drawCircle(offset, 100, paint); // 位置と輪っかの大きさを指定
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}
```

:::

## 3. 輪っかをアニメーションさせる

### AnimationController をつくって CustomPaint に渡す

まだ動きがなくて寂しいですね。
ここからアニメーションをつけていきます。

::: details コード

````dart
class RipplePointer extends StatefulWidget {
  const RipplePointer({Key key, @required this.offset, @required this.duration})
      : super(key: key);
  final Offset offset;
  final Duration duration;
  @override
  _RipplePointerState createState() => _RipplePointerState();
}

class _RipplePointerState extends State<RipplePointer>
    with SingleTickerProviderStateMixin {
  // これを忘れずに

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      painter: RipplePainter(controller: controller, offset: widget.offset),
    );
  }

  AnimationController controller;
  @override
  void initState() {
    super.initState();
    // SingleTickerProviderStateMixin を使うことで, vsync に this で渡せる。
    // これによりアニメーションの変化に合わせて画面が再描画されていく。
    controller = AnimationController(vsync: this, duration: widget.duration);
    controller.forward(); // アニメーションをスタート
  }

  @override
  void dispose() {
    controller.dispose();
    super.dispose();
  }
}

class RipplePainter extends CustomPainter {
  RipplePainter({@required this.controller, @required this.offset})
      : super(repaint: controller); // repaint に controller を渡さないと再描画されない
  final Offset offset;
  final Animation<double> controller;

  @override
  void paint(Canvas canvas, Size size) {
    /// ```dart
    /// Tween<T>(begin: ,end: ) // はじめとおわりの値を指定できる
    /// CurveTween(curve: ) // Curves. でアニメーションカーブを指定できる
    /// ```
    /// - 輪っかの大きさ
    /// - 線の太さ
    /// - 透明度
    /// この 3 つをアニメーションで制御している。
    final circleValue = Tween<double>(begin: 8, end: 80)
        .animate(controller.drive(CurveTween(curve: Curves.easeOutExpo)))
        .value;
    final widthValue = Tween<double>(begin: 12, end: 2)
        .animate(controller.drive(CurveTween(curve: Curves.easeInOut)))
        .value;
    final opacityValue = Tween<double>(begin: 1, end: 0)
        .animate(controller.drive(CurveTween(curve: Curves.easeInOut)))
        .value;

    final paint = Paint()
      ..style = PaintingStyle.stroke
      ..color = Colors.blue.withOpacity(opacityValue)
      ..strokeWidth = widthValue;
    canvas.drawCircle(offset, circleValue, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}

````

:::

## 4. 輪っかをたくさん出せるようにする

これが意外と難しく、いまだに最適解がわかっていません。
Stack に List を渡すことで複数の波紋を持てるようにして、アニメーションが終了したら remove するような実装になっています。
Unity とかだと destroy とか generate みたいなのがあって画面にオブジェクトを生成したり廃棄したりするのが簡単なのですが、Flutter はそういう実装どうするのがいいんですかね。

::: details コード

```dart
class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTapDown: generateRipplePointer,
      child: Scaffold(
        body: Stack(children: ripplePointerList),
      ),
    );
  }

  // RipplePointer のリストをつくってタップと同時にそこに追加している
  // アニメーションの終了を Future<void>.delayed で待ち、終わった時に removeAt(0) でリストから取り出している
  // 取り出すと そのタイミングで dispose が呼ばれる。
  List<RipplePointer> ripplePointerList = <RipplePointer>[];
  Future<void> generateRipplePointer(TapDownDetails details) async {
    const duration = const Duration(milliseconds: 800);
    final ripplePointer = RipplePointer(
      key: UniqueKey(), // 必ずキーを与えること。これによりそれぞれが独立した描画になります。
      offset: details.globalPosition,
      duration: duration,
    );
    setState(() {
      ripplePointerList.add(ripplePointer);
    });
    await Future<void>.delayed(duration);
    setState(() {
      ripplePointerList.removeAt(0);
    });
  }
}
```

:::

# 全体像

アニメーションカーブや線の太さ、大きさなどのパラメータをいじって遊んでみてください！
::: details コード全文
@[gist](https://gist.github.com/kenta-wakasa/785d1250e967e754d4faf3913456b75c)
:::
それではまたどこかで。
