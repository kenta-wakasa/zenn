---
title: 重い処理でもUIの動きを止めたくない
---

# A.compute を使いましょう

Flutter では非同期処理を使えるので UI を描画しているメインのスレッドをあまり邪魔することなく、それなりの計算処理を行うことができます。

ですが計算処理やネットワークとの通信などがあまりにも重すぎると UI 描画にも影響を与える場合があります。これはあくまでも単一のスレッドでプログラムが実行されているためです。ですので別スレッドで実行すれば UI の動きを損なうことなく計算を行うことができます。

## compute の使い方

簡単に別スレッドでの実行を行う方法として compute 関数が用意されています。使い方はとても簡単です。

第一引数には関数を与えます。そして第二引数ではその関数に与えたい引数を与えます。

```dart
  compute(第一引数, 第二引数);
```

int を double に変える簡単な関数を作ってみましょう。ただし、1 秒間プログラムを停止させるコードを挟みます。

```dart
double delaySecond(int input) async {
  sleep(const Duration(seconds: 1));
  return input * 1.0;
}
```

これを compute で実行したい場合はこのように書きます。

```dart
/// int を受け取って double を返すときはこう書く
final result = await compute<int, double>(delaySecond, 10);
```

簡単ですよね。

それでは実際に compute を使う場合と使わない場合でどのような変化が起きるのか試してみましょう。コピペで動くサンプルコードも用意しましたので活用してください。

## コピペで動くサンプルコード

![](/images/q20/1.gif)

```dart
import 'dart:io';
import 'dart:math';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

void main() {
  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      home: DemoCompute(),
    );
  }
}

class DemoCompute extends StatefulWidget {
  const DemoCompute({Key? key}) : super(key: key);

  @override
  State<DemoCompute> createState() => _DemoComputeState();
}

class _DemoComputeState extends State<DemoCompute> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: const Center(child: FlutterKazaguruma(size: 120, duration: Duration(seconds: 1))),
      floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
      floatingActionButton: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          // compute使った場合
          ElevatedButton(
            onPressed: () async {
              /// int を受け取って double を返すときはこう書く
              final result = await compute<int, double>(delaySecond, 1);
              print(result);
            },
            child: const Text('別スレッドで実行'),
          ),
          const SizedBox(width: 16),
          // 使わなかった場合
          ElevatedButton(
            onPressed: () {
              /// int を受け取って double を返すときはこう書く
              final result = delaySecond(3);
              print(result);
            },
            child: const Text('同じスレッドで実行'),
          ),
        ],
      ),
    );
  }
}

/// トップレベル関数
double delaySecond(int input) {
  sleep(const Duration(seconds: 1));
  return input * 1.0;
}

class FlutterKazaguruma extends StatelessWidget {
  const FlutterKazaguruma({
    Key? key,
    required this.size,
    required this.duration,
  }) : super(key: key);

  final double size;
  final Duration duration;

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        FlutterWing(
          size: size,
          duration: duration,
        ),
        FlutterWing(
          size: size,
          duration: duration,
          angle: (pi / 2) * 1, // 90度回転
        ),
        FlutterWing(
          size: size,
          duration: duration,
          angle: (pi / 2) * 2, // 180度回転
        ),
        FlutterWing(
          size: size,
          duration: duration,
          angle: (pi / 2) * 3, // 270度回転
        ),
      ],
    );
  }
}

/// 風車の羽根の部分
class FlutterWing extends StatefulWidget {
  const FlutterWing({
    Key? key,
    required this.size,
    required this.duration,
    this.angle = 0,
  }) : super(key: key);
  final double size;
  final double angle;
  final Duration duration;

  @override
  State<FlutterWing> createState() => _FlutterWingState();
}

class _FlutterWingState extends State<FlutterWing> with SingleTickerProviderStateMixin {
  late final controller = AnimationController(vsync: this, duration: widget.duration)..repeat();

  @override
  Widget build(BuildContext context) {
    return Transform.rotate(
      angle: widget.angle,
      child: Container(
        // 回転軸をずらすためにロゴを左下においやっています
        width: widget.size * 2,
        height: widget.size * 2,
        alignment: Alignment.bottomLeft,
        child: RotationTransition(
          turns: controller,
          // ロゴそのものの回転軸は右上にしています。
          alignment: Alignment.topRight,
          child: FlutterLogo(
            size: widget.size,
          ),
        ),
      ),
    );
  }
}
```
