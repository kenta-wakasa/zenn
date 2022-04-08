---
title: "IconButtonのPaddingを取り除きたい"
---

# A. BoxConstrains を忘れずに与えましょう

デザイン要求でアイコンボタンが密集していることがあります。マテリアルデザインでは禁止されているかもしれませんが、やらなければいけません。IconButton を使って並べてみましょう。まず普通に並べると次のようになります。

```dart
  Wrap(
    children: [
      for (var i = 0; i < 100; i++)
        IconButton(
          color: Colors.pink,
          onPressed: () {},
          icon: const Icon(Icons.pets),
        ),
    ],
  ),
```

![](/images/q8/1.png)

このパディングをできるだけ取り除きたいと思ったとき、IconButton に padding というプロパティがないかが気になります。調べてみると padding プロパティはあるようなので、それをゼロにしてしまいましょう。

```dart
  Wrap(
    children: [
      for (var i = 0; i < 100; i++)
        IconButton(
          color: Colors.pink,
          padding: EdgeInsets.zero, // これを追加
          onPressed: () {},
          icon: const Icon(Icons.pets),
        ),
    ],
  ),
```

これで実行してみます。

![](/images/q8/1.png)

あれ、何も変わらない。

実は IconButton にはデフォルトで BoxConstrains が指定されています。

```dart
const BoxConstraints(
  minWidth: kMinInteractiveDimension,
  minHeight: kMinInteractiveDimension,
)
```

ここで kMinInteractiveDimension は 48.0 だそうです。
ですのでこれを 0 にする必要があります。

```dart
  Wrap(
    children: [
      for (var i = 0; i < 100; i++)
        IconButton(
          color: Colors.pink,
          padding: EdgeInsets.zero,
          constraints: const BoxConstraints(), // これを追加
          onPressed: () {},
          icon: const Icon(Icons.pets),
        ),
    ],
  ),
```

![](/images/q8/2.png)

うまくいきました！

こういう問題に行き詰まったら内部実装を眺めてみることをおすすめします。
実装内容をみたい Widget の上で cmd + クリックするだけなので、お試しください。
こんな書き方もあるのかという発見もありますよ！

## コピペで動くサンプルコード

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
              child: Wrap(
                children: [
                  for (var i = 0; i < 100; i++)
                    IconButton(
                      padding: EdgeInsets.zero,
                      constraints: const BoxConstraints(),
                      splashRadius: 12,
                      color: Colors.pink,
                      onPressed: () {},
                      icon: const Icon(Icons.pets),
                    ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}
```
