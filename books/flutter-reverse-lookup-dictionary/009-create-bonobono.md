---
title: "ぼのぼのをつくりたい"
---

# A. Container で楕円をつくって組み合わせましょう

https://www.bonobono.jp

ぼのぼのがかわいいので、今回はぼのぼのをつくろうと思います。

ぼのぼのは多くの楕円で構成されているので、まずは楕円を簡単につくれる自作 Widget を定義しましょう。

## EllipticalContainer の定義

```dart
/// 楕円形のContainer
class EllipticalContainer extends StatelessWidget {
  const EllipticalContainer({
    Key? key,
    required this.width,
    required this.hight,
    this.color,
    this.child,
  }) : super(key: key);
  final double width;
  final double hight;
  final Color? color;
  final Widget? child;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: width,
      height: hight,
      alignment: Alignment.center,
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.all(
          Radius.elliptical(width, hight),
        ),
      ),
      child: child,
    );
  }
}
```

## EllipticalContainer の使い方

```dart
// 基本は width と height を与えるだけです。
// 色や子要素をもたせることもできます。
EllipticalContainer(width: 100, height: 200)
```

それではここからぼのぼのを作っていきましょう。
Stack と Align を使って丁寧に並べていきます。

出来上がったものがこちらです。

![](/images/q9/1.png)

## コピペで描けるぼのぼの

```dart
import 'package:flutter/material.dart';

void main() => runApp(const App());

class App extends StatelessWidget {
  const App({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Builder(
        builder: (context) {
          return Scaffold(
            body: Center(
              child: EllipticalContainer(
                width: 720,
                hight: 640,
                color: const Color.fromARGB(255, 125, 184, 213),
                child: Stack(
                  // Alignで細かく相対位置を指定できます
                  children: const [
                    // 目
                    Align(
                      alignment: Alignment(-.75, -.4),
                      child: EllipticalContainer(
                        width: 40,
                        hight: 48,
                        color: Color.fromRGBO(24, 21, 28, 1),
                      ),
                    ),
                    Align(
                      alignment: Alignment(.75, -.4),
                      child: EllipticalContainer(
                        width: 40,
                        hight: 48,
                        color: Color.fromRGBO(24, 21, 28, 1),
                      ),
                    ),
                    // 鼻
                    Align(
                      alignment: Alignment(-.3, .45),
                      child: EllipticalContainer(
                        width: 180,
                        hight: 150,
                        color: Color.fromRGBO(206, 190, 203, 1),
                      ),
                    ),
                    Align(
                      alignment: Alignment(.3, .45),
                      child: EllipticalContainer(
                        width: 180,
                        hight: 150,
                        color: Color.fromRGBO(206, 190, 203, 1),
                      ),
                    ),
                    Align(
                      alignment: Alignment(0, .2),
                      child: EllipticalContainer(
                        width: 130,
                        hight: 120,
                        color: Color.fromRGBO(24, 21, 28, 1),
                      ),
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}

/// 楕円形のContainer
class EllipticalContainer extends StatelessWidget {
  const EllipticalContainer({
    Key? key,
    required this.width,
    required this.hight,
    this.color,
    this.child,
  }) : super(key: key);
  final double width;
  final double hight;
  final Color? color;
  final Widget? child;

  @override
  Widget build(BuildContext context) {
    return Container(
      width: width,
      height: hight,
      alignment: Alignment.center,
      decoration: BoxDecoration(
        color: color,
        borderRadius: BorderRadius.all(
          Radius.elliptical(width, hight),
        ),
      ),
      child: child,
    );
  }
}
```
