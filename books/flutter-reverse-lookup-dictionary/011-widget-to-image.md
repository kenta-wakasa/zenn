---
title: "Widgetを画像化したい"
---

# A. RenderRepaintBoundary の toImage メソッドを使いましょう

Widget を画像化できたら SNS などに共有できるので、アプリがバズるための一助となるかもしれません。そこで今回は Widget を画像化する方法を紹介します。

## 手順

およそ次のような手順で達成できます。

1. 画像化したい Widget を RepaintBoundary でラップする
1. 画像化したい Widget に GlobalKey を渡す。
1. GlobalKey から context を引っ張ってきて findRenderObject を実行する。
1. 得られたものを RenderRepaintBoundary にキャストする。
1. RenderRepaintBoundary の toImage を用いて画像データを取得する。

サンプルコードを書きましたのでそちらを参考にしていただければと思います！

# コピペで動くサンプルコード

![](/images/q11/1.gif)

```dart
import 'dart:typed_data';
import 'dart:ui';

import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

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
  Uint8List? bytes;
  final globalKey = GlobalKey();

  Future<void> widgetToImage() async {
    final boundary = globalKey.currentContext?.findRenderObject() as RenderRepaintBoundary?;
    if (boundary == null) {
      return;
    }
    final image = await boundary.toImage();
    final byteData = await image.toByteData(format: ImageByteFormat.png);
    bytes = byteData?.buffer.asUint8List();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            child: Column(
              children: [
                ElevatedButton(
                  child: const Text('画像化'),
                  onPressed: () async {
                    await widgetToImage();
                    setState(() {});
                  },
                ),
                const SizedBox(height: 16),

                // この widget を画像化したい
                RepaintBoundary(
                  key: globalKey,
                  child: Container(
                    decoration: BoxDecoration(
                      border: Border.all(),
                    ),
                    width: 320,
                    height: 200,
                    alignment: Alignment.center,
                    child: Padding(
                      padding: const EdgeInsets.all(16),
                      child: TextFormField(),
                    ),
                  ),
                ),
                const SizedBox(height: 32),
                const Text('この下にイメージ化したWidgetが表示されます'),
                const SizedBox(height: 32),
                // もし画像化されればここに表示される
                if (bytes != null) Image.memory(bytes!.buffer.asUint8List())
              ],
            ),
          ),
        ),
      ),
    );
  }
}
```
