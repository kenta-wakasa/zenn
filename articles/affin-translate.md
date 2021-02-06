---
title: "【Flutter】アフィン変換で星をまわせ"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [flutter, math, dart]
published: true
---

# 作ったもの

https://twitter.com/pressedkonbu/status/1357907770662522881

:::details 環境
マシン: M1 MacBook Air
エディタ: VSCode
リポジトリ: https://github.com/kenta-wakasa/flutter_playground
:::

# ざっくりアフィン変換の話

アフィン変換を簡単に式で表すとこのようなことになっています。

$$
y = Ax + t\tag1
$$

これがなにを示しているかというと、$x$ に何らかの変化を与える係数 $A$ と平行を移動を示す $t$ によって、$x$ が $y$ に変化しますよということです。

コンピュータグラフィックの世界では、このありがたい数式によって、図形を拡大したり回転したり平行移動したりすることができます。

たとえば、( 1 ) の式を　 x 座標, y 座標にあてはめて考えると次のようになります。

$$
\begin{cases}
x' = ax+by+x_0 \\
y' = cx+dy+y_0
\end{cases}
$$

これを行列で書くとこんな感じです。自分で展開してみるとよくわかると思います。

$$
\begin{bmatrix}
x'\\
y'\\
1
\end{bmatrix}
=
\begin{bmatrix}
a & b & x_0\\
c & d & y_0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

この $a, b, c, d$ の部分が ( 1 ) での A に相当するので

$$
A =
\begin{bmatrix}
a & b \\
c & d\\
\end{bmatrix}
$$

となっています。$x , y$ を $\theta$ だけ回転させたい場合の行列は次のように表すことができます。

$$
A =
\begin{bmatrix}
\cos\theta & -\sin\theta \\
\sin\theta &  \cos\theta\\
\end{bmatrix}
$$

いわゆる回転行列というやつです。
ここに定数をかけてやれば $x$ と $y$ 方向に同じ比率で拡大と縮小ができます。
つまり定数を $a$ とするとこう書けます。

$$
A = a
\begin{bmatrix}
\cos\theta & -\sin\theta \\
\sin\theta &  \cos\theta\\
\end{bmatrix}
$$

あとはこれをコードに落とし込むだけ！

# 書いたソースコード

アフィン変換はこうなります。
といっても、拡大、縮小、回転、平行移動だけで、せん断とかはやっていません。
```dart:アフィン変換をやってる部分
  Offset _affinTranslate(
    Offset pos, {
    double radians = 0,
    double scale = 1.0,
    double tx = 0,
    double ty = 0,
  }) {
    final dx = scale * (pos.dx * cos(radians) - pos.dy * sin(radians)) + tx;
    final dy = scale * (pos.dx * sin(radians) + pos.dy * cos(radians)) + ty;
    return Offset(dx, dy);
  }

```
----
**コントローラー**
星を書いているのでその部分も何かの参考になればと思います。
コードで図形を描くの楽しいですね。
````dart:canvas_controller.dart
import 'dart:math';

import 'package:flutter/material.dart';

class CanvasController extends ChangeNotifier {
  CanvasController() {
    _radians = 0;
    _scale = 1;
    _tx = 0;
    _ty = 0;
    _initOffset = const Offset(0, -10); // 星型の最初の頂点を決める。
    _sourceOffsetList.add(_initOffset);
    // 星型は同一円の円周上を 4π/5 ずつ回転させた点を結ぶことで描ける。
    for (var radians = 0.0; radians <= 4 * pi; radians += 4 * pi / 5) {
      _sourceOffsetList.add(_affinTranslate(_initOffset, radians: radians));
    }
    _destinationOffsetList = _sourceOffsetList;
  }

  double _radians;
  double get radians => _radians;
  set radians(double radians) {
    _radians = radians;
    _update();
  }

  double _scale;
  double get scale => _scale;
  set scale(double scale) {
    _scale = scale;
    _update();
  }

  double _tx;
  double get tx => _tx;
  set tx(double tx) {
    _tx = tx;
    _update();
  }

  double _ty;
  double get ty => _ty;
  set ty(double ty) {
    _ty = ty;
    _update();
  }

  Offset _initOffset;
  final _sourceOffsetList = <Offset>[];
  List<Offset> _destinationOffsetList;
  List<Offset> get destinationOffsetList => _destinationOffsetList;

