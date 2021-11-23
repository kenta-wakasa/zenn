---
title: "StackとAlignで好きな場所に配置する"
emoji: "🛰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter]
published: true
---

# つくったもの

@[codepen](https://codepen.io/kenta-wakasa/pen/MWvLpOe)

# はじめに

Widget を他の Widget の配置に関係なく自由な場所に配置したいことってあると思います。

Column や Row でやろうとすると、ほかの Widget の大きさなどが影響して位置がずれてしまうので難しいです。

今回はそれを Stack と Align を使って解決するお話です。

# 方針

1. Stack で描画範囲を指定する。（今回は画面全域）
2. Align で描画位置を指定する。

ある特定の描画範囲内で Widget を Stack していけば、ほかの Widget の位置に左右されることなく、狙った場所に Widget を配置できるようになります。

配置系の Widget には Positioned や Align がありますが、今回は相対的な位置関係を指定して配置させたかったので Align を使っています。

座標指定の間隔で絶対的な位置に Widget を配置したい方は Positioned を使うとよいでしょう。

## 1. Stack で描画範囲を指定する。（今回は画面全域）

Stack は Widget を重ねて表示させるための Widget です。
children に与えた Widget が重なっていきます。

重なり方の順番は小さい番号の index が奥、大きい番号の index が手前に表示されます。

今回、画像データを画面いっぱいに表示させたかったので（サンプルコードではコメントアウトしています）、fit というパラメータに StackFit.expand を与えています。

```dart
Stack(
  fit: StackFit.expand, // これを設定すると画面いっぱいに画像が描画されます。
  children: [] // ここの表示させたいWidget
)
```

## 2. Align で描画位置を指定する。

何か配置させたい Widget を Align でラップすることによって狙った場所に配置できます。

```dart
Align(
  alignment: const Alignment(-0.8, 0.4),
  child: widget // なにかしらのWidget
)
```

重要なのは alignment というパラメータです。
ここに Alignment を与えることで位置を指定ます。

第 1 引数が横軸の相対的な位置、第 2 引数が縦軸の相対的な位置を表現していて、それぞれ -1 ~ 1 の範囲を取ります。

具体的には次のような感じです。

- 左上に配置したい場合は Alignment(-1, -1)
- 右上に配置したい場合は Alignment(1, -1)
- 中央に配置したい場合 Alignment(0, 0)

今回は Stack と Align を使って、ランダムな位置に星を散りばめるサンプルコードを作成しましたので、参考にしていただければ幸いです。説明が必要そうな部分で適宜コメントを入れていますのでそちらも合わせてご覧ください。

# サンプルコード

```dart
import 'package:flutter/material.dart';
import 'dart:math';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home:StackSample(),
    );
  }
}
class StackSample extends StatefulWidget {
  const StackSample({Key? key}) : super(key: key);

  @override
  _StackSampleState createState() => _StackSampleState();
}

class _StackSampleState extends State<StackSample> {
  final random = Random();

  double get randomValue => (random.nextDouble() * 2) - 1;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        fit: StackFit.expand, // これを設定すると画面いっぱいに画像が描画されます。
        children: [
          Container(
            color: Colors.black,
          ),
          // Image.network(
          //   'https://pbs.twimg.com/media/EXJpPggVcAIfDtH?format=jpg&name=4096x4096',
          //   fit: BoxFit.cover, // 縦横比を変えず両端を切り落とす形で表示してくれる。
          // ),
          Align(
            // 縦横 -1 ~ 1 の範囲で自由に表示場所を指定できる
            // Alignment(横軸で表示させたい場所を -1 ~ 1 の範囲で指定 , 縦軸で表示させたい場所を -1 ~ 1 の範囲で指定)
            // -1 ~ 1 を越えた値を指定するとstackの描画範囲を超えて表示させることも可能（今回は扱わない）
            alignment: const Alignment(-0.8, 0.4),
            child: Column(
              // Columnは縦いっぱいに広がろうとしてしまうので、minを与えないとレイアウトが崩れます。
              mainAxisSize: MainAxisSize.min,
              crossAxisAlignment: CrossAxisAlignment.start,
              children: const [
                Text(
                  'Align',
                  style: TextStyle(
                    fontWeight: FontWeight.bold,
                    fontSize: 40,
                    color: Colors.white,
                  ),
                ),
                Text(
                  '自由にWidgetを配置する',
                  style: TextStyle(
                    // fontWeight: FontWeight.bold,
                    fontSize: 20,
                    color: Colors.white,
                  ),
                ),
              ],
            ),
          ),
          for (var i = 0; i < 100; i++)
            Align(
              alignment: Alignment(randomValue, randomValue),
              child: Icon(
                Icons.star,
                color: Colors.amber,
                size: random.nextInt(30) * 1.0,
              ),
            ),
        ],
      ),
    );
  }
}

```
