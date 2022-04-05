---
title: "Containerから画像がはみ出ないようにしたい"
---

# A. Clip.antiAlias を指定しましょう。

## なぜかはみ出す画像

Container で角丸な図形を作ったり、丸を作ったりしたいときがあります。そのとき、画像等を child に配置すると Container の描画領域からはみ出してしまった経験はないでしょうか？

例えばこのような状況です。

```dart
  Container(
    width: 240,
    height: 240,
    decoration: const BoxDecoration(
      // BoxShapeをcircleにしているので丸型になってほしい
      shape: BoxShape.circle,
      color: Colors.blue,
    ),
    // 正方形の画像を表示する
    // Containerは丸型なので丸くなってほしい
    child: Image.network(
      'https://pbs.twimg.com/media/FPf418SaUAA0bbN?format=jpg&name=900x900',
    ),
  ),
```

ぜひ実行してみてほしいのですが、これだと画像は正方形の状態で表示されてしまいます。
![](/images/q5/1.png)
_悲しい_

## clipBehavior プロパティに Clip.antiAlias を与える

実はデフォルトでは、はみ出した部分は切り取ってくれないようになっています。
そこを修正するには Container の clipBehavior プロパティを変更しましょう。

```dart
  Container(
    clipBehavior: Clip.antiAlias // これを指定
```

これではみ出した部分が切り取られるはずです！

![](/images/q5/1.gif)

# コピペで動くサンプルコード

```dart
import 'package:flutter/material.dart';

void main() {
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
  bool enableAntiAlias = false;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.pink[100],
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(
              clipBehavior: enableAntiAlias ? Clip.antiAlias : Clip.none,
              width: 240,
              height: 240,
              decoration: const BoxDecoration(
                shape: BoxShape.circle,
                color: Colors.blue,
              ),
              child: Image.network(
                'https://pbs.twimg.com/media/FPf418SaUAA0bbN?format=jpg&name=900x900',
              ),
            ),
            const SizedBox(height: 16),
            const Text(
              'アンチエイリアスを有効にする',
              style: TextStyle(
                fontWeight: FontWeight.bold,
              ),
            ),
            Switch(
              value: enableAntiAlias,
              onChanged: (value) {
                setState(() {
                  enableAntiAlias = value;
                });
              },
            )
          ],
        ),
      ),
    );
  }
}
```
