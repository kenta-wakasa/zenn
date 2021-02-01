---
title: "【Flutter】ライフゲームそれは人生"
emoji: "👨‍👩‍👧‍👦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, firebase, web, dart]
published: true
---

# はじめに

すごろくの方ではない[人生ゲーム](https://ja.wikipedia.org/wiki/ライフゲーム)を Flutter web で作ってみました。
:::details 環境
マシン: M1 MacBook Air
エディタ: VSCode
リポジトリ: https://github.com/kenta-wakasa/game_of_life
:::

# 作ったもの

https://twitter.com/pressedkonbu/status/1355750891085721608

ライフゲームというのは、小さなセルがアルゴリズムに従って生まれたり死んだりするゲームです。

## 遊び方

1. セルを好きに塗りつぶして初期条件を決定する
1. 再生ボタンを押してスタート
1. 眺める

特にゲーム性はないのですが、単純なルールで複雑な図形が生きているように動きます。

これが不思議で、眺めているとなんだか落ち着く、そんなゲームです。

[ここ](https://game-of-life-93ab9.web.app/#/)から遊べるので、眠れない夜にどうぞ。

# ライフゲームのアルゴリズム

ライフゲームではセルが生と死の状態を持っています。

それらのセルの周囲 8 マスに生きているセルが何個いるかで次の世代での生死が決定します。

生きているセルと、死んでいるセルのそれぞれついてアルゴリズムを紹介します。

## 生きているセル

**周囲に生きているセルが 2 つか 3 つの場合**に次の世代で生き残れる。

## 死んでいるセル

**周囲に生きているセルが 3 つの場合**に次の世代で生き返れる。

## 具体例

次のようにセルを表します。
生きている → 🟠
死んでいる → 🔵
真ん中のセルがどうなるかについてみていきます。

### 生き残れる例

```
🟠🟠🔵 　　　　 🔵🟠🔵 　　　　 🔵🔵🟠 　　　　 🟠🔵🔵
🟠🟠🔵 　　　　 🟠🟠🟠 　　　　 🔵🟠🔵 　　　　 🔵🟠🟠
🔵🔵🔵 　　　　 🔵🟠🔵 　　　　 🟠🔵🔵 　　　　 🔵🔵🔵
```

### 死んでしまう例

```
🟠🟠🔵 　　　　 🟠🟠🟠 　　　　 🔵🔵🔵 　　　　 🔵🔵🔵
🟠🟠🔵 　　　　 🟠🟠🟠 　　　　 🔵🟠🔵 　　　　 🔵🟠🔵
🔵🔵🟠 　　　　 🟠🟠🟠 　　　　 🟠🔵🔵 　　　　 🔵🔵🔵
```

### 生き返れる例

```
🟠🟠🔵 　　　　 🟠🟠🟠 　　　　 🟠🔵🔵 　　　　 🔵🔵🟠
🔵🔵🔵 　　　　 🔵🔵🔵 　　　　 🔵🔵🟠 　　　　 🔵🔵🔵
🔵🔵🟠 　　　　 🔵🔵🔵 　　　　 🔵🟠🔵 　　　　 🟠🔵🟠
```

ざっくりいうと、孤独すぎても密集しすぎても死んでしまうのです。
ちょうどよい距離感のときに繁栄します。

あゝ人生。

# 実装方針

1. Cell class の実装
1. セルの描画
1. 初期条件の決め方
1. アルゴリズムの実装

# 1. Cell class の実装

セルは width × height の数だけ

```dart
List.generate(width * height, (index) => false);
```

みたいな感じで生成してしまうのが一番早いと思います。

一次元配列であっても

```
0 1 2
3 4 5
6 7 8
```

このようにとらえてあげればよいわけです。
このときの x 座標と y 座標は

```
x: index  % width
y: index ~/ width
```

で表現できますね。

ですが今回はわざわざ x 座標と y 座標を直接持たせたクラスを作りました。

こちらの方が直感的なので可読性が上がると思ったんですよね。

```dart:cell.dart
import 'dart:math';

import 'package:flutter/material.dart';

@immutable
class Cell {
  const Cell({
    this.alive = false,
    @required this.pos,
  });

  /// 持っている変数はこの二つだけです。
  /// point は math.dart で定義されています。 x と y を持つので使わせてもらっています。
  final bool alive;
  final Point<int> pos;

  Cell copyWith({bool alive, Point<int> pos}) =>
      Cell(alive: alive ?? this.alive, pos: pos ?? this.pos);

  /// これは width と height を与えるとその数分のリストを生成してくれるメソッドです。
  static List<Cell> generateCells(int width, int height) {
    final cellList = <Cell>[];
    for (var y = 0; y < height; y++) {
      for (var x = 0; x < width; x++) {
        cellList.add(Cell(pos: Point<int>(x, y)));
      }
    }
    return cellList;
  }

  /// これは index から 座標変換の逆ですね。
  /// 座標から index に変換しています。
  /// せっかく x と y の情報を持たせたのにと思ったそこのあなた。自分もそう思いました。
  /// ですが、index 使った方が検索性が高くて都合がいいんですよね。
  static int pointToIndex(Point<int> point, int width) {
    return point.x + point.y * width;
  }

  /// 周囲のセルの場所を返すメソッドです
  /// (-1, -1) ( 0, -1), ( 1, -1)
  /// (-1,  0) ( 0,  0), ( 1,  0)
  /// (-1,  1) ( 0,  1), ( 1,  1)
  /// このように考えると 自分の位置から x と　y　の方向に -1 だけずれたところから開始して
  /// 入れ子の for を回せばよさそうですね。あとは自分自身は除く処理を加えれば完成です。
  List<Point<int>> get getPointAroundMe {
    final pointList = <Point<int>>[];
    for (var x = 0; x < 3; x++) {
      for (var y = 0; y < 3; y++) {
        final point = Point<int>(pos.x - 1 + x, pos.y - 1 + y);
        if (pos != point) {
          pointList.add(point);
        }
      }
    }
    return pointList;
  }
}
```

# 2. セルの描画

Flutter にはセルを表現する方法がいくつかあります。
ぱっと思いついたのは [GridView](https://api.flutter.dev/flutter/widgets/GridView-class.html) で [Container](https://api.flutter.dev/flutter/widgets/Container-class.html) を並べる方法です。

```dart
GridView.builder(
    gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: width
    ),
    itemCount: width * height,
    itemBuilder: (BuildContext context, int index) {
        return Container( 色とか大きさとか );
    },
)
```

こんな感じでいけそうですよね。

ですが 100 × 100 くらいの規模のセルを並べると、かなりの重さで動きが固まってしまいました。

そのため今回は低レイヤーの描画手法である [CustomPaint](https://api.flutter.dev/flutter/widgets/CustomPaint-class.html) を使ってグリッドを表現します。

```dart:painter.dart
import 'package:flutter/material.dart';
import 'package:game_of_life/cell.dart';

class Painter extends CustomPainter {
  Painter({
    @required this.basicLength,
    @required this.width,
    @required this.height,
    @required this.cells,
  });
  double basicLength; // 1 グリッドの長さ
  int width;
  int height;
  List<Cell> cells;

  void paintCells(Canvas canvas, Paint paint, List<Cell> cells) {
    for (final cell in cells) {
      if (cell.alive) {
        /// 正方形を描画しているのはこの部分
        /// Rect.fromLTWH は
        /// legt, top, width, height を与えて 長方形を作る つまり...
        /// 左上の x 座標, 左上の y 座標, 指定した頂点から x 方向への長さ, 指定した頂点から y 方向への長さ
        /// これで 長方形を表現する。
        canvas.drawRect(
          Rect.fromLTWH(
            basicLength * cell.pos.x,
            basicLength * cell.pos.y,
            basicLength,
            basicLength,
          ),
          paint,
        );
      }
    }
  }

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..style = PaintingStyle.fill
      ..color = Colors.yellow[700];

    paintCells(canvas, paint, cells);
  }

  @override
  bool shouldRepaint(Painter oldDelegate) {
    /// 再描画可能かどうかのフラグ
    return true;
  }
}
```

ここで定義した Painter クラスはたとえばこのように呼んで使います。

```dart
body: CustomPaint(
    painter: Painter(
    basicLength: _provider.baseLength,
    height: _provider.height,
    width: _provider.width,
    cells: _provider.cellList,
    ),
),
```

CustomPaint はひとつの Widget としてみてください。

# 3. 初期条件の決め方

ゲーム開始時にランダムで塗りつぶす箇所を生成するなどの方法が考えられます。

今回は自由に初期条件を決めたかったので、クリックした箇所の bool が反転するようにしました。

クリックは Widget を使っているなら [Inkwell](https://api.flutter.dev/flutter/material/InkWell-class.html) でラップすればいいのですが、今回は CustomPaint を使っているのでそうもいきません。

こんなときは [GestureDetector](https://api.flutter.dev/flutter/widgets/GestureDetector-class.html) を使って直接画面の座標を取得しましょう。

取得した座標と該当する cell を紐づけて反転するという方針でやっています。

座標を取るのは簡単です。

```dart
GestureDetector(
    /// onTapDown はクリックした時に呼ばれる details のなかにいろいろ情報が入っている。
    onTapDown: (details) {
        if (_provider.editable) {
            /// .localPosition で親 Widget の中での相対的な位置を取得することができる。
            print(details.localPosition);
        }
    },
)
```

ここで得られる position 情報を idnex 情報に変換して活用しています。

変換のコードはこんな感じです。

```dart
int positionToIndex(Offset position) =>
      position.dx ~/ baseLength + position.dy ~/ baseLength * width;
```

Cell class で定義した pointToIndex に baseLength が入っただけですね。

# 4. アルゴリズムの実装

Cell クラスの中で周囲 8 マスの座標リストを取ってくるメソッドを定義したのでもう簡単です。

```dart
/// 全ての cells について次の世代の生死を調べる
  void _nextGenerations() {

    /// もとあるリストに上書きしていく計算途中で値が変化していってしまいます。
    /// なので一度別リストとして避難させています。
    final tmpList = List<Cell>.from(_cellList);
    for (var index = 0; index < cellList.length; index++) {
      final pointList = cellList[index].getPointAroundMe;
      var count = 0;
      for (final point in pointList) {
        /// 座標が 設定した width height を超えていないかチェックしています。
        if (point.x > -1 &&
            point.x < width &&
            point.y > -1 &&
            point.y < height) {

            /// ここでわざわざ index に戻して使っています。
            /// こうすれば配列を直接指定して調べられますね。
          if (cellList[Cell.pointToIndex(point, width)].alive) {
            count++;
          }
        }
      }
      /// セルが生きていれば 2, 3 で生存
      if (cellList[index].alive) {
        if (count < 2 || count > 3) {
          tmpList[index] = tmpList[index].copyWith(alive: false);
        }
        /// セルが死んでいれば 3 で誕生
      } else {
        if (count == 3) {
          tmpList[index] = tmpList[index].copyWith(alive: true);
        }
      }
      /// 現状維持を除くと、条件ってこの二つしかないんですよね。
    }
    /// 最後に一時退避していたリストを元のリストに返して更新をかけています。
    _cellList = tmpList;
  }
```

解説は以上です！　コードの全体像はこちらを参考にしてください。
https://github.com/kenta-wakasa/game_of_life

# おしまいに

使う Widhet によってこんなにも重さが変わるのかと実感したプロジェクトでした。
これとはまた違った実装方針で作っているものを見てみたいですね。
1 日くらいでできると思うので、暇つぶしにいかがでしょうか。
