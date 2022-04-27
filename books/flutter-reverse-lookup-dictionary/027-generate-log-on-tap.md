---
title: タップした位置にWidgetを生成したい
---

![](/images/q27/1.gif)

# A. GestureDetector でタップ座標を取得しましょう

tap した位置に何かエフェクトを出したいことがあるかもしれません。その前段階として、タップした位置に Widget を生成する方法を紹介します。

GestureDetector の onTapDown というコールバックを使えばタップした座標情報を取得できます。

```dart
  GestureDetector(
    onTapDown: (tapDownDetails) {
      // 座標が取れる
      print(tapDownDetails.localPosition);
    }
  )
```

座標の取得ができたので、ここに Widget を生成することを考えます。指定した位置に Widget を作るには Positioned が使えます。

```dart
  Positioned(
    left: tapDownDetails.localPosition.dx,
    top: tapDownDetails.localPosition.dy,
    child: widget // 表示させたいWidgetをここに
  )
```

この Widget を配列に追加していくことにしましょう。今回は FlutterLogo をたくさん生成してみたいと思います。

状態としてまずは空配列を用意します。

```dart
  final widgets = [];
```

ここに onTapDown のタイミングで Widget を追加していきます。

```dart
  onTapDown: (tapDownDetails) {
    widgets.add(
      Positioned(
        left: tapDownDetails.localPosition.dx,
        top: tapDownDetails.localPosition.dy,
        child: const FlutterLogo(),
      ),
    );
    setState(() {});
  },
```

あとは widgets を表示させるだけですが、これは Stack を使うとよいでしょう。

```dart
  Stack(
    fit: StackFit.expand,
    children: [
      ...widgets,
      // tap できる領域を確保
      Container(
        color: Colors.transparent,
      ),
    ],
  ),
```

## コピペで動くサンプルコード

最後にコード全文を掲載しておしまいにします。

```dart
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
  final widgets = [];
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('タップした位置にWidgetを生成したい'),
      ),
      body: GestureDetector(
        onTapDown: (tapDownDetails) {
          widgets.add(
            Positioned(
              left: tapDownDetails.localPosition.dx,
              top: tapDownDetails.localPosition.dy,
              child: const FlutterLogo(),
            ),
          );
          setState(() {});
        },
        child: Stack(
          fit: StackFit.expand,
          children: [
            ...widgets,
            // tap できる領域を確保
            Container(
              color: Colors.transparent,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          widgets.clear();
          setState(() {});
        },
        child: const Icon(Icons.cleaning_services),
      ),
    );
  }
}
```