  /// 拡大・回転・平行移動のアフィン変換は次のようになっている
  /// ```
  /// dx =  scale　*　( x*cosθ - y*sinθ ) + tx
  /// dy =  scale　*　( x*sinθ + y*cosθ ) + ty
  /// ```
  Offset _affinTranslate(
    Offset pos, {
    double radians = 0,
    double scale = 1.0,
    double tx = 0,
    double ty = 0,
  }) {
    final dx = scale * (pos.dx * cos(radians) - pos.dy * sin(radians)) + tx;
    final dy = scale * (pos.dx * sin(radians) + pos.dy * cos(radians)) + ty;
    return Offset(dx, dy);
  }

  /// すべての offset に対してアフィン変換をかける
  void _update() {
    _destinationOffsetList = _sourceOffsetList
        .map((s) =>
            _affinTranslate(s, radians: radians, scale: scale, tx: tx, ty: ty))
        .toList();
    notifyListeners();
  }
}

````
----
**ページ**
スライダーとかテキストボックスで入力をやっています
```dart:canvas_page.dart
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'canvas_controller.dart';
import 'painter.dart';

final canvasProvider = ChangeNotifierProvider.autoDispose<CanvasController>(
  (ref) => CanvasController(),
);

class CanvasPage extends ConsumerWidget {
  const CanvasPage({Key key}) : super(key: key);
  static const String title = 'アフィン変換';
  static const textStyle = TextStyle(fontSize: 24, fontWeight: FontWeight.bold);
  @override
  Widget build(BuildContext context, ScopedReader watch) {
    final _provider = watch(canvasProvider);
    return Scaffold(
        appBar: AppBar(
            title: const Text(title,
                style: TextStyle(fontWeight: FontWeight.bold))),
        body: SafeArea(
            child: Stack(children: [
          Center(
            child: CustomPaint(
                painter: Painter(offsetList: _provider.destinationOffsetList)),
          ),
          Column(
              mainAxisAlignment: MainAxisAlignment.end,
              crossAxisAlignment: CrossAxisAlignment.stretch,
              children: [
                SizedBox(
                    height: 80,
                    width: 300,
                    child: Column(
                        crossAxisAlignment: CrossAxisAlignment.center,
                        children: [
                          Slider(
                              onChanged: (value) => _provider.radians = value,
                              value: _provider.radians,
                              min: 0,
                              max: 2 * pi),
                          const Text('Rotation', style: textStyle)
                        ])),
                SizedBox(
                    height: 80,
                    width: 300,
                    child: Column(
                        crossAxisAlignment: CrossAxisAlignment.center,
                        children: [
                          Slider(
                              value: _provider.scale,
                              onChanged: (value) => _provider.scale = value,
                              min: 1,
                              max: 20),
                          const Text('Scale', style: textStyle)
                        ]))
              ]),
          Align(
              alignment: Alignment.bottomCenter,
              child:
                  Row(mainAxisAlignment: MainAxisAlignment.center, children: [
                SizedBox(
                  width: 160,
                  child: TextField(
                    style: const TextStyle(fontSize: 36),
                    decoration: const InputDecoration(
                        labelText: 'tx',
                        hintText: '0',
                        border: OutlineInputBorder()),
                    onChanged: (tx) {
                      _provider.tx = double.parse(tx);
                    },
                  ),
                ),
                const SizedBox(width: 16, height: 500),
                SizedBox(
                    width: 160,
                    child: TextField(
                        style: const TextStyle(fontSize: 36),
                        decoration: const InputDecoration(
                          labelText: 'ty',
                          hintText: '0',
                          border: OutlineInputBorder(),
                        ),
                        onChanged: (ty) {
                          _provider.ty = double.parse(ty);
                        }))
              ]))
        ])));
  }
}

```
----
**ペインター**
ここで図形の描画をやっています。
今回は offset で点群データを用意。
それらを繋げて線分で描画、ということをやっています。
`Path()..addPolygon` で点をつなげて線を書いていけます。
```dart:painter.dart
import 'package:flutter/material.dart';

class Painter extends CustomPainter {
  Painter({@required this.offsetList});
  List<Offset> offsetList;
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.orange
      ..strokeWidth = 2.0
      ..style = PaintingStyle.stroke;
    final path = Path()..addPolygon(offsetList, false);
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(Painter oldDelegate) {
    return true;
  }
}
```

# おしまいに
`Offset` クラスにはスケールと平行移動のメソッドがあった気がしますが、原点を中心に回転するメソッドはないんですよね。普通につけておいてくれてもいい気がするんですが、と思いました。