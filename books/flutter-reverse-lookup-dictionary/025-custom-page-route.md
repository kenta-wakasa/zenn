---
title: ページ遷移をカスタムしたい
---

![](/images/q25/1.gif)

# A. PageRoute を継承しましょう

この画面は左側から出てくるようにしたい。いや、上から下にスライドするようにしたい。フェードインやフェードアウトさせてみたい。といった要望がでることがあります。

私たちが普段使っている画面遷移はこんな感じです。

```dart
  Navigator.of(context).push(
    MaterialPageRoute(
      builder: (context) => const NextPage(),
    ),
  )
```

この MaterialPageRoute という部分を代えてあげることで独自に定義したアニメーションでの画面遷移が可能になります。

## CustomPageRoute をつくってみる

フェードインとフェードアウトをさせてみたいので、そのような画面遷移ができる CustomPageRoute を作ってみましょう。いろいろ書いてありますが、みなさんがカスタムして使う場合に変更を加えるべき箇所をコメントしましたのでそこに注目していただければと思います。

```dart
class CustomPageRoute<T> extends PageRoute<T> {
  CustomPageRoute(this.child);

  @override
  Color get barrierColor => Colors.white;

  @override
  String? get barrierLabel => null;

  final Widget child;

  @override
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  ) {
    // ここを変えればいろんなトランジションにできるぞ
    return FadeTransition(
      opacity: animation,
      child: child,
    );
  }

  @override
  bool get maintainState => true;

  @override
  Duration get transitionDuration => const Duration(
        milliseconds: 1000, // 変化にかかる時間を指定
      );
}
```

## 使い方

使い方はいたってシンプルです。

```dart
  Navigator.of(context).push(
    CustomPageRoute( // ここが変わっただけ
      const NextPage(),
    ),
  );
```

MaterialPagerRoute の場合と見比べてみてください。

## コピペで動くサンプルコード

それではいつものようにコピペで動くサンプルを示しておしまいにしたいと思います。

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
  var position = const Offset(0, 0);
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('ページ遷移をカスタムしたい'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.of(context).push(
              CustomPageRoute(
                const NextPage(),
              ),
            );
          },
          child: const Text(
            'NEXT',
            style: TextStyle(
              fontWeight: FontWeight.bold,
              fontSize: 16,
            ),
          ),
        ),
      ),
    );
  }
}

class NextPage extends StatelessWidget {
  const NextPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: const Center(
        child: FlutterLogo(
          size: 120,
        ),
      ),
    );
  }
}

class CustomPageRoute<T> extends PageRoute<T> {
  CustomPageRoute(this.child);

  @override
  Color get barrierColor => Colors.white;

  @override
  String? get barrierLabel => null;

  final Widget child;

  @override
  Widget buildPage(
    BuildContext context,
    Animation<double> animation,
    Animation<double> secondaryAnimation,
  ) {
    // ここを変えればいろんなトランジションにできるぞ
    return FadeTransition(
      opacity: animation,
      child: child,
    );
  }

  @override
  bool get maintainState => true;

  @override
  Duration get transitionDuration => const Duration(
        milliseconds: 1000, // ミリ秒でトランジションにかかる時間を指定
      );
}
```
