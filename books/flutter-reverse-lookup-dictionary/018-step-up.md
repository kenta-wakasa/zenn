---
title: 少しずつ進んでいくWidgetを作りたい
---

# A. AnimatedPositioned を使いましょう

何かを成しとけで成長したときなど、Widget を一定の規則性で移動させて変化を示したいことがあると思います。今回はその方法をご紹介します。

AnimatedPositioned は更新前と更新後のパラメータの変化に応じて決められた秒数でアニメーションしながら Widget を動かすことができます。

```dart
  AnimatedPositioned(
    duration: const Duration(milliseconds: 400),
    left: null, // ここに位置情報を記述する
    top: null, // ここに位置情報を記述する
    right: null, // ここに位置情報を記述する
    bottom: position, // ここに位置情報を記述する
    child: Container( // ここは動かしたいWidgetを記述する
      width: 50,
      height: 50,
      color: Colors.amber,
    ),
  ),
```

下から上へ動かしたいのであれば、bottom にはじめ 0 を与え、順番に数字を増やしていくことで対象の Widget が上昇していきます。

サンプルとして、ただのオレンジ色の箱が FloatingActionButton を押すごとに上に昇っていくコードを書きましたので参考にしてみてください！

## コピペで動くサンプルコード

![](/images/q18/1.gif)

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
  var position = 0.0;

  void stepUp() {
    setState(() {
      position += 50;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        alignment: Alignment.bottomCenter,
        children: [
          AnimatedPositioned(
            duration: const Duration(milliseconds: 400),
            left: null, // ここに位置情報を記述する
            top: null, // ここに位置情報を記述する
            right: null, // ここに位置情報を記述する
            bottom: position, // ここに位置情報を記述する
            child: Container(
              // ここは動かしたいWidgetを記述する
              width: 50,
              height: 50,
              color: Colors.amber,
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: stepUp,
        child: const Icon(Icons.arrow_upward),
      ),
    );
  }
}
```
