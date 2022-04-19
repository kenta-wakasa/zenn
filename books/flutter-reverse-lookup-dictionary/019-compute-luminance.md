---
title: ボタンの文字色を見やすくしたい
---

# A, computeLuminance で算出できる輝度を活用しましょう

ElevatedButton であとから色を変更したとき、そのボタンの上に表示させる文字やアイコンが見づらくなってしまった経験はありませんか？

たとえば Colors.yellow に背景色を変更したとします。
すると文字色は白のままなので、見づらくなってしまいました。

![](/images/q19/1.png)

これを解決する方法はないのでしょうか？

背景色が明るい色であれば、黒を、暗い色であれば白を表示するようにすればよさそうです。

## 輝度は computeLuminance で算出できる。

Color クラスには メソッドとして computeLuminance() が備わっています。
これは相対輝度を求めるもので、暗ければ 0 に近い値が、明るければ 1 に近い値が 0~1 の範囲で返ってきます。算出方法などについては Wikipedia の記事をご覧ください。

https://en.wikipedia.org/wiki/Relative_luminance

この相対輝度を用いて、明るければ黒、暗ければ白を返すメソッドを用意すればよさそうです。Color の Extension にそのメソッドを生やしてみます。

```dart
extension OnPrimary on Color {
  /// 輝度が高ければ黒, 低ければ白を返す
  Color get onPrimary {
    // 輝度により黒か白かを決定する
    if (computeLuminance() < 0.5) {
      return Colors.white;
    }
    return Colors.black;
  }
}
```

## CustomElevatedButton に組み込む

この Extension を使って独自定義の CustomElevatedButton を作成しましょう。
外側から引数で与えた色を元に onPrimary メソッドを使って黒か白かを判定してあげるだけです。

```dart
class CustomElevatedButton extends StatelessWidget {
  const CustomElevatedButton({
    Key? key,
    required this.primaryColor,
    required this.text,
    required this.onPressed,
  }) : super(key: key);
  final Color primaryColor;
  final String text;
  final VoidCallback? onPressed;
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        primary: primaryColor,
        onPrimary: primaryColor.onPrimary, // ここで文字色が決定される
        textStyle: const TextStyle(
          fontWeight: FontWeight.bold,
          fontSize: 32,
        ),
      ),
      onPressed: onPressed,
      child: Text(text),
    );
  }
}
```

完成しました！
いろいろな色で試せるサンプルコードを書きましたので、そちらも併せてお試しください。

## コピペで動くサンプルコード

![](/images/q19/1.gif)

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
  final colors = [
    Colors.amber,
    Colors.black,
    Colors.blue,
    Colors.brown,
    Colors.cyan,
    Colors.orange,
    Colors.green,
    Colors.grey,
    Colors.indigo,
    Colors.lightGreen,
    Colors.lime,
    Colors.pink,
    Colors.purple,
    Colors.red,
    Colors.teal,
    Colors.yellow,
  ];

  Color selectedColor = Colors.blue;
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          Center(
            child: SizedBox(
              height: 160,
              width: 160,
              child: CustomElevatedButton(
                primaryColor: selectedColor,
                text: 'ボタン',
                onPressed: () {},
              ),
            ),
          ),
          Align(
            alignment: Alignment.bottomCenter,
            child: Row(
              children: colors
                  .map(
                    (e) => Expanded(
                      child: InkWell(
                        onTap: () {
                          setState(() {
                            selectedColor = e;
                          });
                        },
                        child: Container(
                          width: 50,
                          height: 50,
                          color: e,
                        ),
                      ),
                    ),
                  )
                  .toList(),
            ),
          )
        ],
      ),
    );
  }
}

extension OnPrimary on Color {
  /// 輝度が高ければ黒, 低ければ白を返す
  Color get onPrimary {
    // 輝度により黒か白かを決定する
    if (computeLuminance() < 0.5) {
      return Colors.white;
    }
    return Colors.black;
  }
}

class CustomElevatedButton extends StatelessWidget {
  const CustomElevatedButton({
    Key? key,
    required this.primaryColor,
    required this.text,
    required this.onPressed,
  }) : super(key: key);
  final Color primaryColor;
  final String text;
  final VoidCallback? onPressed;
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        primary: primaryColor,
        onPrimary: primaryColor.onPrimary, // ここで文字色が決定される
        textStyle: const TextStyle(
          fontWeight: FontWeight.bold,
          fontSize: 32,
        ),
      ),
      onPressed: onPressed,
      child: Text(text),
    );
  }
}
```
