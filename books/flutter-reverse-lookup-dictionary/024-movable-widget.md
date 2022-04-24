---
title: Widgetを自由に動かしたい
---

![](/images/q24/1.gif)

# A. GestureDetector と Positioned を組み合わせましょう

Widget をタップしながら自由な場所に移動させたいときがあります。2 つの Widget を簡単に組み合わせるだけで実装できますのでやっていきましょう。

自由な場所に移動させるためには、

1. tap している場所情報を取得する
1. Widget をその場所に配置する

この 2 つのことが必要です。

## GestureDetector について

1 は GestureDetector で達成することができます。使い方はこうです。

```dart
  GestureDetector(
    /// tapしながら動かしているときの座標が取れる
    onPanUpdate: (dragUpdateDetails) {
      print(dragUpdateDetails.localPosition);
    },
    child: child // 何かしらのWidget
  )
```

onPanUpdate を使えばドラッグしている指やマウスの位置が変化するたびにその座標情報を取得することができます。

## Positioned について

ここで取得した座標情報をつかって Widget を配置するには Positioned を使います。Positioned はこ Widget を好きな位置に配置することができる Widget です。

使い方はこんな感じです。

```dart
  Positioned(
    // 左上からどれだけ右にあるか
    left: 100,
    // 左上からどれだけ下にあるか
    top: 200,
    child: child // 何かしらのWidget
  ),
```

基準点を左端にして、そこからどれだけ離れているのか、基準点を上端にしてそこから離れているのか、などを指定することで子 Widget の位置を指定できます。

## コピペで動くサンプルコード

それではこれらを組み合わせて作ったサンプルコードを紹介して、おしまいにします。

```dart
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: DemoPage(),
    );
  }
}

class DemoPage extends StatefulWidget {
  const DemoPage({Key? key}) : super(key: key);

  @override
  State<DemoPage> createState() => _DemoPageState();
}

class _DemoPageState extends State<DemoPage> {
  var position = const Offset(0, 0);
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Widgetを自由に動かしたい'),
      ),
      body: GestureDetector(
        // ドラッグのスタートをタップした直後に設定
        dragStartBehavior: DragStartBehavior.down,
        onPanUpdate: (dragUpdateDetails) {
          position = dragUpdateDetails.localPosition;
          setState(() {});
        },
        child: Stack(
          children: [
            Positioned(
              // 左上からどれだけ右にあるか
              left: position.dx,
              // 左上からどれだけ下にあるか
              top: position.dy,
              child: const FlutterLogo(
                size: 80,
              ),
            ),
            Container(
              color: Colors.transparent,
            ),
          ],
        ),
      ),
    );
  }
}
```
