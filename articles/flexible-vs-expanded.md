---
title: "Flexible と Expanded って何が違うんだっけ？"
emoji: "🧽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter"]
published: true
---

![](https://storage.googleapis.com/zenn-user-upload/22286993c6f1-20220310.png)

# 何度も調べたけれど、結局なんだっけ？

Flexible と Expanded は Flutter のレイアウトにおいてとても重要な Widget です。
しかしながらこの２つ、よく似た特性をもっていて、違いを忘れがちです。

今回はこの 2 つの Widget について改めて調査をしてみました。
これでもう忘れないぞというレベルまで言語化できましたので紹介していきます！

# つまるところ

このシンプルな表にまとめることができました。

|                                       | Flexible             | Expanded         |
| ------------------------------------- | -------------------- | ---------------- |
| 最大サイズより child が「大きい」場合 | 最大サイズになる     | 最大サイズになる |
| 最大サイズより child が「小さい」場合 | **サイズに変化なし** | 最大サイズになる |

それでは解説していきます。

# 詳しい解説

## Expanded の特性

基本的にはまず Expanded の特性を押さえると話がわかりやすくなります。
Expanded は子要素を描画領域の最大サイズまで引き伸ばす、もしくは縮小させる Widget です。

具体例を見ていきましょう。
ここに横幅 300, 縦幅 300 の Container があったとします。

![](https://storage.googleapis.com/zenn-user-upload/aa73c5956086-20220311.png)

ここに Row の中に Expanded で囲んだ横幅 100, 縦幅 100 の ColoredBox を入れたらどうなるでしょうか？

コードで書くとこんな感じです。

ぜひ結果を予測してみてください。

```dart
  Container(
    width: 300,
    height: 300,
    decoration: BoxDecoration(
      color: Colors.grey[300],
    ),
    child: Row(
      children: const [
        Expanded(
          child: ColoredBoxWithSize(
            width: 100,
            heigh: 100,
            color: Colors.blue,
          ),
        ),
      ],
    ),
  ),
```

では実際はどうなるかというと...

![](https://storage.googleapis.com/zenn-user-upload/32fefe874891-20220311.png)

このようになります。

子要素の横幅は 100 ですが、親要素の描画領域の最大である 300 まで引き伸ばされています。

ここでもうひとつ注目すべきは縦幅です。

よくみると縦幅は 100 のままで変化していません。

Row や Column の中で Expanded や Flexible を使う場合、その Cross 方向には引き伸ばされないのです。Row であれば横幅に影響を与え、Column であれば縦幅に影響を与えるということですね。

## Flexible の特性

では Flexible だとどうなるのでしょうか？

先ほどと同じように親の 300×300 の Container の中に Flexible で囲った 100×100 の ColoredBox を入れてみます。

コードで書くとこんな感じです。

ぜひまた結果を予測してみてください。

```dart
  Container(
    width: 300,
    height: 300,
    decoration: BoxDecoration(
      color: Colors.grey[300],
    ),
    child: Row(
      children: const [
        Flexible(
          child: ColoredBoxWithSize(
            width: 100,
            heigh: 100,
            color: Colors.blue,
          ),
        ),
      ],
    ),
  ),
```

結果はこうなります。

![](https://storage.googleapis.com/zenn-user-upload/fe6c1401a3b8-20220311.png)

このように **Flexible は子要素のサイズが親の描画領域より小さければ変化しない**のです。

これが Flexible のもつ Expanded との大きな違いです。

逆を言えばこれ以外はすべて同じ特性を持っています。

## 最大描画領域よりも子要素が大きい場合

では、子要素が親の描画領域を超えていた場合はどうなるのでしょうか？

子要素のサイズを 5000 × 100 にして実験してみます。

今度は両方並べていっきにみてみましょう。

![](https://storage.googleapis.com/zenn-user-upload/3d6fb98b1b98-20220311.png)

そうです。この場合、結果は同じになります。
子要素のサイズが親の最大描画領域よりはみ出していた場合は、その最大サイズに収まるようなるのです。

ここで最初に登場させた表をもう一度見てみましょう。
すんなりと理解できるのではないでしょうか。

|                                       | Flexible             | Expanded         |
| ------------------------------------- | -------------------- | ---------------- |
| 最大サイズより child が「大きい」場合 | 最大サイズになる     | 最大サイズになる |
| 最大サイズより child が「小さい」場合 | **サイズに変化なし** | 最大サイズになる |

実はとてもシンプルな違いでした。
これでもう忘れずにすみそうですね。

今回実験用に作った[サンプルプロジェクト](https://dartpad.dartlang.org/?id=6e53ae105a75054430bb77af7d0da9a6)をブラウザで実行できますので、自由に遊んでいただければと思います。

https://dartpad.dartlang.org/?id=6e53ae105a75054430bb77af7d0da9a6

それではまた次回どこかでお会いしましょう！

# 付録：コード全文

```dart
import 'dart:ui';

import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Flexible vs. Expanded',
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
      body: SafeArea(
        child: Center(
          child: SingleChildScrollView(
            scrollDirection: Axis.horizontal,
            child: SingleChildScrollView(
              child: Row(
                mainAxisAlignment: MainAxisAlignment.center,
                children: [
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.center,
                    children: [
                      const Text(
                        'Flexible',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 2),
                      Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          Container(
                            width: 300,
                            height: 300,
                            decoration: BoxDecoration(
                              color: Colors.grey[300],
                            ),
                            child: Row(
                              children: const [
                                Flexible(
                                  child: ColoredBoxWithSize(
                                    width: 5000,
                                    heigh: 100,
                                    color: Colors.blue,
                                  ),
                                ),
                                Flexible(
                                  child: ColoredBoxWithSize(
                                    width: 100,
                                    heigh: 200,
                                    color: Colors.amber,
                                  ),
                                ),
                              ],
                            ),
                          ),
                          const RotatedBox(
                            quarterTurns: 1,
                            child: Text(
                              '300',
                              style: TextStyle(color: Colors.grey),
                            ),
                          ),
                        ],
                      ),
                      const Text(
                        '300',
                        style: TextStyle(color: Colors.grey),
                      ),
                    ],
                  ),
                  const SizedBox(
                    height: 40,
                    width: 40,
                  ),
                  Column(
                    crossAxisAlignment: CrossAxisAlignment.center,
                    children: [
                      const Text(
                        'Expanded',
                        style: TextStyle(
                          fontSize: 20,
                          fontWeight: FontWeight.bold,
                        ),
                      ),
                      const SizedBox(height: 2),
                      Row(
                        mainAxisSize: MainAxisSize.min,
                        children: [
                          Container(
                            width: 300,
                            height: 300,
                            decoration: BoxDecoration(
                              color: Colors.grey[300],
                            ),
                            child: Row(
                              children: const [
                                Expanded(
                                  child: ColoredBoxWithSize(
                                    width: 5000,
                                    heigh: 100,
                                    color: Colors.blue,
                                  ),
                                ),
                                Expanded(
                                  child: ColoredBoxWithSize(
                                    width: 100,
                                    heigh: 200,
                                    color: Colors.amber,
                                  ),
                                ),
                              ],
                            ),
                          ),
                          const RotatedBox(
                            quarterTurns: 1,
                            child: Text(
                              '300',
                              style: TextStyle(color: Colors.grey),
                            ),
                          ),
                        ],
                      ),
                      const Text(
                        '300',
                        style: TextStyle(color: Colors.grey),
                      ),
                    ],
                  ),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}

class ColoredBoxWithSize extends StatelessWidget {
  const ColoredBoxWithSize({
    Key? key,
    required this.width,
    required this.heigh,
    required this.color,
  }) : super(key: key);
  final double width;
  final double heigh;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return ColoredBox(
      color: color,
      child: SizedBox(
        width: width,
        height: heigh,
        child: Center(
          child: Text(
            '$width × $heigh',
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.white,
              fontFeatures: [FontFeature.tabularFigures()],
            ),
          ),
        ),
      ),
    );
  }
}

```
