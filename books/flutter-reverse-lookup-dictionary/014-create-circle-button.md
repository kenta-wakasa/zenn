---
title: "丸いボタンを作りたい"
---

![](/images/q14/1.gif)

# A. FloatingActionButton を使いましょう

Flutter で丸ボタンを作る方法はいろいろあります。わたしは Container や InkWell などを組み合わせて RippleEffect が出るように工夫していました。ただ最近になって FloatingActionButton（以下、FAB）を使えば簡単に実装できることに気づき、この記事を書いています。

FAB は丸いのでそのまま丸いボタンとして利用できます。よく右下に置いてあることが多いので限定的な場所でしか使えないと思われがちですが、FAB はどこにでも置けます。

たが画面から浮いているように見える Elevation が嫌いだという方や、FAB は大きすぎるという方もいらっしゃると思いますのでそこを解消するサンプルコードを書きました。

```dart
  FloatingActionButton(
    mini: true, // trueにするととで小さくなる
    onPressed: () {},
    elevation: 0, // 通常時のエレベーション
    hoverElevation: 0, // マウスホバー時のエレベーション
    highlightElevation: 0, // ボタン押下時のエレベーション
    child: Icon(Icons.face),
  ),
```

プロパティの解説はコメントをご覧ください。

それでは短いですが、最後にコピペで動くサンプルコードを紹介しておしまいにします。

## コピペで動くサンプルコード

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
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text(
          '丸いボタン',
          style: TextStyle(
            fontWeight: FontWeight.bold,
          ),
        ),
      ),
      body: const Center(
        child: FloatingActionButton(
          mini: false,
          onPressed: null,
          elevation: 0, // 通常時のエレベーション
          hoverElevation: 0, // マウスホバー時のエレベーション
          highlightElevation: 0, // ボタン押下時のエレベーション
          child: Icon(Icons.face),
        ),
      ),
    );
  }
}
```
